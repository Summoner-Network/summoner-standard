# Versioning policy

The normative requirement is:

- Implementations MUST expose a profile version identifier.
- Breaking changes to parsing, matching, or execution semantics MUST increment the major version.
- Backward-compatible additions MAY increment the minor version.

This page makes that policy easier to apply consistently.

## 1. What counts as a breaking change

Treat the following as breaking changes (major bump):
- route parsing changes that accept or reject strings differently
- changes to canonicalization that change the canonical string for existing routes
- changes to node matching semantics for existing token forms
- changes to receiver eligibility rules or initial-route behavior
- changes to activation rules for TEST / STAY / MOVE
- changes to hook ordering rules
- changes to sender triggering rules or compatibility checks

## 2. What counts as a backward-compatible addition

Treat the following as backward-compatible additions (minor bump):
- adding new token forms while preserving the semantics of the existing required forms
- adding optional trace fields in conformance mode (without changing existing comparisons)
- adding new recommended ordering tie-breakers that do not change the ordering for existing equal keys

## 3. Recommended profile identifiers

A profile identifier should be stable, machine-readable, and easy to include in traces and DNA.

Examples:
- `summoner-standard-profile/0.1`
- `summoner-standard-profile/1.0`

If you publish the profile as releases, also include the git tag:
- `summoner-standard-profile/0.1 (tag v0.1.0)`

## 4. Declaring conformance

A simple conformance declaration can include:
- profile identifier and version
- supported tape shapes (single, many, index-single, index-many)
- supported arrow styles (if you support multiple)
- whether conformance mode forces determinism in scheduling
- any declared nondeterminism and how it is handled in trace comparison
