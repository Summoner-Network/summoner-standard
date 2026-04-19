# Summoner Standard Profiles

**Webpage:** [https://summoner-network.github.io/summoner-standard](https://summoner-network.github.io/summoner-standard)

This repository hosts a draft standards set for the interoperability-critical parts of Summoner.

The current review release intentionally standardizes two portable boundaries:
- the **core execution profile** for routes, matching, tapes, receiver processing, and receiver-triggered untimed sender admission
- the **agent identity profile** for portable self-signed public identity records (`id.v1`)

## Why standardize Summoner?

Summoner is valuable because it makes agent coordination explicit where many systems leave it implicit:
- routes make admission logic visible
- tape state makes runtime context inspectable
- receiver outcomes make control flow traceable
- portable public identity makes continuity reviewable

A standard is useful because it turns those semantics from SDK convention into a public contract. That helps:
- users understand what “compatible Summoner behavior” actually means
- reviewers evaluate a bounded semantics instead of reverse-engineering one runtime
- power testers build black-box compatibility and trace comparators
- implementers reproduce the behavior in another language or runtime

The intent is not to freeze innovation. The intent is to stabilize the parts that need to be shared so implementations can diverge internally without losing interoperability.

## Design approach

This standards set follows a few simple design rules:
- standardize **portable observables**, not one scheduler, queue layout, or decorator API
- keep the first release **narrow and layered**, rather than forcing every runtime feature into one document
- separate **core execution semantics** from **public identity semantics**
- defer features that change the public receiver-event-sender contract until they can stand as their own justified extension profile

## Current profile set

| Profile | Identifier | Purpose | Status |
| --- | --- | --- | --- |
| Core | `summoner-core-profile/0.1` | Execution semantics and conformance traces | Draft for review |
| Agent identity | `summoner-agent-identity-profile/id.v1` | Portable public agent identity and verification | Draft for review |

## Deferred from this review release

The following areas are intentionally not standardized yet:
- the richer sender orchestration extensions currently present in the `summoner-core` 1.2.x line, including `Event.data`, `use_data`, `data_mode`, `every`, and `run_while`
- transport protocols, deployment, or hosting
- UI tooling
- application-specific workflows
- registry-backed trust policy and higher-level identity governance

## Status

**Standards set release:** `v0.2.0`

This draft is aimed at:
- users who want a crisp description of stable interoperability surfaces
- reviewers who need a bounded, security-reviewable standard
- power testers building trace comparators and compatibility vectors
- independent implementations looking for a portable baseline

## Read the docs

- **Landing page:** [`docs/index.md`](docs/index.md)
- **Core profile (normative):** [`docs/core-semantics.md`](docs/core-semantics.md)
- **Agent identity profile (normative):** [`docs/identity-profile.md`](docs/identity-profile.md)
- **Conformance guidance (informative):** [`docs/conformance.md`](docs/conformance.md)
- **Supplementary note (informative):** [`docs/supplementary-note.md`](docs/supplementary-note.md)
- **Versioning policy:** [`docs/versioning.md`](docs/versioning.md)
- **FAQ:** [`docs/faq.md`](docs/faq.md)

## Related implementation repositories

These repositories are informative implementation references. They help readers locate the current public codebases behind the standard, but they are not normative sources for conformance.

- **Core SDK reference:** [`Summoner-Network/summoner-core`](https://github.com/Summoner-Network/summoner-core)
- **Aurora identity reference:** [`Summoner-Network/extension-agentclass`](https://github.com/Summoner-Network/extension-agentclass)

## Intended use

If you are reviewing or implementing this standards set:
1. Read the landing page and confirm the profile boundary matches your use case.
2. Treat `docs/core-semantics.md` and `docs/identity-profile.md` as the normative review documents.
3. Use `docs/conformance.md` to design trace-level and record-level compatibility tests.
4. Treat sender orchestration beyond the core profile as deliberately deferred unless and until a separate extension profile is published.

## Contributions

Discussion and change proposals are welcome:
- Use GitHub Issues for questions, ambiguities, and proposed clarifications.
- Use Pull Requests for text changes with a clear rationale.
- When a change affects interoperability semantics, include:
  - the motivation
  - the exact observable behavior change
  - the profile impacted
  - the expected versioning impact

## Licensing

[Apache 2.0](/LICENSE)

## Contact

Maintainers:

- Rémy Tuyéras (CTO, Summoner Corp.) | rtuyeras@summoner.org
