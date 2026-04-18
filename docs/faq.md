# FAQ

## What is included in the current standards set?

The current draft standardizes two profiles:
- the core execution profile
- the `id.v1` agent identity profile

It intentionally does not standardize the full sender orchestration runtime from the `summoner-core` 1.2.x line.

## Does this standards set require using the Summoner SDK?

No. Both profiles are written for implementation-independent conformance.

## Why standardize Summoner at all?

Because Summoner already exposes route structure, state admission, receiver outcomes, and portable identity in a way that can be reviewed and compared. The standard exists to turn those explicit semantics into a stable public contract instead of leaving them as SDK-specific convention.

## Why is full send not standardized yet?

Because the current sender runtime mixes portable observables with runtime-specific orchestration choices such as batching, timers, guard callables, and sender-owned data handoff. The current review release standardizes only the narrower receiver-triggered untimed sender boundary inside the core profile.

## Where do the current public implementations live?

The current public implementation references are:
- [`Summoner-Network/summoner-core`](https://github.com/Summoner-Network/summoner-core) for the core SDK behavior
- [`Summoner-Network/extension-agentclass`](https://github.com/Summoner-Network/extension-agentclass) for the Aurora identity implementation

Those repositories are useful for code review and context, but they are not normative sources for the standard.

## Does the standard include agent identity?

Yes. The separate `id.v1` agent identity profile standardizes the portable public identity record, its canonical signing bytes, verification rules, fingerprint derivation, and continuity semantics.

## Does the identity profile standardize local identity files?

No. The normative identity profile covers the portable public record. Aurora's local identity-file format is documented as an implementation note, not as a conformance requirement.

## Is `id.v1` a DID?

No. `id.v1` is a self-signed public identity record profile. It can coexist with registries, directories, or DID-based systems, but it does not require DID resolution or DID-document semantics.

## Does the core profile standardize `Event.data`, `use_data`, `data_mode`, `every`, or `run_while`?

No. Those richer sender orchestration features are intentionally deferred from the current standards set.

## Why does the core profile mention `multi` but defer `use_data`?

Because `multi` is a sender-local emission rule, while `use_data` changes the receiver-event-sender contract itself.

`multi` only affects how many payloads an already admitted sender invocation may emit. `use_data` introduces receiver-attached event payloads, sender argument passing, payload transfer policy, and per-event invocation semantics. That broader contract deserves a separate extension profile if it is standardized.

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
