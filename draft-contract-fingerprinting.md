---
title: "MX Contract Fingerprinting and Signing note"
docname: draft-cranstoun-mx-contract-fingerprinting
date: 2026-04-27
consensus: false
keyword:
  - mx
  - signing
  - fingerprint
  - contract
  - integrity
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Contract Fingerprinting and Signing note

**Version:** 1.5-draft
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

**Signing is optional.** Most cogs ship unsigned. This note specifies the contract a cog satisfies *when it elects to be signed*; it does not require a cog to be signed.

When a cog is signed, two frontmatter fields make the signature scope explicit. `contractFields` lists top-level keys covered by the signature. `metadataFields` lists keys explicitly excluded (timestamps, tags, registry annotations) — values that may change without invalidating the signature.

This note defines only the **fingerprint** — the deterministic byte sequence over which a signature is produced (canonical JSON projection of the contract keys, hashed with SHA-256). Signature algorithm, key management, and transport are out of scope.

The signature attests **provenance and integrity, not truth**: a verifier confirms that the cog's contract surface was last in this exact state when the named signer applied the signature, and nothing more. Section 9 explains why the narrower claim is deliberate.

---

## 2. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) when, and only when, they appear in all capitals.

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field — heading, property table, definition prose, example — and is binding on this note. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

This note defines three conformance levels (modelled on the [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level A/AA/AAA pattern):

- **Level 1 (MUST)** — A cog declaring `contractFields` MUST also declare `metadataFields`. The two arrays MUST be disjoint.
- **Level 2 (SHOULD)** — A signed cog SHOULD declare both arrays. Verifiers SHOULD reject a signed cog whose declared scope cannot be reconciled with the signature.
- **Level 3 (MAY)** — Implementations MAY extend the fingerprint algorithm with additional canonicalisation passes provided they are documented in a `validatesAgainst` declaration on the cog.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Reference implementations exist in JavaScript and Rust under the `cog-spec` v1.0 codebase; their kebab-case field names (`x-mx-contract-fields`, `x-mx-metadata-fields`) will be aligned with the camelCase forms in this note in a future revision.

---

## 3. Scope

### 3.1 What this note covers

- The two fields `contractFields` and `metadataFields` (definitions in §4).
- The deterministic fingerprint algorithm (§5).
- Conformance rules and verifier expectations (§6).

### 3.2 Out of scope

- Signature algorithms (Ed25519, ECDSA, RSA, post-quantum) — defer to [W3C VC Data Integrity](https://www.w3.org/TR/vc-data-integrity/), [JWS (RFC 7515)](https://datatracker.ietf.org/doc/html/rfc7515), or [COSE (RFC 9052)](https://datatracker.ietf.org/doc/html/rfc9052).
- Key management, distribution, or rotation — defer to organisational PKI.
- Transport encoding (detached vs envelope, PEM vs raw bytes) — defer to the chosen signature standard.
- Multi-signer coordination and revocation — open question (§7).

### 3.3 Relationship to existing standards

| Standard | Relationship |
|----------|--------------|
| [W3C VC Data Integrity](https://www.w3.org/TR/vc-data-integrity/) | Compatible signing model. The MX fingerprint may be wrapped in a VC proof. |
| [JWS RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) | Compatible signing model. The MX fingerprint may be the JWS payload. |
| [COSE RFC 9052](https://datatracker.ietf.org/doc/html/rfc9052) | Compatible signing model for binary/CBOR transports. |
| [C2PA Content Credentials](https://c2pa.org/) | Compatible model for signing assertions about content provenance. |
| [FIPS 180-4](https://csrc.nist.gov/publications/detail/fips/180/4/final) | SHA-256, the hash function used by the fingerprint algorithm in §5. |
| [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259) | The JSON Data Interchange Format, used as the canonical serialisation in §5. |

---

## 4. Field definitions

### 4.1 `contractFields`

| Property | Value |
|----------|-------|
| **Type** | array of string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) when a signature is present |
| **Default** | *(none)* |

**Definition:** Frontmatter top-level keys whose values are covered by the cog's signature. Defines the contract fingerprint scope. Each entry MUST be the name of a key that exists at the top level of the cog's frontmatter.

**Example:**

```yaml
contractFields:
  - title
  - schema
  - x-mx-thresholds
  - x-mx-approvers
  - x-mx-approvalProcedure
```

**Normative notes:**

- The array MUST be disjoint from `metadataFields`.
- Every entry SHOULD reference a key that exists in the frontmatter; verifiers MAY treat stray entries as a warning.
- Entries MAY include `x-mx-` and `x-mx-p-` namespaced fields. Private fields included in `contractFields` are still covered by the signature even though their values are obfuscated.

### 4.2 `metadataFields`

| Property | Value |
|----------|-------|
| **Type** | array of string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) when `contractFields` is declared |
| **Default** | *(none)* |

**Definition:** Frontmatter top-level keys explicitly EXCLUDED from the contract fingerprint. Values of these keys may change without invalidating the signature. Typical members are timestamps, tags, registry annotations, and other non-contract metadata.

**Example:**

```yaml
metadataFields:
  - created
  - modified
  - tags
```

**Normative notes:**

- The array MUST be disjoint from `contractFields`.
- Keys not listed in either array are **implementation-defined** — verifiers MAY default unlisted keys to either category, but MUST document their choice.
- A best-practice declaration lists every top-level frontmatter key in exactly one of the two arrays.

### 4.3 Mandatory fields when signing

When a cog is signed, the following frontmatter MUST be present and well-formed. A signer MUST refuse to sign a cog that fails any of these checks; a verifier MUST refuse a signed cog whose mandatory fields are missing or malformed.

| Field | Requirement | Rationale |
|-------|-------------|-----------|
| `title` | Non-empty string at the top level. | The signed claim records what was signed; without a title there is no human-readable identity to bind to. |
| `validatesAgainst` | Non-empty array of validator names. Every entry MUST be resolvable — that is, registered with the verifying runtime — and MUST pass when executed. | The signature claims the cog passed every named validator at sign time. An unresolvable validator means the claim cannot be interpreted; a failing validator means the claim is false. |
| `schema` | Reference to the schema document that defines the cog's contract structure. If declared in frontmatter, MUST resolve to a real schema. | The schema specifies the contract shape the signed claim covers. |

These three fields together form the *minimum signed surface*. The fingerprint algorithm in §5 covers them automatically because they are top-level frontmatter keys typically named in `contractFields`.

A signer SHOULD perform these checks during a pre-signature review pass (§6.4) before producing a fingerprint. A signer that emits signatures over cogs missing any of the three is not conforming.

### 4.4 Default-excluded metadata fields

When a schema does not explicitly declare which top-level keys are part of the contract, implementations SHOULD treat the following as `metadataFields` by default — that is, exclude them from the fingerprint:

- `modified`
- `version`
- `created`
- `author`
- `updateInstructions`

These values change with editorial activity that does not alter the contract surface. Excluding them by default lets a cog be re-edited (timestamp bumped, version incremented) without invalidating its signature.

The default can be overridden via a schema declaration:

- `x-mx-contractFields` (positive list) — when present, only the named keys are part of the contract; everything else is metadata.
- `x-mx-metadataFields` (negative list) — when present, the named keys are excluded; everything else is contract.

If both are declared, the positive list (`x-mx-contractFields`) takes priority.

---

## 5. The fingerprint algorithm

### 5.1 Overview

The contract fingerprint is a deterministic byte sequence derived from the values of the keys named in `contractFields`. Two implementations applying this algorithm to the same frontmatter MUST produce identical fingerprints.

### 5.2 Steps

1. **Project.** Build an object containing only the keys listed in `contractFields`. Keys absent from the frontmatter are omitted (verifiers MAY warn).
2. **Sort.** Order keys lexicographically by Unicode code point.
3. **Canonicalise values.** For each value:
   - Strings: serialised as UTF-8 with no escaping beyond what JSON requires.
   - Numbers: serialised as their canonical JSON form.
   - Booleans: `true` or `false`.
   - Objects and arrays: recursively canonicalised by the same rules; keys of nested objects are also lexicographically sorted.
   - Null: `null`.
4. **Serialise.** Emit the canonical object as UTF-8 JSON ([RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259)) with no insignificant whitespace.
5. **Hash.** Apply SHA-256 ([FIPS 180-4](https://csrc.nist.gov/publications/detail/fips/180/4/final)) to the canonical byte sequence. The 32-byte digest is the fingerprint.

The signature is then produced by signing the fingerprint bytes with the chosen signature algorithm. The signature MAY be carried in any envelope format (JWS, COSE, detached signature, sidecar file).

### 5.3 Worked example

Given:

```yaml
title: Approve invoices below 1000
schema: ./schemas/invoice-approval.v1.yaml
x-mx-thresholds:
  autoApproveBelow: 1000
contractFields: [title, schema, x-mx-thresholds]
metadataFields: [created, modified, tags]
```

The projection is:

```json
{"schema":"./schemas/invoice-approval.v1.yaml","title":"Approve invoices below 1000","x-mx-thresholds":{"autoApproveBelow":1000}}
```

(Keys sorted lexicographically: `schema`, `title`, `x-mx-thresholds`.)

The fingerprint is `SHA-256` of these bytes.

---

## 6. Conformance: signers and verifiers

### 6.1 Signer conformance

A signer:

- MUST refuse to sign a cog that fails the §4.3 mandatory-fields check (missing `title`, missing or unresolvable `validatesAgainst`, missing/invalid `schema`).
- MUST run every validator listed in `validatesAgainst` and refuse to sign if any reports failure.
- MUST treat `contractFields` and `metadataFields` as authoritative when both are declared.
- MUST treat the §4.4 default-excluded metadata field list as the fallback when neither `contractFields` nor `metadataFields` is declared and the schema does not override.

### 6.2 Verifier conformance

A verifier:

- MUST refuse a signed cog whose `contractFields` and `metadataFields` overlap.
- MUST refuse a signed cog whose declared `contractFields` cannot be reconstructed from the frontmatter (missing keys would change the projection).
- MUST refuse a signed cog missing any of the §4.3 mandatory fields, or whose `validatesAgainst` entries cannot be resolved or fail validation at verification time.
- SHOULD warn when frontmatter contains top-level keys not listed in either array.
- SHOULD log the resolved `contractFields` set alongside any verification result for transparency.
- MUST NOT silently substitute a default scope when the cog declares neither array.

### 6.3 Recommended pre-signature review pipeline (Informative)

*This subsection is informative; it describes recommended practice but is not normative.*

A conforming signer SHOULD execute a multi-phase review before producing a signature. A typical pipeline:

1. **Inventory** — enumerate all sections and top-level fields of the cog.
2. **Classify** — assign each section a layer (declarative, executable, narrative).
3. **Lift** — identify the load-bearing body sections that materially affect the contract.
4. **Declare-Contracts** — audit that `schema` and `validatesAgainst` are declared, well-formed, and reference resolvable validators (§4.3 check).
5. **Validate** — run every validator named in `validatesAgainst`. All MUST pass.
6. **Notarise** — only when phases 1–5 succeed, compute the fingerprint per §5 and produce the signature.

Implementations MAY combine, rename, or extend phases. The MUST is that a signer never produces a signature over an artefact that has not passed an equivalent of phases 4 and 5.

---

## 7. Open questions

The following are flagged for community discussion before this draft is offered for ratification:

1. **Key management** — should this note reference a default PKI model, or stay agnostic?
2. **Multi-signer** — when multiple parties sign the same cog, is each signature scoped to the same `contractFields` or may each declare its own scope?
3. **Revocation** — how does a verifier discover that a previously valid signature has been revoked? CRL, OCSP, on-chain registry, an MX registry?
4. **Schema-derived contract** — should `contractFields` be derivable automatically from a cog's `schema` reference, or remain explicit?
5. **Canonical YAML alternative** — §5 chooses canonical JSON for portability. Should an opt-in canonical-YAML form be standardised for cogs that wish to sign over their authored representation directly?

---

## 8. Security and privacy considerations

- A cog whose `contractFields` does not list every load-bearing field is **silently weakly bound** — values not in the contract may be modified after signing without invalidating the signature. Authors SHOULD list every contract-bearing field explicitly.
- Including `x-mx-p-` private fields in `contractFields` makes the signature cover values whose meaning is opaque without the private registry. Verifiers without registry access can still verify the signature but cannot interpret what was signed.
- The fingerprint is NOT encrypted. Anyone with the cog can compute the fingerprint and learn the contract scope. Confidentiality of contract values is the responsibility of the carrier transport.

---

## 9. Rationale: attestation, not verification

*This section is informative.*

A signed cog makes one narrow claim, and the narrowness is what makes it tractable. The signature attests provenance and integrity, not truth:

- The signature confirms that the cog's contract surface (the keys named in `contractFields`) was last in this exact state when the named signer applied the signature.
- It does not assert that the values are factually correct, sound, or current. A cog claiming "WWII did not happen" can be perfectly well-signed; the signature only confirms the cog genuinely came from the named signer and has not been altered downstream. Whether the claim is true is a different problem — one no signing format can solve at web scale, and one this note deliberately does not take on.

The narrower claim is intentional. Editorial truth-curation is what made the Open Directory Project (DMOZ) unsustainable: human editors could not keep pace with the web, and the implicit claim that a categorised entry was *appropriate* (worth listing, accurately classified) aged badly. The signing model in this note attests only *this is what the owner published, unaltered*. That smaller claim is what compounds.

Implementations and downstream consumers SHOULD be precise in user-facing language: the system has *attested* a cog, not *verified* it. "Verified" implies a truth claim that no signer in this model makes.

---

## 10. References

### 10.1 Normative references

- [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119), [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) — keywords for use in RFCs to indicate requirement levels
- [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259) — The JSON Data Interchange Format (canonical serialisation in §5)
- [FIPS 180-4](https://csrc.nist.gov/publications/detail/fips/180/4/final) — SHA-256 hash function

### 10.2 Informative references

- [W3C Verifiable Credentials Data Integrity](https://www.w3.org/TR/vc-data-integrity/) — compatible signing model
- [JWS RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) — JSON Web Signature
- [COSE RFC 9052](https://datatracker.ietf.org/doc/html/rfc9052) — CBOR Object Signing and Encryption
- [C2PA Content Credentials](https://c2pa.org/) — provenance signing model
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — conformance level model that inspired the Level 1/2/3 framework in §2.1

---

## 11. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-proposed | 2026-04-26 | Initial draft. Imports the contract-fingerprinting model from earlier reference implementations, drops the `x-mx-` prefix to make the fields first-class, defines the canonical JSON / SHA-256 fingerprint algorithm. |
| 1.0-draft | 2026-04-27 | Renamed from "MX Contract Fingerprinting and Signing Standard" to a "note" to clarify this is a draft by Tom Cranstoun, not a ratified standard. Made the note standalone — removed cross-references to other Gathering drafts and inlined required material. |
| 1.1-draft | 2026-04-27 | §1 abstract gains an explicit statement that signing is optional. New §4.3 documents the mandatory fields when signing (`title`, `validatesAgainst` with resolvable validators, `schema`). New §4.4 documents the default-excluded metadata fields plus the `x-mx-contractFields` / `x-mx-metadataFields` schema overrides. §6 split into signer (§6.1), verifier (§6.2), and recommended pre-signature review pipeline (§6.3). |
| 1.2-draft | 2026-04-27 | Added "What the signature attests" subsection in §1: the signature attests provenance and integrity, not truth. |
| 1.3-draft | 2026-04-27 | Condensed §1 abstract; moved the attestation-vs-verification rationale into a new informative §9 "Rationale: attestation, not verification". Marked §6.3 (pre-signature review pipeline) as informative. Renamed §3.2 from "What this note does not cover" to "Out of scope" (IETF style). References renumbered to §10, change log to §11. |
| 1.4-draft | 2026-04-27 | Reformatted to plain markdown with kramdown-rfc YAML frontmatter (matching the other vocabulary drafts). The old MX-style frontmatter and the runbook are removed; the file is renamed from `draft-contract-fingerprinting.cog.md` to `draft-contract-fingerprinting.md` to reflect that this is a draft note about cog signing, not itself a cog. |
| 1.5-draft | 2026-04-27 | Defers to the MX Field Definition Pattern note (the primary note of the draft set). Field definitions in §4 conform to that pattern; the pattern's authoring rules are not restated here. |
