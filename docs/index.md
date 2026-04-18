# Summoner Standard Profiles

This site publishes a draft standards set for the parts of Summoner that are strong candidates for implementation-independent interoperability.

**Standards set release:** `v0.2.0`<br>
**Date:** April 2026<br>
**Status:** Draft for review

The current review release is intentionally conservative. It standardizes the portable core execution boundary and the portable public identity boundary. It does not yet standardize the richer sender orchestration runtime from the `summoner-core` 1.2.x line.

## Current profiles

| Profile | Identifier | What it standardizes | Main document |
| --- | --- | --- | --- |
| Core profile | `summoner-core-profile/0.1` | routes, matching, tapes, receiver processing, receiver-triggered untimed sender admission, and the core DNA view | [`core-semantics.md`](core-semantics.md) |
| Agent identity profile | `summoner-agent-identity-profile/id.v1` | the self-signed public identity record, canonical signing bytes, verification, fingerprint derivation, and continuity semantics | [`identity-profile.md`](identity-profile.md) |

## Why a standard helps

Summoner already makes important parts of agent behavior explicit:
- route structure
- state admission and activation
- receiver outcomes
- portable public identity

The reason to standardize is to keep those explicit structures portable. A standard turns them into a shared contract for users, reviewers, power testers, and future independent implementations.

The point is not to force one runtime architecture. The point is to preserve the semantics that should remain comparable even when the internal implementation changes.

## Design approach

This review set is intentionally designed to:
- standardize externally observable behavior rather than internal queue or scheduler mechanics
- use layered profiles instead of one monolithic specification
- standardize only what can be described cleanly and tested independently
- defer richer sender-runtime features until they can justify a separate extension profile

## Why this review set is narrow

This standards set focuses on behavior that is:
- portable across independent implementations
- externally observable at a conformance boundary
- precise enough for reviewers and power testers to compare

That is why the current set includes:
- the core execution semantics
- the public identity object

and defers:
- richer sender orchestration features such as `Event.data`, `use_data`, `data_mode`, `every`, and `run_while`
- transport and deployment choices
- registry-backed trust and governance policy

## Related implementation repositories

These repositories are informative reference points for the current public implementations behind the standard. They are useful for orientation and code review, but they are not normative parts of the profile definitions.

- **Core SDK reference:** [`Summoner-Network/summoner-core`](https://github.com/Summoner-Network/summoner-core)
- **Aurora identity reference:** [`Summoner-Network/extension-agentclass`](https://github.com/Summoner-Network/extension-agentclass)

## Documents

- **Core profile (normative):** [`core-semantics.md`](core-semantics.md)
- **Agent identity profile (normative):** [`identity-profile.md`](identity-profile.md)
- **Conformance guidance (informative):** [`conformance.md`](conformance.md)
- **Supplementary note (informative):** [`supplementary-note.md`](supplementary-note.md)
- **Versioning policy:** [`versioning.md`](versioning.md)
- **FAQ:** [`faq.md`](faq.md)

## Review focus

This draft is written for:
- users who want a stable contract for what Summoner standardizes today
- reviewers checking whether the standard boundary is coherent and justified
- power testers building route, trace, and identity compatibility vectors
- future independent implementations

Useful review questions:
- Is the core profile precise enough to compare runtimes without overfitting to one SDK?
- Is the `id.v1` public identity record a clean portable boundary?
- Are the deferred sender-runtime features correctly left outside the first standard review release?
