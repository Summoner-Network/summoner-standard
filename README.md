# Summoner Standard Profile (Draft v0.1.0)

This repository hosts a draft standard profile for the interoperability-critical semantics of Summoner.

The profile focuses on:
- route grammar and parsing (including canonicalization)
- node matching semantics (gating and acceptance)
- tape model (single and indexed)
- receive/send execution semantics (hooks, eligibility, activation, triggering)
- a conformance boundary suitable for implementation-independent testing

Out of scope:
- transport protocols
- deployment or hosting
- UI tooling
- application-specific workflows
- cryptography (this profile defines control points but does not standardize crypto)

## Status

**Draft v0.1.0**

This is an open draft intended to support:
- precise interoperability discussions
- trace-level conformance tests
- independent implementations

## Version

- **Profile:** `summoner-standard-profile/0.1` (draft series)
- **Release tag:** `v0.1.0`

See [`docs/versioning.md`](docs/versioning.md) for the versioning policy.

## Read the spec

- **Rendered site (GitHub Pages):** see `/docs` once Pages is enabled
- **Core spec (normative):** [`docs/core-semantics.md`](docs/core-semantics.md)
- **Supplementary note (informative):** [`docs/supplementary-note.md`](docs/supplementary-note.md)
- **Conformance notes:** [`docs/conformance.md`](docs/conformance.md)
- **Versioning policy:** [`docs/versioning.md`](docs/versioning.md)
- **FAQ:** [`docs/faq.md`](docs/faq.md)

## Repository structure

- `docs/`
  - `index.md`: landing page for GitHub Pages
  - `core-semantics.md`: normative profile specification
  - `supplementary-note.md`: operational and compositional context
  - `conformance.md`: conformance boundary and trace guidance
  - `versioning.md`: profile versioning rules
  - `faq.md`: common questions and non-goals

## Intended use

If you are implementing the profile:
1. Implement the normative sections in `docs/core-semantics.md`.
2. Expose a conformance mode that emits trace-level observables (parsing, matching, execution, emission).
3. Compare outputs against the conformance expectations described in `docs/conformance.md`.

## Contributions

Discussion and change proposals are welcome:
- Use GitHub Issues for questions, ambiguities, and proposed clarifications.
- Use Pull Requests for text changes with a clear rationale.
- When a change affects interoperability semantics, it must include:
  - the motivation
  - the exact observable behavior change
  - the expected versioning impact (major vs minor)

## Licensing

[Apache 2.0](/LICENSE)

## Contact

Maintainer: 
```
Rémy Tuyéras (Summoner Corp.)
rtuyeras@summoner.org
```

