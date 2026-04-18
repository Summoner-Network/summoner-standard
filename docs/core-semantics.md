# Core Profile: Semantics and Conformance Requirements

**Profile ID:** `summoner-core-profile/0.1`<br>
**Standards set release:** `v0.2.0`<br>
**Date:** April 2026<br>
**Status:** Draft for review.

## 1. Scope

This document defines the first core profile in the standards set for the interoperability-critical semantics of Summoner:
- route grammar and canonicalization
- node matching
- tape structure
- receive hooks, receiver eligibility, and tape activation semantics
- receiver-triggered untimed sender admission and ordering
- a core DNA view suitable for debugging, audit, and conformance traces

It is intended to support implementation-independent conformance testing.

This profile intentionally does not standardize:
- transport, deployment, or UI tooling
- application-specific workflows
- the richer sender orchestration runtime found in current SDKs, including `Event.data`, `use_data`, `data_mode`, `every`, `run_while`, and sender modes that continue or fire without pending receiver outcomes
- full SDK DNA capture formats used for code replay, merger reconstruction, or context shipping

A richer runtime MAY expose more behavior than this document requires. Conformance to the core profile means that the richer runtime can be projected to the observable boundary defined here.

The current public implementation reference for this area is [`Summoner-Network/summoner-core`](https://github.com/Summoner-Network/summoner-core), especially the current 1.2.x line. That repository is informative, not normative: conformance is defined by this document, not by a particular SDK release.

### 1.1 Design intent

This core profile is not intended to freeze one SDK's internal scheduler, queue layout, worker model, or decorator surface. It is intended to standardize the smallest execution contract that can be:
- observed from outside the runtime
- reproduced by an independent implementation
- compared by a conformance harness

This design choice explains why the profile includes sender-local multiplicity via `multi` but defers `Event.data`, `use_data`, and `data_mode`.

`multi` is a sender-local emission rule. Once a sender has already been admitted, `multi` only changes whether that sender invocation yields one outbound payload or several.

`use_data` is broader. It changes the receiver-event-sender contract itself by introducing receiver-attached event payloads, sender parameter passing, payload transfer policy, and per-event invocation behavior. Those concerns can make sense as a later extension profile, but they are intentionally outside the first core profile.

## 2. Conventions and terminology

The keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are normative.

- **Node**: a state token used for gating and activation.
- **Route**: structured syntax connecting source Nodes, label Nodes, and target Nodes.
- **Tape**: the runtime collection of active Nodes.
- **Receiver**: a handler that processes inbound messages for a Route.
- **Sender**: a handler that emits outbound messages based on a Route and a receiver outcome.
- **Event**: a receiver outcome carrying an Action and Trigger; implementations MAY attach additional payload or metadata, but those extensions are outside this profile.
- **Pending outcome**: a receiver outcome that has been admitted for possible sender processing in an untimed sender pass.

## 3. Route grammar and canonicalization

A Route is either:
- an **object route** consisting of one or more Nodes
- an **arrow route** with three parts: source Nodes, label Nodes, and target Nodes

### 3.1 Grammar

```ebnf
Token      := Identifier | "/all" | "/oneof(" List ")" | "/not(" List ")"
List       := Token ("," Token)*
Object     := List
Arrow      := Source "--[" Label "]-->" Target
Source     := List | ε
Label      := List | ε
Target     := List | ε
Route      := Object | Arrow
Identifier := [A-Za-z_][A-Za-z0-9_]*
```

Implementations MUST define at least one arrow style equivalent to `--[ ... ]-->` with comma-separated tokens.

Whitespace MAY appear anywhere and MUST be ignored for parsing.

### 3.2 Canonicalization requirements

Implementations MUST define a canonical string form so that parsing then re-serializing yields a stable representation.

Canonicalization requirements:
- token strings MUST be trimmed of leading and trailing whitespace
- empty tokens, after trimming, MUST be rejected
- if multiple arrow styles are supported, canonicalization MUST select exactly one canonical style for serialization

## 4. Node matching semantics

Node matching is defined by a predicate `accepts(gate, state)` used during gating and route-to-route compatibility checks.

A conforming implementation MUST support the following Node forms:
- `A` (plain): matches only the same identifier
- `/all`: matches any Node
- `/oneof(A,B,...)`: matches a plain Node whose identifier is in the listed set
- `/not(A,B,...)`: matches any plain Node not in the listed set

Implementations MAY extend the token language, but MUST preserve backward compatibility for the forms above within this profile.

## 5. Tape model

A Tape is a collection of active Nodes.

This profile supports four tape shapes:
- **single**: one Node, treated as a singleton list
- **many**: a list of Nodes
- **index-single**: a mapping from keys to one Node each, treated as singleton lists
- **index-many**: a mapping from keys to lists of Nodes

Keys are application-defined strings. Implementations MAY add internal prefixes for storage, but MUST present keys in their original form at the conformance boundary.

## 6. Execution semantics

The core profile standardizes an observable pipeline with four steps:
1. receive hooks process an inbound message
2. eligible receivers run for that message
3. tape updates are derived from returned Events
4. untimed sender passes evaluate pending receiver outcomes

Receiver outcomes produced in Sections 6.1 through 6.3 become pending outcomes for sender processing. A sender pass MAY consume outcomes produced by one inbound message or by multiple inbound messages batched together, but the observable result MUST be deterministic at the conformance boundary.

This profile defines only sender work driven by pending receiver outcomes. Sender modes that continue across passes, depend on timers or runtime guards, or fire without a pending receiver outcome are outside this profile.

### 6.1 Hook semantics

Hooks are ordered by a priority tuple using ascending lexicographic order. Each hook receives the current payload and returns either a transformed payload or `None`.

Normative behavior:
- if any receive hook returns `None`, the inbound message MUST be dropped and no receivers run
- if any send hook returns `None`, the outbound emission MUST be dropped
- hook ordering MUST be deterministic

### 6.2 Receiver eligibility

Given a Route with source gates `X_1, ..., X_n`, a receiver is eligible for a tape entry `(key, state)` if `accepts(X_i, state)` holds for at least one `i`.

Initial arrow routes, with empty source, MUST be eligible exactly once per inbound message.

Receiver invocation ordering MUST be deterministic. This profile RECOMMENDS ordering by:
1. receiver priority tuple
2. route string
3. tape key and state representation

### 6.3 Receiver outcomes and activation

Each receiver returns an Event with:
- an **Action** in `{TEST, STAY, MOVE}`
- a **Trigger** value, which is implementation-defined

Conforming implementations MAY attach additional event payload or metadata to receiver outcomes, but those fields are outside this profile and MUST NOT be required for core conformance.

The Action determines which Nodes are activated into the tape.

#### Activation rules

| Route kind | TEST activates | STAY activates | MOVE activates |
| --- | --- | --- | --- |
| Object route, no target and no label | none | source Nodes | source Nodes |
| Arrow route | label Nodes | source Nodes | label Nodes plus target Nodes |

Tape update semantics MUST be deterministic.

If the tape is indexed, activated Nodes are added to the same tape key that provided eligibility unless the route is initial, in which case key association is implementation-defined.

### 6.4 Sender triggering and sender passes

This section defines only untimed sender activations driven by pending receiver outcomes.

A sender registration in this profile is attached to a Route and MAY be filtered by:
- receiver Action
- receiver Trigger

A sender is eligible for a given pending outcome if:
1. the sender Route is compatible with the originating receiver Route under Node acceptance
2. any declared Action filter matches the receiver Action
3. any declared Trigger filter matches the receiver Trigger

Normative behavior:
- a sender pass evaluates sender registrations against the pending outcomes available to that pass
- a sender MAY be marked `multi` to allow multiple emissions per sender pass
- if `multi` is not set, then for a given sender registration there MUST be at most one emission per sender pass per `(route, tape key)`
- implementations that support per-event sender invocations from receiver-attached event data, or sender modes that continue across passes, are defining extensions outside this profile
- sender invocation ordering MUST be deterministic

## 7. DNA representation

DNA in this profile is a **core projection** of registered behavior. It is not required to be a byte-for-byte copy of any particular SDK export format.

A conforming core DNA document MUST enumerate registrations relevant to this profile:
- receivers
- senders
- hooks

For each registration, the core DNA view MUST include:
- registration kind: `receiver`, `sender`, or `hook`
- canonical route string, except for hooks
- priority tuple, which may be empty
- function identity metadata sufficient for debugging, such as module and function name
- for senders: Action and Trigger filters, and the `multi` flag

DNA output MUST be deterministic for a fixed set of registrations. Implementations SHOULD produce a stable ordering to support signing, diffing, and audit workflows.

Implementations MAY export richer SDK DNA that includes source code, state-upload or state-download adapters, context headers, or sender extension fields. Core conformance to this profile MUST NOT depend on those extra entries or fields.

## 8. Conformance requirements

A conforming implementation MUST provide a conformance mode that exposes trace-level observables sufficient to validate the semantics in Sections 3 through 7.

Required conformance properties:
- **Parsing conformance:** Route parsing and canonicalization MUST round-trip.
- **Matching conformance:** Node acceptance MUST match the semantics in Section 4.
- **Execution conformance:** Given the same initial tape and the same sequence of inbound messages, the implementation MUST produce the same normalized sequence of receiver outcomes and tape deltas, modulo explicitly declared nondeterminism.
- **Emission conformance:** Receiver-triggered untimed sender eligibility, per-pass emission limits, and drop behavior MUST match Sections 6.1 and 6.4.
- **DNA conformance:** If the implementation exposes DNA for interchange or audit, the core DNA view MUST match Section 7.
- **Determinism:** Any intended nondeterminism MUST be declared by the profile, and conformance outputs MUST specify what is compared.

This profile RECOMMENDS a reference test harness that compares implementations using normalized traces:
- ordered receiver activations
- canonical tape delta representations
- ordered sender passes and sender emissions

See [`conformance.md`](conformance.md) for practical guidance.

If an implementation supports out-of-scope sender extensions such as receiver-attached event data or timed scheduling, its conformance mode MUST disable those features or exclude their observables from the normalized trace used for core comparison.

## 9. Security considerations

This profile does not standardize cryptography. It does standardize control points relevant to security review:
- receive hooks as validation and normalization points
- drop semantics, via `None`, to prevent unsafe processing
- DNA as an audit surface over enumerable registered behavior

Implementations SHOULD define limits for message size, handler runtime, and tape growth.

For standardized public identity semantics, see [`identity-profile.md`](identity-profile.md).

## 10. Versioning

Implementations claiming conformance to this document MUST expose the profile identifier `summoner-core-profile/0.1`.

Breaking changes to parsing, matching, DNA meaning, or execution semantics MUST increment the major version.

Backward-compatible clarifications or additive features within the core boundary MAY increment the minor version.

See [`versioning.md`](versioning.md) for the multi-profile versioning policy.
