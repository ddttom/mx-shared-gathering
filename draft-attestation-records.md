---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Attestation Records and Verification note"
docname: draft-cranstoun-mx-attestation-records
date: 2026-05-15
consensus: false
keyword:
  - mx
  - attestation
  - registry
  - verification
  - signing
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-attestation-records.md
---

# MX Attestation Records and Verification note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 15 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies the **attestation record** — a cog produced by an MX trust layer that wraps another cog with cryptographic apparatus. The attestation record references the attested cog by ID and digest; the attested cog is unchanged. A cog can be unattested, attested by one trust layer, or attested independently by several, without any change to its schema or bytes.

The note defines the record's fields, the relationship between the attestation and the cog it covers, and the verification model: trust layers **produce** attestations on submission and **serve** them on retrieval; verification is a client-side operation against existing standards. The registry serves; verifiers verify.

The signing and canonicalisation mechanics referenced here are inherited unchanged from the **MX Contract Fingerprinting and Signing note**; this note specifies only the record format and the lifecycle around it.

---

## 2. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) when, and only when, they appear in all capitals.

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of the draft set). That pattern governs the structural template for every field; this note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

A trust layer or verifier claiming conformance to this note SHALL satisfy:

- **Level 1 (record format)** — Produced attestation records carry the §4.2 fields, well-formed; absent fields are not silently defaulted.
- **Level 2 (verification correctness)** — Level 1 plus: a verifier resolves the attested cog by ID and digest, recomputes the fingerprint per the MX Contract Fingerprinting and Signing note §5, and surfaces any mismatch as an explicit failure.
- **Level 3 (independent verifiability)** — Level 2 plus: the attestation record can be verified offline given the attested cog, the record, and the public key material referenced by `issuedBy`, with no live call to the issuing trust layer.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Implementations SHOULD note the draft version they implemented against.

---

## 3. Scope

### 3.1 In scope

- The fields of an attestation record (§4.2).
- The reference direction between an attestation and the cog it attests (§4.1).
- The lifecycle around production, retrieval, and verification (§4.3–§4.5).
- The batching mechanism for high-volume attestation (§4.6), inheriting Merkle-tree mechanics from existing publications.

### 3.2 Out of scope

- The fingerprint algorithm itself — defers to the **MX Contract Fingerprinting and Signing note** §5.
- Signature algorithms (Ed25519, ECDSA, RSA, post-quantum) — defer to [W3C VC Data Integrity](https://www.w3.org/TR/vc-data-integrity/), [JWS (RFC 7515)](https://datatracker.ietf.org/doc/html/rfc7515), or [COSE (RFC 9052)](https://datatracker.ietf.org/doc/html/rfc9052).
- DID method specifics for `issuedBy` — defer to [W3C DID Core](https://www.w3.org/TR/did-core/).
- Key management, distribution, rotation, and revocation infrastructure — open questions (§7).
- Compliance claims about external standards — owned by the **MX Compliance Claims note**.

### 3.3 Relationship to existing standards

| Standard | Relationship |
|----------|--------------|
| **MX Contract Fingerprinting and Signing note** | Defines the canonicalisation and signing model this note builds on. The attestation record carries the signature; §5 of the sister note specifies what was signed. |
| **MX Compliance Claims note** | Adjacent: a compliance claim *is* a signed cog and MAY itself be wrapped in an attestation record by a trust layer. |
| [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) | Date and time format used by `issuedAt`. |
| [W3C DID Core](https://www.w3.org/TR/did-core/) | Identifier format used by `issuedBy` and by the key reference inside `signature`. |
| [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962) | Certificate Transparency Merkle-tree mechanics — referenced by §4.6 batching. |

---

## 4. Attestation records and verification

### 4.1 Position

An attestation record is produced by a trust layer — an MX registry operator — and wraps a cog with cryptographic apparatus. The cog itself is unchanged. The attestation references the cog by ID and digest.

The cog is data. The attestation is the trust wrapper. The two are deliberately separated (§3.1): a cog can be unattested, attested by one registry, or attested by a different registry, without any change to the cog's schema or bytes.

A trust layer does not verify attestations. It produces them on submission and serves them on retrieval. Verification is a client-side operation against existing standards. The registry serves; verifiers verify.

### 4.2 Attestation record fields

An attestation record is itself a cog. It declares `type: attestation` at the top level (Zone 1b, per the **MX Core Metadata note**). It carries `cogUrn` (the universal identifier of the attestation record itself, in the trust layer's namespace) and `schemaVersion` per the **MX Cogs note** §6.5.3 and §6.5.4. This note specifies the attestation-specific top-level fields below.

| Field | Semantics |
|-------|-----------|
| `attests` | Reference to the cog being attested. Object with `cogUrn` (the attested cog's universal identifier) and `digest` (the SHA-256 of the attested cog's JCS-canonicalised bytes). Batch-root attestations replace `cogUrn`/`digest` with `merkleRoot`, `leafCount`, and `hashFunction`; see §4.6. |
| `issuedBy` | DID of the trust-layer operator (the registry instance that produced the attestation). |
| `issuedAt` | [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) timestamp of attestation. |
| `mechanism` | One of `individual` or `merkleBatch`; see §4.6 for batching. |
| `signature` | When `mechanism: individual` (and for batch-root attestations under `merkleBatch`): signature over the attested digest, with algorithm identifier and key reference (DID URL). |
| `batch` | When the record is a per-cog leaf under `merkleBatch`: a `rootAttestation` reference, a `leafIndex`, and an inclusion `proof` array (see §4.6). |

The attestation references the cog by `cogUrn` and digest. The cog does not reference the attestation. This direction matters: a cog can exist without any attestation, and multiple independent attestations of the same cog by different trust layers can coexist without any of them being privileged inside the cog itself.

### 4.3 Production

*(To be drafted: the lifecycle of producing an attestation on submission to a trust layer — what the trust layer MUST check, what it MUST refuse, and what it serialises into the attestation record.)*

### 4.4 Retrieval

*(To be drafted: how a verifier discovers attestations for a given cog. Discovery patterns expected to cover: well-known registry endpoints, sidecar files, in-cog hint fields pointing outward to the registry.)*

### 4.5 Verification

*(To be drafted: the verifier's algorithm — fetch attested cog, recompute fingerprint per the MX Contract Fingerprinting and Signing note §5, resolve `issuedBy` key material, verify signature, surface any mismatch as explicit failure.)*

### 4.6 Batching: `merkleBatch`

*(To be drafted: Merkle-tree batching for high-volume issuance — when one signed root covers a batch of attested cogs and each individual attestation carries an inclusion proof. Modelled on [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962) Certificate Transparency.)*

---

## 5. *(Reserved for §5)*

*(No content yet. Drop this heading before publishing if no §5 is needed.)*

---

## 6. *(Reserved for §6)*

*(No content yet. Drop this heading before publishing if no §6 is needed.)*

---

## 7. Open questions

The following remain open for community discussion:

1. **Key rotation** — when the key material referenced by `issuedBy` rotates, how does a verifier distinguish a rotation from a revocation, and how are historical attestations validated against retired keys?
2. **Revocation** — how does a verifier discover that a previously valid attestation has been revoked? CRL, OCSP, on-chain registry, an MX registry endpoint?
3. **Cross-registry attestation** — when two trust layers independently attest the same cog, MAY a verifier treat them as additively strengthening trust, or are they always independent claims?
4. **Trust-layer discovery** — how does a verifier discover *which* trust layers have attested a given cog without prior knowledge of those trust layers?

---

## 8. Implementation guidance (informative)

*This section is informative; it is not a normative part of the standard.*

### 8.1 Producing an attestation

*(To be drafted.)*

### 8.2 Verifying an attestation offline

*(To be drafted: the canonical verification pipeline — fetch cog, recompute fingerprint, resolve DID, verify signature, surface result.)*

### 8.3 Verification

*(To be drafted: shell-level commands or other independently runnable checks a verifier can use to confirm conformance.)*

---

## 9. Security and privacy considerations

- An attestation record asserts only that the trust-layer operator named in `issuedBy` produced the record at `issuedAt` and that the attested digest matches the cog at that moment. It does not assert that the cog's values are factually correct, sound, or current. The narrowness is intentional and inherits the rationale from the **MX Contract Fingerprinting and Signing note** §9.
- A trust layer that produces attestations without first checking the cog passes the mandatory-fields check in the **MX Contract Fingerprinting and Signing note** §4.3 produces records whose semantics are degraded; verifiers presented with such records cannot tell from the record alone whether the check was performed. Trust layers SHOULD record, and MAY surface in the attestation, evidence that the pre-signature review pipeline ran.
- The attestation record contains the attested cog's digest, the issuer DID, and a timestamp. A verifier in possession of the record alone can learn which cog was attested, by whom, and when, even without access to the cog body. This is a deliberate property of the format — it is also a privacy consideration in contexts where the attested cog is private.
- Batching mechanics (§4.6) trade per-cog signature cost against the risk that compromise of a batch root invalidates every member of the batch. Trust layers using `merkleBatch` SHOULD document their batch size policy and the consequences of a compromised root.

---

## 10. IANA considerations

This note proposes no new IANA registrations.

---

## 11. References

### 11.1 Normative references

- [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119), [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) — keywords for use in RFCs to indicate requirement levels.
- [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) — Date and Time on the Internet: Timestamps.
- [W3C DID Core](https://www.w3.org/TR/did-core/) — Decentralised Identifiers (DIDs) v1.0.
- **MX Contract Fingerprinting and Signing note** — canonicalisation, fingerprint, and signing model on which this note builds.
- **MX Core Metadata note** — Zone 1 / Zone 2 frontmatter, `contentType`, `version`.

### 11.2 Informative references

- [W3C Verifiable Credentials Data Integrity](https://www.w3.org/TR/vc-data-integrity/) — compatible signing model.
- [JWS RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) — JSON Web Signature.
- [COSE RFC 9052](https://datatracker.ietf.org/doc/html/rfc9052) — CBOR Object Signing and Encryption.
- [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962) — Certificate Transparency, the model behind §4.6 batching.
- [C2PA Content Credentials](https://c2pa.org/) — compatible model for signing assertions about content provenance.

---

*End of MX Attestation Records and Verification note draft.*
