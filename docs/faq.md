# FAQ

## What is included in the current standards set?

The current draft standardizes two profiles:
- the core execution profile
- the `id.v1` agent identity profile

It leaves extended sender orchestration outside the current review release.

## Does this standards set require using the Summoner SDK?

No. Both profiles are written for implementation-independent conformance.

## Why standardize Summoner at all?

Because Summoner already exposes route structure, state admission, receiver outcomes, and portable identity in a way that can be reviewed and compared. The standard exists to turn those explicit semantics into a stable public contract instead of leaving them as SDK-specific convention.

## Why does the core profile stop at receiver-triggered untimed sender admission?

Because that is the smallest sender boundary that is easy to observe, compare, and reproduce across implementations. More elaborate sender orchestration mixes portable behavior with runtime policy such as batching, timers, guard callables, event-carried data, and sender-owned data handoff.

## Where do the current public implementations live?

The current public implementation references are:
- [`Summoner-Network/summoner-core`](https://github.com/Summoner-Network/summoner-core) for the core SDK behavior
- [`Summoner-Network/extension-agentclass`](https://github.com/Summoner-Network/extension-agentclass) for the Aurora identity implementation

Those repositories are useful for code review and context, but they are not normative sources for the standard.

## Does the standard include agent identity?

Yes. The separate `id.v1` agent identity profile standardizes the portable public identity record, its canonical signing bytes, verification rules, fingerprint derivation, and continuity semantics.

## Does the identity profile standardize local identity files?

No. The normative identity profile covers the portable public record. Aurora's local identity-file format is documented as an implementation note, not as a conformance requirement.

## How should I think about `id.v1` relative to SSI, TLS, or DID?

`id.v1` is best understood as a self-signed, self-sovereign-style, TLS-inspired agent identity profile.

It is specified directly as its own Summoner profile. It can coexist with registries, directories, and DID-based systems, but readers should not assume DID-specific identifier syntax, DID-document structure, or DID-resolution rules unless a future profile defines them.

## Does the core profile include extended sender orchestration features?

No. The current core profile stops at receiver-triggered untimed sender admission. Event-carried sender data, sender data-transfer policy, and timed or continuing sender modes remain outside this review release.

## Why does the core profile mention `multi` but leave `use_data` outside the profile?

Because `multi` is a sender-local emission rule, while `use_data` changes the receiver-event-sender contract itself.

`multi` only affects how many payloads an already admitted sender invocation may emit. `use_data` introduces receiver-attached event payloads, sender argument passing, payload transfer policy, and per-event invocation semantics. That broader contract belongs in a separate profile if it is standardized.

## Does core DNA mean the raw `summoner-core` DNA export?

No. The core profile defines a projected, interoperable DNA view over receivers, senders, and hooks. SDK-specific DNA exports may include source code, state adapters, context headers, or extension fields beyond the core view.

## What is the minimum needed to claim core conformance?

At minimum:
- implement parsing and canonicalization rules
- implement the required node matching semantics
- implement the tape model and receiver semantics as specified
- implement the receiver-triggered untimed sender boundary as specified
- provide a conformance mode with trace-level observables suitable for comparison

## What is the minimum needed to claim identity-profile conformance?

At minimum:
- generate and/or verify `id.v1` public identity records
- reconstruct the canonical public core exactly
- verify Ed25519 signatures over the canonical public core
- derive fingerprints exactly as specified
- reject unsupported identity versions

## What is the intended path from draft to stronger standard status?

Typical steps:
- stabilize the profile boundary and change-control process
- publish test vectors and a reference comparator
- obtain at least one independent implementation per profile
- promote draft review releases into stable profile releases
