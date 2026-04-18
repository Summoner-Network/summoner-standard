# Supplementary Note: Operational Semantics and Compositional Structure

**Standards set release:** `v0.2.0`<br>
**Date:** April 2026<br>
**Status:** Supplementary and informative. Complements the core profile within the current standards set.

## Purpose

This note provides additional context on the operational model behind Summoner, and why its compositional structure makes interoperability assumptions explicit.

The goal is to clarify what could be standardized and what can be tested, without requiring adoption of any particular implementation.

This note is meant to complement the main core profile, not replace it. Portable public identity is standardized separately in [`identity-profile.md`](identity-profile.md).

The broader vision is to make Summoner reviewable as a system of explicit semantics rather than as one SDK's hidden behavior. A standard is useful here because it turns routes, state changes, receiver outcomes, and public identity into shared artifacts that users can understand, reviewers can inspect, and implementations can test against.

## 1. Operational model at a glance

Summoner is a protocol and SDK for agent orchestration where message handling is organized around routes and an explicit execution semantics.

At runtime, a client does three things:
1. registers handlers (receive, send, hooks)
2. evaluates routes against a state tape
3. executes a receive-then-send cycle with explicit outcomes

### 1.1 Registers handlers (receive, send, hooks)

Developers register async handlers using decorators:
- `@receive(route=...)` for inbound processing and decision
- `@send(route=..., on_actions=..., on_triggers=...)` for outbound emission
- `@hook(Direction.RECEIVE|SEND)` for validation, filtering, and normalization

Minimal sketch:

```python
client = SummonerClient(name="ExampleAgent")

@client.hook(direction=Direction.RECEIVE)
async def validate_incoming(msg):
    if not isinstance(msg, dict):
        return None   # drop
    return msg        # keep

@client.receive(route="A --[ f ]--> B")
async def on_A_to_B(msg):
    return Move(Trigger.ok)

@client.send(
    route="A --[ f ]--> B",
    on_actions={Action.MOVE},
    on_triggers={Trigger.ok},
)
async def emit_trace():
    return "Moved A -> B via f"
````

### 1.2 Evaluates routes against a state tape

Summoner uses a light state structure (a "tape") that stores active states representing current context.

The tape can be:

* a single list of states, or
* an indexed family of state lists (useful for multi-threaded, multi-peer contexts)

A route can include gate conditions that determine when a receiver is eligible to run.

Minimal sketch:

```python
@client.upload_states()
async def upload_states(_payload):
    return ["A"]   # only A-gated receivers are eligible

@client.receive(route="A --[ f ]--> B")
async def decide(msg):
    if msg.get("ready"):
        return Move(Trigger.ok)
    return Stay(Trigger.ok)
```

### 1.3 Executes a receive-then-send cycle with explicit outcomes

Each receiver returns an explicit Action:

* **TEST**: only activate transition states, if any
* **STAY**: remain in the same region of state
* **MOVE**: advance by activating transition and target states

These actions update the tape and trigger relevant senders. This makes control flow observable and testable as a trace.

In the current core profile, this send step is limited to receiver-triggered untimed senders. Richer sender-side features such as event-data handoff or timed scheduling may exist in implementations, but they are not part of the first core review profile.

## 2. Route structure as a compositional interface

A Summoner route is structured syntax with an operational meaning:

* **Source states are gates.** They define when a receiver is eligible to run.
* **Label states name the transition.** Useful for intent, auditing, and downstream conditioning.
* **Target states are postconditions.** They become active after a successful advance.

Example route:

```
A, B, C --[ f, g ]--> X, Y, Z
```

### 2.1 Why this matters for interoperability

Many agent frameworks rely on structural assumptions that exist in practice but are not explicit in the interface (state machines, eligibility rules, propagation of intent, secondary decisions). When these assumptions remain implicit, composition breaks across systems.

Summoner pushes these assumptions into the route layer, where they can be:

* normalized (canonical parsing and route equivalence)
* inspected (enumerate routes, gates, and outcomes)
* tested (trace-level conformance and tape deltas)

### 2.2 Gate-and-activation intuition

**Eligibility.** A receiver is eligible if the current tape contains a state that matches at least one source gate. Source matching is existential:

* if the tape contains `A`, the route can fire
* if the tape contains `B`, the route can fire
* if the tape contains `C`, the route can fire
* if the tape contains none of them, it cannot fire

**Activation on MOVE.** If the receiver returns MOVE, Summoner activates label and target states:

* activated on MOVE: `f, g, X, Y, Z`
* not activated on MOVE: `A, B, C` (they are gates, not outputs)

Minimal illustration:

```python
tape = ["A"]
route = "A,B,C --[f,g]--> X,Y,Z"
# eligible because A matches at least one gate
```

**What happens on STAY or TEST.**

* On STAY: activate the source states (execution remains in the same region).
* On TEST: activate only the label states (record/observe without advancing).

### 2.3 Existential gating and conjunctive activation

A useful intuition is that routes bridge:

* an existential condition on the source side (at least one gate matches), to
* a structured outcome on the target side (all outputs become active).

## 3. Eligibility and gating, with minimal state syntax

Routes can gate on source states. Eligibility is decided by a small matching rule on states:

* `A` matches only `A`
* `/all` matches anything
* `/oneof(A,B)` matches `A` or `B`
* `/not(A,B)` matches anything except `A` or `B`

Minimal route example:

```python
@client.receive(route="/oneof(A,B) --[ f ]--> C")
async def f_to_C(msg):
    return Move(Trigger.ok)
```

Operational takeaway: gating is centralized and mechanically testable. This reduces ad hoc `if state == ...` logic spread across handlers.

## 4. Beyond 1-cells: amendments and intent

A recurrent source of interoperability breakage is that "secondary decisions" are encoded in ad hoc ways:

* amendments to earlier commitments
* policy constraints that refine downstream choices
* negotiation steps that transform a plan without changing the stage

Summoner supports this by allowing routes whose sources and targets include not only object states (`A, B, C`), but also transition states (`f, g, p, q`). This makes "transitions between transitions" first-class.

Standards relevance: this isolates what belongs to the main stage progression versus what belongs to amendments and intent. Conformance can test them separately.

## 5. Mini structural examples

The examples below are intentionally self-contained. They are not references to another repository; they are small local sketches meant to show the structural point directly in this document.

### 5.1 Single-route example

What it demonstrates:

* a single route
* eligibility via tape gating
* explicit outcome via MOVE / STAY / TEST

This is the smallest useful shape for a conformance-style trace test.

Receiver snippet:

```python
@client.upload_states()
async def upload_states(_msg):
    return ["draft"]

@client.receive(route="draft --[approve]--> approved")
async def evaluate_approval(msg):
    if msg.get("approved"):
        return Move(Trigger.ok)
    if msg.get("review_only"):
        return Test(Trigger.ok)
    return Stay(Trigger.retry)
```

This one example already shows the core shape:

* `draft` is the gate
* `approve` is the explicit transition label
* `approved` is the postcondition activated on MOVE
* STAY and TEST remain visible, rather than being hidden in ad hoc control flow

### 5.2 Direct step versus staged path

What it demonstrates:

* the structural difference between a direct step and a refinement into intermediate stages
* why interoperability review must say which observables it compares

Minimal route sketches:

```text
Design A:
request --[approve]--> approved

Design B:
request --[review]--> under_review
under_review --[approve]--> approved
```

Both designs can end in `approved`, but they do not expose the same intermediate state structure. Summoner makes that difference explicit instead of hiding it.

A practical review question is therefore not just "do they both finish?" but "at what abstraction layer are they intended to be compared?" The standard benefits from this explicitness because it can specify exactly which traces must match and which structural refinements remain implementation choice.

## 6. State-with-context and branching workflows (optional intuition)

This section is optional. It is not required to understand the operational semantics, but it can help explain why agent state is better modeled as state-with-context.

* The tape encodes position plus active commitments.
* Routes define the local continuation structure.
* Handlers implement local rules for which branch to take.

## 7. Application-shaped examples

### 7.1 Supply-chain amendment workflow

This pattern models a common industrial workflow:

* an initial operational decision
* amendments that refine commitments
* execution under feasibility constraints

Minimal route sketch:

```text
Stage progression:
request --[quote]--> quoted
quoted --[allocate]--> allocated
allocated --[ship]--> shipped

Primary sourcing choice:
request --[source_local]--> quoted
request --[source_import]--> quoted

Amendments to sourcing intent:
source_local --[green_clause]--> green
source_local --[low_cost_clause]--> low_cost
source_import --[green_clause]--> green
source_import --[low_cost_clause]--> low_cost
```

Execution can then branch at `allocated --[ship]--> shipped` based on feasibility plus whichever sourcing path and amendment states are active.

Operationally, this yields a trace that is mechanically testable: tape states and emitted messages reflect the chosen 1-cell, the chosen amendment, the executed shipment, and any exception taken.

### 7.2 Handshake and session establishment

A handshake can be expressed as a state machine with explicit gates (preconditions) and explicit commitments (activated states) that later steps depend on. This makes "what was verified" and "what was committed" explicit, which is useful for interoperability and audit.

Minimal route sketch:

```text
unverified --[hello]--> challenged
challenged --[proof_ok]--> authenticated
authenticated --[session_open]--> ready
```

Later routes can gate on `authenticated` or `ready`, which makes the handshake result part of the visible state model instead of an invisible side effect.

### 7.3 Composition by extension: adding negotiation

An existing workflow can be extended by inserting new routes and states without rewriting the entire agent. For example, a negotiation phase can be inserted by introducing intermediate states and amendment routes that refine downstream choices while keeping prior semantics intact.

Minimal route sketch:

```text
Base workflow:
quoted --[accept_quote]--> committed

Extended workflow:
quoted --[counter_offer]--> negotiating
negotiating --[accept_counter]--> committed
negotiating --[reject_counter]--> quoted
```

This is useful standard-wise because it shows how the route layer can grow without making control flow opaque. Extensions remain explicit and inspectable.

## 8. What belongs in the standard set

The immediate standard candidate is not the full SDK. It is a core profile capturing the interoperability-critical semantics:

* route grammar, parsing, and canonicalization
* token matching rules (gating and acceptance)
* tape model (single or indexed) and activation rules
* Action semantics (MOVE / STAY / TEST) and tape update rules
* hook ordering and drop behavior (returning `None`)
* receiver-triggered untimed sender eligibility and ordering

A conformance suite can be derived as trace-level tests: given an initial tape and a sequence of inputs, a conforming implementation produces the same sequence of tape deltas and observable events.

Current SDK sender extensions such as receiver-attached event payload data, sender-owned payload transfer policy, and timed or guard-based scheduling are promising, but they fit better as later extension profiles than as part of the first core standard.

Portable self-signed public identity is a different kind of candidate. It fits the standard set more cleanly because it has a crisp portable object boundary and verification contract, which is why the current review release standardizes it separately in [`identity-profile.md`](identity-profile.md).

## 9. DNA: representing agent behavior as a portable bundle

Summoner can serialize handler registrations into a compact DNA representation (routes, triggers, priorities, and source-level code pointers).

Intended uses:

* portability of agent behavior across environments
* reproducible deployment of the same semantics
* a concrete object for security review (what handlers exist, what routes they bind to, what they emit)
* a unit for signing, versioning, and conformance tests

Minimal usage:

```python
dna_json = client.dna()
entries = json.loads(dna_json)  # enumerate routes and handlers
```

## 10. Why this matters now

This framing is intended to support a responsible path toward a testable profile. It is designed to separate:

* what must be interoperable and observable, from
* implementation and deployment choices.
