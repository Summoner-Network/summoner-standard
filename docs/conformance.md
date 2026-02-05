# Conformance guidance (trace-level)

This page is informative. The normative conformance requirements are in `core-semantics.md` Section 8.

The goal is to make interoperability testable without requiring a shared implementation.

## 1. Conformance mode

A conforming implementation MUST expose a conformance mode that emits observables sufficient to validate:
- parsing and canonicalization (Section 3)
- node matching (Section 4)
- execution semantics and tape deltas (Section 6)
- emission semantics and drop behavior (Sections 6.1 and 6.4)

The profile RECOMMENDS comparing implementations using normalized traces.

## 2. What to log

A practical minimal trace model includes:

### 2.1 Parsing and canonicalization
For each route string under test:
- input route string
- parsed structure (source, label, target tokens)
- canonical serialized route string

Expected property: parse then serialize is stable.

### 2.2 Matching
For each matching case:
- gate token
- state token
- boolean result of `accepts(gate, state)`

Expected property: results match the profile semantics.

### 2.3 Execution
For each inbound message processed:
- pre-cycle tape (canonical representation)
- receive hook outcomes (ordered list; include drop if any hook returns None)
- ordered receiver activations:
  - receiver identity metadata
  - route (canonical)
  - tape key and state that provided eligibility (if applicable)
  - returned Action and Trigger
- tape delta:
  - added nodes (with tape key association if indexed)
  - removed nodes (if your implementation supports removal; if not, keep empty)
- ordered sender activations:
  - sender identity metadata
  - route (canonical)
  - Action and Trigger filters
  - multi flag
  - send hook outcomes (including drop)

Expected property: for identical initial tape and identical message sequence, normalized traces match (modulo explicitly declared nondeterminism).

## 3. Canonical encodings

To compare independent implementations, define canonical encodings for:
- tokens and routes (using the canonicalization rules)
- tape
- tape deltas
- receiver and sender identity metadata (module name and function name is sufficient for debugging)

A recommended tape encoding:

- **single / many**: sorted list of canonical token strings
- **index-single / index-many**: sorted list of `(key, sorted tokens)` pairs

Sorting rules must be documented and deterministic.

## 4. Determinism and concurrency

If your implementation is concurrent, you still need deterministic observables.

Two common strategies:
1. Force a deterministic scheduler in conformance mode.
2. Keep execution concurrent but emit a normalized trace that is deterministic (for example, sort activations by a defined key).

If nondeterminism is intended, it MUST be declared and the trace comparator must specify what is compared and what is ignored.

## 5. Suggested repository evolution

A standard-style repo typically adds these next:
- `conformance/vectors/`: route strings, tapes, and message sequences
- `conformance/expected/`: expected normalized traces
- `conformance/harness/`: a black-box runner and comparator

This repo can start with documentation-only and grow toward a test harness as implementations appear.
