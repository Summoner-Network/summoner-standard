# FAQ

## What does "standard profile" mean here?

It means a narrow specification of interoperability-critical semantics that can be tested across independent implementations, without standardizing transport, deployment, or application workflows.

## Does this profile require using the Summoner SDK?

No. The profile is intended to support implementation-independent conformance testing. You can implement the semantics in another runtime and still conform, as long as the observables match.

## Does this profile standardize transport?

No. Transport is explicitly out of scope.

## Does this profile standardize cryptography?

No. The profile defines control points relevant to security review (hooks, drop semantics, DNA as an audit surface), but it does not standardize cryptographic primitives or key management.

## What is the minimum needed to claim conformance?

At minimum:
- implement parsing and canonicalization rules
- implement the required node matching semantics
- implement the tape model and execution semantics as specified
- provide a conformance mode that exposes trace-level observables suitable for comparison

## Why is "DNA" included?

DNA is a portable serialization of registered behavior. Within this profile, it is defined as a deterministic JSON document enumerating receivers, senders, and hooks with metadata sufficient for debugging and audit.

## What is the intended path from draft to stronger standard status?

Typical steps:
- publish stable versions and changelogs
- add test vectors and a reference trace comparator
- obtain at least one independent implementation that passes the conformance suite
- establish a clear change-control process for profile evolution
