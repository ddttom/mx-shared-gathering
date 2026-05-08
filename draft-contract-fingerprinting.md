---
title: "MX Contract Fingerprinting and Signing note"
docname: draft-cranstoun-mx-contract-fingerprinting
date: 2026-05-08
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
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-contract-fingerprinting.md
---

# MX Contract Fingerprinting and Signing note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 8 May 2026
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

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Reference implementations exist in JavaScript and Rust under the `cog-spec` v1.0 codebase; those implementations carry the kebab-case forms (`x-mx-contract-fields`, `x-mx-metadata-fields`), while the camelCase forms in this note are the canonical names.

---

## 3. Scope

### 3.1 What this note covers

- The two fields `contractFields` and `metadataFields` (definitions in §4).
- The deterministic fingerprint algorithm (§5).
- Conformance rules and verifier expectations (§6).

The fingerprint algorithm in §5 is generic over any structured object. Although §4 specifies it for cog frontmatter, sister notes that need to sign other MX payloads adopt §5 unchanged; §5.4 makes that adoption pattern explicit. Verifiers built against this note can therefore verify any MX signature uniformly without per-payload variants.

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

### 4.4 Computing the contract scope

Implementations resolve the contract scope (which top-level keys are signed) in the following precedence order:

1. **Explicit `contractFields` array on the cog.** When the cog declares `contractFields`, that array is authoritative. The companion `metadataFields` array names the explicit exclusions. The two arrays MUST be disjoint (§4.1, §4.2).

2. **Schema-derived projection.** When the cog does not declare `contractFields` but does declare `schema:`, implementations MUST resolve the schema and compute the contract scope from per-property annotations:

   - A schema property carrying `x-mx-contract: true` is a contract field; its top-level key (or its dotted path for nested keys) is included in the projection.
   - A schema property carrying `x-mx-contract: false` is metadata; the key is excluded.
   - A schema property with no annotation falls through to step 3.

   When schema-derivation is in effect, the cog SHOULD NOT also declare `contractFields` — declaring both is permitted (the explicit array still wins) but the schema annotations are then redundant.

3. **Default classification.** When neither an explicit array nor a schema annotation resolves a key, the following defaults apply:

| Top-level key | Default classification | Rationale |
|---------------|------------------------|-----------|
| `modified` | metadata | Updated whenever the document content changes; not a contract claim. |
| `created` | metadata | Editorial provenance, not contract surface. |
| `originator` (alias `author`) | metadata | Identity field; provenance signature covers authorship separately. |
| `updateInstructions` | metadata | Operational guidance for stewards; not contract surface. |
| `version` | **contract** | Bumping a version is a substantive editorial act. A signature that survives version bumps lets the same signed bytes be republished as v1, v2, v3 without invalidation; that is undesirable. Re-versioning a signed cog re-invalidates the signature; re-signing is required. |
| (anything else) | implementation-defined | Verifiers MAY default unlisted keys to either category but MUST document their choice and apply it consistently. |

**Schema-derivation example.** A schema declared as:

```yaml
$id: ./schemas/invoice-approval.v1.yaml
type: object
properties:
  title:
    type: string
    x-mx-contract: true
  schema:
    type: string
    x-mx-contract: true
  thresholds:
    type: object
    x-mx-contract: true
  modified:
    type: string
    x-mx-contract: false
  tags:
    type: array
    x-mx-contract: false
```

…allows the cog to omit `contractFields`/`metadataFields` entirely. The validator computes the projection from the schema annotations and produces a fingerprint covering `title`, `schema`, `thresholds`. An author writes the schema once; the contract surface is automatically projected from it.

### 4.5 Pre-signature contract declaration (Normative)

An unsigned cog MAY declare `contractFields` and `metadataFields` even though no signature is being applied. The declaration is a **unilateral commitment** — a statement of what the author would sign over if signing were applied — and serves as a useful authorial signal: it tells future readers (human and machine) which keys are intended to be contract surface and which are intended to be metadata.

When an unsigned cog carries the two arrays, verifiers MUST treat the declarations as **informational**:

- A verifier MUST NOT treat the absence of a signature as a conformance failure when `contractFields` is declared on its own. Signing is optional (§1).
- A verifier MAY compute the would-be fingerprint over the declared `contractFields` and surface it as a "pre-signature digest" — a fixed, citeable hash that lets the unsigned cog be referenced by content without depending on later signing activity.
- A verifier SHOULD record that the declarations are unsigned so a downstream consumer cannot mistake them for an attestation.

The §4.1, §4.2, and §4.3 normative requirements (disjoint arrays; mandatory `title` / `validatesAgainst` / `schema`; well-formed entries) apply to the declarations whether or not a signature is present. The §6.1 signer-conformance and §6.2 verifier-conformance rules attach to the signature itself; they do not preclude the declarations from existing on an unsigned cog.

Authors using schema-derivation (§4.4 step 2) need not duplicate the projection as an explicit `contractFields` array on the cog itself; the schema annotations are the declaration. The same informational treatment applies to a verifier that resolves the contract surface from the schema on an unsigned cog.

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

### 5.4 Application to non-cog payloads

The §5.2 canonicalisation algorithm (project, sort, canonicalise values, serialise as canonical JSON, SHA-256) is generic. It applies unchanged to any structured object an MX sister note specifies for signing, not only to cog frontmatter. A note adopting §5 for a non-cog payload SHOULD declare so explicitly so that signers and verifiers know which canonicalisation to apply.

For non-cog payloads, the §5.2 step 1 "project" step is replaced by an inclusion rule defined by the adopting note. Two patterns recur:

- **Whole-payload canonicalisation.** The adopting note specifies that the entire payload object is canonicalised, with a single named field (typically the signature carrier itself) excluded so that the signature is computed over a payload that does not contain its own signature. The MX Compliance Claims note follows this pattern: the entire claim object is canonicalised excluding the `signature` field.
- **Field-list canonicalisation.** The adopting note specifies a fixed list of payload fields to include, analogous to `contractFields` for cogs. This pattern is appropriate where the payload carries fields the adopting note classifies as metadata that should not affect the signature.

Whichever pattern the adopting note chooses, the adopting note MUST be precise about: the inclusion rule, whether unlisted keys are included or excluded, and how the rule interacts with future schema evolution. The §5.2 canonicalisation steps themselves do not vary across adopters; only the projection rule is note-specific.

A sister note that adopts §5 inherits the conformance properties of the algorithm: deterministic, portable, recoverable from the canonical bytes alone. Verifiers that implement §5 once can therefore verify cog signatures and non-cog signatures uniformly, given the per-note projection rule as input.

Current adopters:

- **MX Compliance Claims note** — specifies whole-payload canonicalisation for compliance claim signatures, with the `signature` field excluded.

A sister note proposing a new adoption SHOULD coordinate with this note's authors so that the adoption pattern is recorded in the list above and any incompatibility surfaces early.

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
- MUST resolve the contract scope using the precedence in §4.4 (explicit array → schema-derived projection → defaults). A verifier presented with a schema-derived projection MUST resolve the schema and apply the `x-mx-contract` annotations exactly as the signer would have done.
- SHOULD warn when frontmatter contains top-level keys not listed in either array and not resolvable from the schema.
- SHOULD log the resolved contract set alongside any verification result for transparency, naming the source (explicit array, schema-derived, or default classification).
- MUST NOT silently substitute a fallback scope when the cog declares neither array AND has no `schema:` reference. In that case, verification falls back to the defaults in §4.4 step 3 and the verifier MUST surface that the projection was defaulted.

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

The following remain open for community discussion:

1. **Key management** — should this note reference a default PKI model, or stay agnostic?
2. **Multi-signer** — when multiple parties sign the same cog, is each signature scoped to the same `contractFields` or may each declare its own scope?
3. **Revocation** — how does a verifier discover that a previously valid signature has been revoked? CRL, OCSP, on-chain registry, an MX registry?
4. **Canonical YAML alternative** — §5 chooses canonical JSON for portability. Should an opt-in canonical-YAML form be standardised for cogs that wish to sign over their authored representation directly?

(The earlier open question on schema-derived contract is closed by §4.4: the contract is derivable from a `schema:` reference whose properties carry `x-mx-contract` annotations, and the explicit `contractFields` array is the override.)

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

The narrower claim is intentional. The signing model in this note attests only *this is what the owner published, unaltered*. That smaller claim is what compounds — it is a question a signature can actually answer, every time, without taking on an editorial truth-curation burden that no human review process could keep current at web scale.

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

## 11. Minimum-viable signed cog (Informative)

A complete worked example: schema, cog, projection, fingerprint. An author can copy and adapt this shape for their own contract.

### 11.1 The schema (annotation-driven)

Saved as `./schemas/invoice-approval.v1.yaml`:

```yaml
$id: https://example.org/schemas/invoice-approval.v1.yaml
type: object
properties:
  title:
    type: string
    x-mx-contract: true
  thresholds:
    type: object
    x-mx-contract: true
    properties:
      autoApproveBelow:
        type: number
  approvers:
    type: array
    x-mx-contract: true
  modified:
    type: string
    x-mx-contract: false
  tags:
    type: array
    x-mx-contract: false
```

### 11.2 The cog (no explicit `contractFields` — schema-derived)

Saved as `approve-invoices.cog.md`:

```yaml
---
title: "Approve invoices below 1000"
originator: "Tom Cranstoun"
created: 2026-05-01
modified: 2026-05-07
version: "1.0"
schema: ./schemas/invoice-approval.v1.yaml
validatesAgainst:
  - cog.meta.v1
  - invoice-approval.v1
thresholds:
  autoApproveBelow: 1000
approvers:
  - finance-lead
  - cfo
mx:
  status: published
  contentType: cog
  canonicalUri: https://example.org/cogs/approve-invoices.v1
  summary: "Approve invoices under 1000 automatically; escalate above."
  conformsTo: [https://mx.allabout.network/cog.html]
  trainingDataPolicy: "Permitted with attribution."
---
```

### 11.3 The resolved projection (computed by signer and verifier)

The signer resolves the schema, applies the `x-mx-contract` annotations, and projects the cog's frontmatter to:

```json
{
  "approvers": ["finance-lead", "cfo"],
  "schema": "./schemas/invoice-approval.v1.yaml",
  "thresholds": {"autoApproveBelow": 1000},
  "title": "Approve invoices below 1000",
  "version": "1.0"
}
```

(Keys sorted lexicographically. `version` is included via the §4.4 default — bumping it requires re-signing.)

### 11.4 The fingerprint

`SHA-256` of the canonical JSON serialisation above. Carry this in any signing envelope (JWS, COSE, VC Data Integrity, or a sidecar `*.sig` file).

---

## 12. Verifier checklist (Informative)

The order a verifier SHOULD process a signed cog:

1. **Read the cog and any signature envelope.** If the envelope is detached, locate the corresponding cog file.
2. **Resolve the contract scope** per §4.4 precedence:
    - Explicit `contractFields` on the cog (authoritative when present).
    - Otherwise, resolve `schema:` and apply `x-mx-contract` annotations.
    - Otherwise, apply defaults in §4.4 step 3.
   Surface the resolved scope and its source in any verification log.
3. **Check §4.3 mandatory fields.** `title`, `validatesAgainst`, `schema` MUST all be present and well-formed.
4. **Run every validator named in `validatesAgainst`.** All MUST pass at verification time. A validator that no longer resolves is a verification failure.
5. **Compute the projection.** Build the canonical JSON object containing only the contract-scope keys, sorted lexicographically, with values canonicalised per §5.3.
6. **Hash the projection with SHA-256.** Compare to the fingerprint the signature was applied over.
7. **Verify the signature** using the appropriate envelope's verification algorithm (JWS, COSE, VC, etc.).
8. **Report.** Use the precise word **attested** for a successful verification. Do not say *verified* — that implies a truth claim no signer makes (§9).
9. **Fail closed.** Any verification failure (validator, fingerprint mismatch, signature failure) MUST surface as an explicit failure. Verifiers MUST NOT downgrade a failed verification to a warning.

---

## 13. Common authoring mistakes (Informative)

Five shapes a fresh author often produces, each with the fix.

**Mistake 1: signing without `validatesAgainst`.**

```yaml
# WRONG — §4.3 mandatory: a signed cog MUST declare validatesAgainst
title: "Approve invoices below 1000"
schema: ./schemas/invoice-approval.v1.yaml
contractFields: [title, schema, thresholds]
```

A signer MUST refuse to sign this cog. Add a non-empty `validatesAgainst` array; every entry MUST resolve to a registered validator and MUST pass at sign time.

**Mistake 2: overlapping arrays.**

```yaml
# WRONG — `tags` appears in both
contractFields: [title, schema, tags]
metadataFields: [created, modified, tags]
```

The two arrays MUST be disjoint (§4.1, §4.2). Move `tags` to one or the other and remove it from the other.

**Mistake 3: bumping version expecting the signature to survive.**

```yaml
# Original (signed)
version: "1.0"

# Edited (signature now invalid)
version: "1.1"
```

`version` is in the default contract scope (§4.4). Bumping it changes the projection and invalidates the signature. The intended workflow is: edit content → bump version → re-sign. Never expect a single signature to survive a version bump.

**Mistake 4: schema-derivation with no annotations.**

```yaml
# Schema with no x-mx-contract annotations
$id: ./schemas/foo.v1.yaml
type: object
properties:
  title: { type: string }
  thresholds: { type: object }
```

Schema-derivation (§4.4 step 2) needs explicit `x-mx-contract: true` or `false` annotations on each property the author intends to classify. A schema with no annotations falls through entirely to step 3 (defaults) — the schema is not actually doing any work.

**Mistake 5: claiming the signature *verifies* the values.**

```text
✓ Cog verified — values are correct.
```

The signature attests provenance and integrity, not truth (§9). The cog came from the named signer unaltered; whether `autoApproveBelow: 1000` is the right policy is not something a signature can answer. User-facing language SHOULD say **attested**, not *verified*.

---

*End of MX Contract Fingerprinting and Signing note draft.*
