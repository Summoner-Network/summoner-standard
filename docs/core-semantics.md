# Core Semantics and Conformance Requirements

**Version:** `0.1.0`
**Date:** January 2026  
**Status:** Draft for discussion.

## 1. Scope

This document defines a standard profile for the interoperability-critical semantics of Summoner:
- route grammar and parsing
- node matching
- tape structure
- receive/send execution cycle

It is intended to support implementation-independent conformance testing.

This profile does not standardize transport, deployment, UI tooling, or application-specific workflows. Those concerns are out of scope for this profile.

## 2. Conventions and terminology

The keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as normative requirements.

- **Node**: a state token used for gating and activation.  
- **Route**: structured syntax connecting source Nodes, label Nodes, and target Nodes.  
- **Tape**: the runtime collection of active Nodes.  
- **Receiver**: a handler that processes inbound messages for a Route.  
- **Sender**: a handler that emits outbound messages based on a Route and a receiver outcome.

## 3. Route grammar and canonicalization

A Route is either:
- an **object route** consisting of one or more Nodes, or
- an **arrow route** with three parts: source Nodes, label Nodes, and target Nodes.

### 3.1 Grammar

The profile defines the following token language and arrow structure:

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
````

Implementations MUST define at least one arrow style equivalent to `--[ ... ]-->` with comma-separated tokens.

Whitespace MAY appear anywhere and MUST be ignored for parsing.

### 3.2 Canonicalization requirements

Implementations MUST define a canonical string form so that parsing then re-serializing yields a stable representation.

Canonicalization requirements:

* Token strings MUST be trimmed of leading and trailing whitespace.
* Empty tokens (after trimming) MUST be rejected.
* If multiple arrow styles are supported, canonicalization MUST select exactly one canonical style for serialization.

## 4. Node matching semantics

Node matching is defined by a predicate `accepts(gate, state)` used during gating and route-to-route compatibility checks.

A conforming implementation MUST support the following Node forms:

* `A` (plain): matches only the same identifier.
* `/all`: matches any Node.
* `/oneof(A,B,...)`: matches a plain Node whose identifier is in the listed set.
* `/not(A,B,...)`: matches any plain Node not in the listed set.

Implementations MAY extend the token language, but MUST preserve backward compatibility for the forms above within this profile.

## 5. Tape model

A Tape is a collection of active Nodes.

This profile supports four tape shapes:

* **single**: one Node (treated as a singleton list)
* **many**: a list of Nodes
* **index-single**: a mapping from keys to one Node each (treated as singleton lists)
* **index-many**: a mapping from keys to lists of Nodes

Keys are application-defined strings. Implementations MAY add internal prefixes for storage, but MUST present keys in their original form at the conformance boundary.

## 6. Execution semantics

An execution cycle processes an inbound message through:

1. receive hooks
2. eligible receivers
3. tape updates
4. eligible senders with send hooks

### 6.1 Hook semantics

Hooks are ordered by a priority tuple (lexicographic order, ascending). Each hook receives the current payload and returns either a transformed payload or `None`.

Normative behavior:

* If any receive hook returns `None`, the inbound message MUST be dropped and no receivers run.
* If any send hook returns `None`, the outbound emission MUST be dropped.
* Hook ordering MUST be deterministic.

### 6.2 Receiver eligibility

Given a Route with source gates `X_1, ..., X_n`, a receiver is eligible for a tape entry `(key, state)` if `accepts(X_i, state)` holds for at least one `i`.

Initial arrow routes (with empty source) MUST be eligible exactly once per cycle.

Receiver invocation ordering MUST be deterministic. This profile RECOMMENDS ordering by:

1. receiver priority tuple
2. route string
3. tape key and state representation

### 6.3 Receiver outcomes and activation

Each receiver returns an Event with:

* an **Action** in `{TEST, STAY, MOVE}`
* a **Trigger** value (implementation-defined)

The Action determines which Nodes are activated into the tape.

#### Activation rules

| Route kind                         | TEST activates | STAY activates | MOVE activates             |
| ---------------------------------- | -------------- | -------------- | -------------------------- |
| Object route (no target, no label) | none           | source Nodes   | source Nodes               |
| Arrow route                        | label Nodes    | source Nodes   | label Nodes + target Nodes |

Tape update semantics MUST be deterministic.

If the tape is indexed, activated Nodes are added to the same tape key that provided eligibility unless the route is initial, in which case key association is implementation-defined.

### 6.4 Sender triggering

Senders are registered on a Route and filtered by:

* receiver Action
* receiver Trigger

A sender is eligible if:

1. its filters match, and
2. its Route is compatible with the receiver Route under Node acceptance.

Normative behavior:

* A sender MAY be marked `multi` to allow multiple emissions per cycle; otherwise it emits at most once per cycle per `(route, tape key)`.
* If both Action and Trigger filters are present, both MUST match for the sender to run.
* Sender invocation ordering MUST be deterministic.

## 7. DNA representation

DNA is a portable serialization of an agent's registered behavior.

Within this profile, DNA is defined as a JSON document that enumerates registrations for:

* receivers
* senders
* hooks

A conforming DNA document MUST include, for each registration:

* registration kind: `receiver` | `sender` | `hook`
* route string (canonicalized per Section 3), except for hooks
* priority tuple (may be empty)
* function identity metadata sufficient for debugging (module and function name)
* for senders: action and trigger filters, and the `multi` flag

DNA output MUST be deterministic for a fixed set of registrations. Implementations SHOULD produce a stable ordering to support signing, diffing, and audit workflows.

## 8. Conformance requirements

A conforming implementation MUST provide a conformance mode that exposes trace-level observables sufficient to validate the semantics in Sections 3 to 6.

Required conformance properties:

* **Parsing conformance:** Route parsing and canonicalization MUST round-trip.
* **Matching conformance:** Node acceptance MUST match the semantics in Section 4.
* **Execution conformance:** Given the same initial tape and the same sequence of inbound messages, the implementation MUST produce the same sequence of tape deltas and receiver Actions (modulo explicitly declared nondeterminism).
* **Emission conformance:** Sender eligibility and drop behavior MUST match Sections 6.1 and 6.4.
* **Determinism:** Any intended nondeterminism MUST be declared by the profile (for example, concurrency), and conformance outputs MUST specify what is compared.

This profile RECOMMENDS a reference test harness that compares implementations using normalized traces:

* ordered receiver activations
* ordered sender emissions
* canonical tape delta representation

See [`conformance.md`](conformance.md) for practical trace guidance.

## 9. Security considerations

This profile does not standardize cryptography. It does standardize control points relevant to security review:

* receive hooks as mandatory validation and normalization points
* drop semantics (`None`) to prevent unsafe processing
* DNA as an audit surface (enumerable behavior surface area)

Implementations SHOULD define limits for message size, handler runtime, and tape growth.

Where DNA is used for interchange, implementations SHOULD support signing and version pinning.

## 10. Versioning

Implementations MUST expose a profile version identifier.

* Breaking changes to parsing, matching, or execution semantics MUST increment the **major** version.
* Backward-compatible additions MAY increment the **minor** version.

See [`versioning.md`](versioning.md) for a practical policy and recommended identifiers.
