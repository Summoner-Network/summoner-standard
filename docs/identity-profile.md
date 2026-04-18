# Agent Identity Profile (`id.v1`)

**Profile ID:** `summoner-agent-identity-profile/id.v1`<br>
**Standards set release:** `v0.2.0`<br>
**Date:** April 2026<br>
**Status:** Draft for review. Aligned to the Aurora `id.v1` implementation.

## 1. Scope

This document defines the portable public agent identity boundary for Summoner.

It standardizes:
- the self-signed public identity record exchanged between peers
- the canonical JSON bytes used for signing and verification
- the cryptographic algorithms and encodings required for interoperability
- fingerprint derivation
- identity continuity semantics relevant to higher-level protocols

This profile intentionally does not standardize:
- local identity-file persistence and password-encryption policy
- registry-backed trust, directory binding, or DID resolution
- session, envelope, or message-encryption semantics
- governance policy about when an organization should trust `meta`

A conforming implementation MAY provide those features, but identity-profile conformance is determined by the portable public record defined here.

## 2. Model and terminology

In this profile, the portable **agent ID** is the signed public identity record itself. It is not a separate application label.

Three related surfaces should be distinguished:

| Surface | Purpose | Portable between peers? | Authenticated by `sig`? |
| --- | --- | --- | --- |
| Public identity record | The identity object exchanged and verified between peers | Yes | Yes |
| Fingerprint | Short local handle derived from `pub_sig_b64` | Yes | Derived from authenticated data |
| Local identity file | Private persistence container for local keys | No | Outside this profile |

The public identity record is the only identity surface standardized here.

## 3. Public identity record

A conforming public identity record is a JSON object with the following fields:

| Field | Required | Type | Meaning |
| --- | --- | --- | --- |
| `created_at` | Yes | string | Identity creation timestamp |
| `pub_enc_b64` | Yes | string | Standard Base64 of the raw 32-byte X25519 public key |
| `pub_sig_b64` | Yes | string | Standard Base64 of the raw 32-byte Ed25519 public key |
| `meta` | No | any JSON value | Optional signed metadata |
| `sig` | Yes | string | Standard Base64 of the raw 64-byte Ed25519 signature |
| `v` | Yes | string | MUST equal `id.v1` |

Generators SHOULD emit `created_at` as an ISO 8601 UTC string with an explicit offset, for example `2026-04-16T17:45:03+00:00`, and SHOULD omit microseconds.

Unknown additional fields MAY be present, but they are not part of the authenticated public core in `id.v1`. Conforming verifiers MUST ignore such fields for verification decisions unless another profile explicitly standardizes them.

A minimal example:

```json
{
  "created_at": "2026-04-16T17:45:03+00:00",
  "pub_enc_b64": "<base64 raw X25519 public key>",
  "pub_sig_b64": "<base64 raw Ed25519 public key>",
  "sig": "<base64 Ed25519 signature>",
  "v": "id.v1"
}
```

## 4. Cryptographic and encoding profile

The compatibility contract for `id.v1` is:

| Purpose | Required algorithm or format |
| --- | --- |
| Signing | Ed25519 |
| Key agreement public key | X25519 |
| Text encoding | UTF-8 |
| Binary-to-text encoding | Standard Base64 with padding |
| Fingerprint hash | SHA-256 |

Key and signature material MUST use raw byte form before Base64 encoding:

| Value | Raw length | Encoded form |
| --- | --- | --- |
| X25519 public key | 32 bytes | standard Base64 |
| Ed25519 public key | 32 bytes | standard Base64 |
| Ed25519 signature | 64 bytes | standard Base64 |

This profile does not standardize private-key storage or password-based protection.

## 5. Canonical public core and signing bytes

The signature does not cover the final identity object verbatim. It covers the canonical JSON encoding of the **public core**.

The public core is the JSON object containing:
- `created_at`
- `pub_enc_b64`
- `pub_sig_b64`
- `meta`, only when `meta` is present and non-`null`

The fields `sig` and `v` are added after signing.

That means:
- omitting `meta` and setting `meta` to `null` are equivalent for signing
- changing `meta` changes the signed public identity record
- additional fields outside the public core are not authenticated by `id.v1`

Canonical signing bytes are the UTF-8 encoding of JSON serialized with:
- lexicographically sorted object keys
- compact separators with no extra whitespace
- standard JSON escaping

An implementation compatible with the Aurora reference must behave as if it ran:

```python
json.dumps(obj, separators=(",", ":"), sort_keys=True).encode("utf-8")
```

To preserve cross-language compatibility, implementations SHOULD avoid floats, implementation-specific map ordering, or other JSON forms that can serialize differently across runtimes when they populate `meta`.

## 6. Generation and verification

To generate a conforming `id.v1` public identity record:
1. produce or load an Ed25519 signing key pair and an X25519 key-agreement key pair
2. construct the public core defined in Section 5
3. sign the canonical public-core bytes with the Ed25519 private key
4. emit the public identity record by adding `sig` and `v == "id.v1"`

A conforming verifier MUST:
1. require the input to be a JSON object
2. require `v == "id.v1"`
3. require the fields `created_at`, `pub_enc_b64`, `pub_sig_b64`, and `sig`
4. reconstruct the canonical public core exactly as in Section 5
5. Base64-decode `pub_sig_b64` and `sig`
6. verify the Ed25519 signature over the canonical public-core bytes

A verifier MUST reject unsupported versions, invalid Base64, invalid key material, invalid signature lengths, and failed signature checks.

## 7. Fingerprint derivation

The profile defines a short fingerprint derived from `pub_sig_b64`.

The fingerprint is computed by:
1. Base64-decoding `pub_sig_b64`
2. hashing the raw Ed25519 public key with SHA-256
3. Base64url-encoding the hash
4. removing trailing `=`
5. taking the first 22 characters

The fingerprint is useful for local indexing, audit references, storage keys, and UI-sized identifiers.

The fingerprint is not the full identity. Peers MUST verify the signed public identity record, not compare fingerprints alone.

## 8. Identity continuity semantics

This profile distinguishes between the public identity record and the derived fingerprint.

| Change | Same fingerprint? | Same public identity record? | Same continuity boundary? |
| --- | --- | --- | --- |
| Update `meta` only | Yes | No | Yes |
| Update `created_at` only | Yes | No | No |
| Rotate Ed25519 signing key | No | No | No |
| Rotate X25519 encryption key only | Yes | No | No |
| Rotate both keys | No | No | No |

For stable identity lifecycle:
- `created_at` SHOULD be written once and preserved
- `meta` MAY evolve over time
- any key rotation SHOULD be treated as a deliberate identity-boundary event unless a higher-level migration protocol says otherwise

## 9. Conformance requirements

A conforming implementation for this profile MUST provide behavior sufficient to validate:
- public record schema and version handling
- canonical public-core reconstruction
- signature verification
- fingerprint derivation

Required conformance properties:
- **Record conformance:** Valid `id.v1` records are accepted and invalid ones are rejected.
- **Canonicalization conformance:** The canonical public-core bytes are stable for the same logical record.
- **Fingerprint conformance:** The derived fingerprint matches Section 7 exactly.
- **Version conformance:** Unsupported `v` values are rejected rather than silently coerced.

See [`conformance.md`](conformance.md) for practical test guidance.

## 10. Security considerations

`id.v1` is a self-signed public identity profile.

It proves that:
- the record is internally consistent
- the holder of the Ed25519 private key signed the public core
- the key-agreement public key and signed metadata are bound into the same public identity object

It does not, by itself, prove:
- that the identity belongs to a specific legal or organizational entity
- that the metadata in `meta` is trusted by a third party
- that a registry, directory, or allowlist recognizes the identity

If stronger real-world binding is required, implementations SHOULD add an external trust policy such as pinning, allowlists, or directory-backed verification.

## 11. Aurora implementation note

The current public implementation reference for Aurora identity is [`Summoner-Network/extension-agentclass`](https://github.com/Summoner-Network/extension-agentclass). That repository is informative, not normative: conformance to `id.v1` is defined by this profile, not by one implementation snapshot.

The current Aurora SDK also defines a local identity-file format with:
- outer `v == "id.v1"`
- plaintext or encrypted private sections
- `scrypt` as the key derivation function
- `AES-256-GCM` for private-section encryption
- associated data `summoner/identity_file/v1`

That local persistence format is useful for SDK interoperability review, but it is not required to claim conformance to the public `id.v1` identity profile defined here.
