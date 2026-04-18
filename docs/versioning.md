# Versioning Policy

This repository publishes a standards set composed of separately versioned profiles.

A repository release tag packages a review snapshot of the whole standards set. A conformance claim, however, should name the specific profile identifier it implements.

## 1. Current profiles

| Profile | Identifier | Current draft |
| --- | --- | --- |
| Core profile | `summoner-core-profile/0.1` | included in review release `v0.2.0` |
| Agent identity profile | `summoner-agent-identity-profile/id.v1` | included in review release `v0.2.0` |

## 2. Repository release tags

Repository release tags, such as `v0.2.0`, mark a coherent review snapshot of the standards set.

A repository release MAY:
- update one profile without changing the other
- add informative guidance
- add a new draft profile
- improve presentation or navigation

A repository release tag is not, by itself, enough to declare conformance. The profile identifier must also be stated.

## 3. Core profile versioning

Breaking changes to the core profile require a major bump. Examples:
- route parsing changes that accept or reject strings differently
- changes to canonicalization that change the canonical string for existing routes
- changes to node matching semantics for existing token forms
- changes to receiver eligibility or initial-route behavior
- changes to activation rules for TEST, STAY, or MOVE
- changes to sender admission or per-pass emission semantics
- changes to the meaning of the core DNA view

Backward-compatible core additions may increment the minor version. Examples:
- adding new token forms while preserving existing required semantics
- adding optional trace fields without changing normalized comparisons
- tightening clarifications without changing observables

## 4. Identity profile versioning

For the identity profile, the wire version tag `v == "id.v1"` is part of the normative public record.

Breaking changes that require a new identity-profile version include:
- changing the required signed fields
- changing canonical JSON rules
- changing key or signature encoding rules
- changing fingerprint derivation
- changing the meaning of the wire version field

Clarifications or additional informative guidance that do not change wire behavior do not require a new wire version.

## 5. Declaring conformance

A simple conformance declaration should include:
- the profile identifier or identifiers claimed
- the repository review release used for review, if helpful
- supported tape shapes and arrow styles for the core profile
- whether conformance mode forces determinism
- for the identity profile, whether the implementation supports generation, verification, fingerprint derivation, or all three

Examples:
- `summoner-core-profile/0.1`
- `summoner-agent-identity-profile/id.v1`
- `summoner-core-profile/0.1 + summoner-agent-identity-profile/id.v1`
