# Summoner Standard Profile (Draft v0.1.0)

This site publishes a draft standard profile for the interoperability-critical semantics of Summoner.

**Status:** Draft v0.1.0.  
**Date:** January 2026  

## Version

- **Profile:** `summoner-standard-profile/0.1` (draft series)
- **Release tag:** `v0.1.0`

## What this profile standardizes

This profile defines:
- route grammar, parsing, and canonical string form
- node matching semantics used for gating and compatibility checks
- the tape model (single and indexed variants)
- deterministic execution semantics for:
  - hook ordering and drop behavior
  - receiver eligibility and ordering
  - tape activation rules for TEST / STAY / MOVE
  - sender triggering and ordering
- a portable DNA representation for enumerating registrations
- conformance requirements suitable for implementation-independent testing

## What this profile does not standardize

This profile does not standardize:
- transport, networking, or deployment
- UI and tooling
- application workflows
- cryptography (the profile defines control points relevant to security review, not cryptographic primitives)

## Documents

- **Core Semantics and Conformance Requirements (normative)**  
  [`core-semantics.md`](core-semantics.md)

- **Operational Semantics and Compositional Structure (supplementary, informative)**  
  [`supplementary-note.md`](supplementary-note.md)

- **Conformance notes (trace-level guidance)**  
  [`conformance.md`](conformance.md)

- **Versioning policy**  
  [`versioning.md`](versioning.md)

- **FAQ**  
  [`faq.md`](faq.md)

## Audience

This profile is written for:
- implementers who want interoperability with compatible behavior
- reviewers who need a crisp description of observable semantics
- teams designing conformance tests and trace comparators

## Feedback

Feedback is welcome via Issues and Pull Requests in the repository.
