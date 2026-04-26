---
title: "MX Contract Fingerprinting and Signing Standard"
version: "1.0-proposed"
created: 2026-04-26
modified: 2026-04-26
author: The Gathering
description: "Formal specification of contract fingerprinting and signing for MX cogs. Defines which frontmatter fields are covered by a cog's signature (contractFields) and which are explicitly excluded (metadataFields). Foundation for portable, replay-resistant cog signing."

mx:
  status: proposed
  license: MIT
  partOf: mx-the-gathering
  contentType: specification
  buildsOn: [mxs-01-core-metadata, mxs-02-extensions]
  inherits: mxs-01-core-metadata
  tags: [standard, signing, contract, fingerprint, integrity, specification]
  audience: [humans, machines]
  cacheability: permanent
  runbook: "This standard defines the MX cog signing model. A signed cog declares two arrays at the top level of its frontmatter: contractFields (keys covered by the signature) and metadataFields (keys explicitly excluded from the signature). The fingerprint is computed by canonicalising the projection of the frontmatter to contractFields keys and hashing the result. Read sections 5 (algorithm) and 6 (conformance) before implementing a verifier."
---

# MX Contract Fingerprinting and Signing Standard

**Version:** 1.0-proposed
**Status:** Proposed (draft for Stream submission, awaiting community review)
**Date:** 26 April 2026
**Governing body:** The Gathering
**License:** MIT

---

## 1. Abstract

This document specifies how an MX cog declares the scope of its signature. Cog signing is intended to be portable, transparent, and replay-resistant: a cog moved between systems, registries, or implementations MUST be verifiable without reconstructing the original authoring environment. Two frontmatter fields — `contractFields` and `metadataFields` — make the signature scope explicit.

`contractFields` lists the top-level frontmatter keys whose values are covered by the cog's signature. `metadataFields` lists keys explicitly excluded — values that may change without invalidating the signature (timestamps, tags, registry annotations).

The signing algorithm itself (key types, signature format, transport) is out of scope here; this standard defines only the **fingerprint**: the deterministic byte sequence over which a signature is produced.

---

## 2. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) (RFC 2119 and RFC 8174) when, and only when, they appear in all capitals.

### 2.1 Conformance levels

- **Level 1 (MUST)** — A cog declaring `contractFields` MUST also declare `metadataFields`. The two arrays MUST be disjoint.
- **Level 2 (SHOULD)** — A signed cog SHOULD declare both arrays. Verifiers SHOULD reject a signed cog whose declared scope cannot be reconciled with the signature.
- **Level 3 (MAY)** — Implementations MAY extend the fingerprint algorithm with additional canonicalisation passes provided they are documented in the cog's `validatesAgainst` list (MXS-01 §6.8).

### 2.2 Draft status

This document is a proposed standard awaiting community review. Reference implementations exist in the [mx-upgraded-reginald](https://github.com/Digital-Domain-Technologies-Ltd/mx-upgraded-reginald) submodule under `impl/js/cogs/` and `impl/rust/`.

---

## 3. Scope and relationship to other standards

### 3.1 What this document covers

- The two fields `contractFields` and `metadataFields` (definitions in §4).
- The deterministic fingerprint algorithm (§5).
- Conformance rules and verifier expectations (§6).

### 3.2 What this document does not cover

- Signature algorithms (Ed25519, ECDSA, RSA, post-quantum) — defer to W3C VC Data Integrity, JWS (RFC 7515), or COSE (RFC 9052).
- Key management, distribution, or rotation — defer to organisational PKI.
- Transport encoding (detached vs envelope, PEM vs raw bytes) — defer to the chosen signature standard.
- Multi-signer coordination and revocation — open question (§7).

### 3.3 Relationship to existing standards

| Standard | Relationship |
|----------|--------------|
| [MXS-01 Core Metadata](mxs-01-core-metadata.cog.md) | Inherits `schema` (§6.7) and `validatesAgainst` (§6.8). Both are typically members of `contractFields`. |
| [MXS-02 Extensions](mxs-02-extensions.cog.md) | Workflow contract extensions (§10.4 of MXS-02) are typically members of `contractFields`. |
| [W3C VC Data Integrity](https://www.w3.org/TR/vc-data-integrity/) | Compatible signing model. MX fingerprint may be wrapped in a VC proof. |
| [JWS RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) | Compatible signing model. MX fingerprint may be the JWS payload. |
| [C2PA](https://c2pa.org/) | Compatible model for signing assertions about content provenance. |

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
4. **Serialise.** Emit the canonical object as UTF-8 JSON with no insignificant whitespace.
5. **Hash.** Apply SHA-256 to the canonical byte sequence. The 32-byte digest is the fingerprint.

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

## 6. Verifier conformance

A verifier:

- MUST refuse a signed cog whose `contractFields` and `metadataFields` overlap.
- MUST refuse a signed cog whose declared `contractFields` cannot be reconstructed from the frontmatter (missing keys would change the projection).
- SHOULD warn when frontmatter contains top-level keys not listed in either array.
- SHOULD log the resolved `contractFields` set alongside any verification result for transparency.
- MUST NOT silently substitute a default scope when the cog declares neither array.

---

## 7. Open questions

The following are flagged for community discussion before this draft moves out of `proposed`:

1. **Key management** — should this standard reference a default PKI model, or stay agnostic?
2. **Multi-signer** — when multiple parties sign the same cog, is each signature scoped to the same `contractFields` or may each declare its own scope?
3. **Revocation** — how does a verifier discover that a previously valid signature has been revoked? CRL, OCSP, on-chain registry, MX-Reginald?
4. **Schema-derived contract** — should `contractFields` be derivable from the cog's `schema` (MXS-01 §6.7) automatically, or remain explicit?
5. **Canonical YAML alternative** — section 5 chooses canonical JSON for portability. Should an opt-in canonical-YAML form be standardised for cogs that wish to sign over their authored representation directly?

---

## 8. Reference implementations

- [mx-upgraded-reginald](https://github.com/Digital-Domain-Technologies-Ltd/mx-upgraded-reginald) — JavaScript and Rust implementations of cog-spec v1.0, including contract-fingerprinting logic in `impl/js/cogs/` and `impl/rust/`. Note: cog-spec v1.0 uses kebab-case field names (`x-mx-contract-fields`, `x-mx-metadata-fields`); MX canon and this standard adopt camelCase per [NDR-2026-02-16](../mx-canon/mx-maxine-lives/registers/NDR/2026-02-16-camelcase-naming.cog.md). A future cog-spec revision will align the names.

---

## 9. Security and privacy considerations

- A cog whose `contractFields` does not list every load-bearing field is **silently weakly bound** — values not in the contract may be modified after signing without invalidating the signature. Authors SHOULD list every contract-bearing field explicitly.
- Including `x-mx-p-` private fields in `contractFields` makes the signature cover values whose meaning is opaque without the private registry. Verifiers without registry access can still verify the signature but cannot interpret what was signed.
- The fingerprint is NOT encrypted. Anyone with the cog can compute the fingerprint and learn the contract scope. Confidentiality of contract values is the responsibility of the carrier transport.

---

## 10. References

- [MXS-01 Core Metadata Standard](mxs-01-core-metadata.cog.md) — `schema`, `validatesAgainst`
- [MXS-02 Extensions Standard](mxs-02-extensions.cog.md) — workflow contract extensions
- [BCP 14 / RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) — keywords for use in RFCs
- [W3C Verifiable Credentials Data Integrity](https://www.w3.org/TR/vc-data-integrity/) — compatible signing model
- [JWS RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) — JSON Web Signature
- [C2PA Content Credentials](https://c2pa.org/) — provenance signing model

---

## 11. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-proposed | 2026-04-26 | Initial draft. Imports the contract-fingerprinting model from cog-spec v1.0 (mx-upgraded-reginald), drops the `x-mx-` prefix to make the fields first-class, defines the canonical JSON / SHA-256 fingerprint algorithm. |
