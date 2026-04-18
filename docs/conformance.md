# Conformance Guidance

This page is informative. The normative requirements live in the relevant profile documents:
- [`core-semantics.md`](core-semantics.md)
- [`identity-profile.md`](identity-profile.md)

The goal is to make interoperability testable without requiring a shared implementation.

## 1. Current conformance surfaces

The current standards set has two normative conformance surfaces:
- the **core profile**, which compares normalized execution traces
- the **agent identity profile**, which compares canonical public-record behavior

A runtime may implement both profiles or only one. Conformance claims should name the profile explicitly.

## 2. Core profile conformance

### 2.1 Conformance mode

A core-conforming implementation MUST expose a mode that emits observables sufficient to validate:
- parsing and canonicalization
- node matching
- receiver outcomes and tape deltas
- receiver-triggered untimed sender admissions and send-hook drop behavior
- the core DNA view, if DNA is exposed

If an implementation supports sender extensions outside the core profile, conformance mode should disable them or log them separately so the core comparison remains stable.

### 2.2 What to log

A practical minimal trace model includes the following.

#### Parsing and canonicalization
For each route string under test:
- input route string
- parsed structure
- canonical serialized route string

Expected property: parsing then serialization is stable.

#### Matching
For each matching case:
- gate token
- state token
- boolean result of `accepts(gate, state)`

Expected property: results match the profile semantics.

#### Receiver processing
For each inbound message processed:
- a message or pass identifier
- pre-message tape in canonical form
- ordered receive-hook outcomes, including drop if any hook returns `None`
- ordered receiver activations:
  - receiver identity metadata
  - canonical route
  - tape key and state that provided eligibility, if any
  - returned Action and Trigger
- tape delta in canonical form

Expected property: for identical initial tape and identical inbound message sequence, the normalized receiver trace matches.

#### Sender passes
For each untimed sender pass:
- sender-pass sequence number
- identifiers of the pending receiver outcomes considered by that pass
- ordered sender activations:
  - sender identity metadata
  - canonical route
  - Action and Trigger filters
  - `multi` flag
  - send-hook outcomes, including drop

Expected property: receiver-triggered untimed sender admissions, per-pass emission limits, and drops match the core profile.

### 2.3 Canonical encodings

To compare independent implementations, define canonical encodings for:
- tokens and routes
- tape
- tape deltas
- receiver and sender identity metadata
- the core DNA view, if exported

A recommended tape encoding is:
- **single** and **many**: sorted list of canonical token strings
- **index-single** and **index-many**: sorted list of `(key, sorted tokens)` pairs

Sorting rules must be documented and deterministic.

### 2.4 Determinism and concurrency

If an implementation is concurrent, it still needs deterministic observables.

Two common strategies are:
1. force a deterministic scheduler in conformance mode
2. keep execution concurrent but emit a normalized trace that is deterministic

If nondeterminism is intended, it MUST be declared and the trace comparator must specify what is compared and what is ignored.

## 3. Identity profile conformance

### 3.1 What to test

A practical identity-profile test set should exercise:
- record schema and required-field handling
- exact handling of `v == "id.v1"`
- canonical public-core reconstruction
- signature verification success and failure
- fingerprint derivation

### 3.2 What to log

For each identity test vector, log:
- the input public identity record
- the reconstructed canonical public-core object
- the canonical public-core bytes, or a stable digest of them
- verification result
- derived fingerprint, when verification is expected to succeed

Expected property: valid records yield the same verification result and the same fingerprint across independent implementations.

### 3.3 Recommended vectors

A useful baseline vector set includes:
- a valid minimal record without `meta`
- a valid record with structured `meta`
- a record with a modified `meta` value but the original `sig`
- a record with the wrong `v`
- a record missing one required field
- a record with invalid Base64 or invalid signature length
- a record with an additional unsigned field, confirming that verification still depends only on the authenticated public core

## 4. Optional Aurora-specific file interoperability

The current standards set does not require local identity-file compatibility.

If you also want to compare SDK persistence behavior, keep those tests separate and label them clearly as Aurora-specific:
- plaintext identity file shape
- encrypted identity file shape
- `scrypt` parameter handling
- `AES-GCM` associated-data handling for `summoner/identity_file/v1`

## 5. Suggested next assets

A review-ready repo typically adds these next:
- `conformance/vectors/` for route, trace, and identity test inputs
- `conformance/expected/` for normalized expected outputs
- `conformance/harness/` for black-box runners and comparators

This repository can remain documentation-first while the profile boundary is still under review, then add executable vectors once the wording stabilizes.
