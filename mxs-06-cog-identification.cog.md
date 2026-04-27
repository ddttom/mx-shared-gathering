---
title: "MX Cog Identification Standard"
version: "1.0-proposed"
created: 2026-04-27
modified: 2026-04-27
author: The Gathering
description: "Frontmatter equivalent of the cog magic-header comment. Defines `cogHeader` (version, spec, runtime, runtimeDoc) so machine consumers can read the cog's spec/runtime conformance claim from queryable YAML rather than parsing a byte-zero HTML comment."

mx:
  status: proposed
  license: MIT
  partOf: mx-the-gathering
  contentType: specification
  buildsOn: [mxs-01-core-metadata]
  inherits: mxs-01-core-metadata
  tags: [standard, cog, identification, magic-header, conformance, specification]
  audience: [humans, machines]
  cacheability: permanent
  runbook: "This standard defines a frontmatter equivalent of the cog magic-header comment. A cog file MAY declare `cogHeader` (with sub-keys version, spec, runtime, runtimeDoc) instead of, or in addition to, the magic-header HTML comment. When both are present they MUST agree. Read sections 4 (field definition) and 5 (equivalence rule) before implementing a parser or validator."
---

# MX Cog Identification Standard

**Version:** 1.0-proposed
**Status:** Proposed (draft for Stream submission, awaiting community review)
**Date:** 27 April 2026
**Governing body:** The Gathering
**License:** MIT

---

## 1. Abstract

A cog file SHOULD identify itself as a cog and point an unfamiliar agent at the specification it claims to follow, the runtime that knows how to consume it, and the runtime's documentation. [Cog Specification v1](https://mx.allabout.network/drafts/cog-spec.v1.md) §2.5 defines a magic-header HTML comment for this purpose:

```html
<!-- cog v1 spec=https://mx.allabout.network/drafts/cog-spec.v1.md runtime=https://mx.allabout.network/drafts/cog-runtime.md -->
```

The comment is byte-zero self-identification — invisible in rendered Markdown, immediately visible to any agent reading the raw text. It is unambiguous for human and agent recognition but invisible to YAML-only consumers (registries, validators, the mx-graph builder, any tool that parses frontmatter and discards comments).

This standard defines a frontmatter equivalent: the `cogHeader` field. A cog MAY declare `cogHeader` instead of, or in addition to, the magic-header comment. Both forms carry the same information; either suffices for cog identification. Implementations SHOULD prefer `cogHeader` for programmatic consumption and the magic-header line for unambiguous file-level recognition.

---

## 2. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) (RFC 2119 and RFC 8174) when, and only when, they appear in all capitals.

### 2.1 Conformance levels

- **Level 1 (MUST when claimed)** — A cog declaring `cogHeader` MUST populate at minimum `version` and `spec`. A cog declaring both `cogHeader` and a magic-header comment MUST keep the two consistent.
- **Level 2 (SHOULD when circulating)** — A cog intended for public registries, mixed audiences, or agent consumption SHOULD include either `cogHeader` or a magic-header comment (preferably both). Cogs operating only within a closed system MAY omit both.
- **Level 3 (MAY)** — `cogHeader` is otherwise optional. Its absence does not invalidate a cog whose structural shape and frontmatter satisfy cog-spec v1.

### 2.2 Draft status

This document is a proposed standard awaiting community review.

---

## 3. Scope and relationship to other standards

### 3.1 What this document covers

- The `cogHeader` frontmatter field and its sub-keys (§4).
- The equivalence rule between `cogHeader` and the magic-header comment (§5).
- Verifier conformance for both forms (§7).

### 3.2 What this document does not cover

- The magic-header comment parser itself — defined in [Cog Specification v1 §2.5](https://mx.allabout.network/drafts/cog-spec.v1.md).
- The cog file format generally — defined in [Cog Specification v1](https://mx.allabout.network/drafts/cog-spec.v1.md).
- Contract fingerprinting and signing — defined in [MXS-05](mxs-05-contract-fingerprinting.cog.md).

### 3.3 Relationship to existing standards

| Standard | Relationship |
|----------|--------------|
| [Cog Specification v1](https://mx.allabout.network/drafts/cog-spec.v1.md) §2.5 | Defines the magic-header comment this standard mirrors. |
| [MXS-01 Core Metadata](mxs-01-core-metadata.cog.md) | `cogHeader` is a Zone 1 (top-level) frontmatter field, sibling to `schema` (§6.7) and `validatesAgainst` (§6.8). |
| [MXS-05 Contract Fingerprinting](mxs-05-contract-fingerprinting.cog.md) | `cogHeader` is typically a member of `metadataFields` (not signed) — its values describe the cog's identity, not its contract. |

---

## 4. Field definition

### 4.1 `cogHeader`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 1 (top-level) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Object carrying the same information as the cog magic-header comment. Sub-keys:

| Sub-key | Type | Required when present | Definition |
|---------|------|-----------------------|-----------|
| `version` | string | yes | Cog spec version the document claims. Format: `v` followed by an integer or a dotted version string (`v1`, `v1.1`, `v2`). Same token as the magic header's version slot. |
| `spec` | string (URL) | yes | URL where the cog specification is published. SHOULD be HTTPS. SHOULD resolve to a stable Markdown document. |
| `runtime` | string (URL) | optional | URL where a runtime implementation can be found, downloaded, or invoked. |
| `runtimeDoc` | string (URL) | optional | URL pointing to runtime documentation explaining how to consume cogs. |

**Example:**

```yaml
cogHeader:
  version: v1
  spec: https://mx.allabout.network/drafts/cog-spec.v1.md
  runtime: https://mx.allabout.network/drafts/cog-runtime.md
  runtimeDoc: https://mx.allabout.network/drafts/cog-runtime.md
```

**Normative notes:**

- A cog declaring `cogHeader` MUST populate `version` and `spec`.
- `runtime` and `runtimeDoc` are optional; consumers MUST NOT treat their absence as a conformance failure.
- All URL-typed sub-keys SHOULD be HTTPS.
- Implementations SHOULD treat `cogHeader` values as `metadataFields` (excluded from the contract fingerprint per [MXS-05](mxs-05-contract-fingerprinting.cog.md)) — these values describe the cog's identity, not its contract.

---

## 5. Equivalence with the magic-header comment

A cog file MAY include any of:

- A magic-header comment only (per cog-spec v1 §2.5).
- A `cogHeader` field only.
- Both forms.
- Neither form (Level 2 SHOULD does not apply within a closed system).

### 5.1 The equivalence rule

When a cog includes both a magic-header comment AND a `cogHeader` field, the two forms MUST agree on every key they both declare:

- The magic header's version token MUST equal `cogHeader.version`.
- The magic header's `spec=` value MUST equal `cogHeader.spec`.
- The magic header's `runtime=` value MUST equal `cogHeader.runtime` when both are present.
- The magic header's `runtime-doc=` value MUST equal `cogHeader.runtimeDoc` when both are present.

A key present in only one form is permitted; a key present in both with mismatched values is a conformance failure.

### 5.2 Implementer guidance

Implementations SHOULD:

- Prefer `cogHeader` for programmatic consumption — it is queryable, robust to comment-stripping parsers, and addressable by registry tooling.
- Prefer the magic-header comment for byte-zero self-identification — it is visible to agents that have not yet parsed YAML.
- Emit both forms when generating cogs intended for circulation, so consumers of either kind can recognise the file without round-trip parsing.

---

## 6. Conformance examples

### 6.1 Magic-header comment only

```html
<!-- cog v1 spec=https://mx.allabout.network/drafts/cog-spec.v1.md runtime=https://mx.allabout.network/drafts/cog-runtime.md -->
---
title: Example cog
description: Demonstrates magic-header-only identification.
---

# Example cog
```

Valid. The cog is identified by the comment; YAML-only consumers must parse the comment to learn the spec/runtime URLs.

### 6.2 `cogHeader` field only

```yaml
---
title: Example cog
description: Demonstrates cogHeader-only identification.
cogHeader:
  version: v1
  spec: https://mx.allabout.network/drafts/cog-spec.v1.md
  runtime: https://mx.allabout.network/drafts/cog-runtime.md
---

# Example cog
```

Valid. The cog is identified by the frontmatter field; agents reading the raw bytes will not recognise the file as a cog until they parse the YAML.

### 6.3 Both forms

```html
<!-- cog v1 spec=https://mx.allabout.network/drafts/cog-spec.v1.md runtime=https://mx.allabout.network/drafts/cog-runtime.md -->
---
title: Example cog
description: Demonstrates both forms in agreement.
cogHeader:
  version: v1
  spec: https://mx.allabout.network/drafts/cog-spec.v1.md
  runtime: https://mx.allabout.network/drafts/cog-runtime.md
---

# Example cog
```

Valid. Both forms agree.

### 6.4 Mismatch (non-conformant)

```html
<!-- cog v1 spec=https://mx.allabout.network/drafts/cog-spec.v1.md -->
---
cogHeader:
  version: v2
  spec: https://example.org/different-spec.md
---
```

Non-conformant. Validators MUST flag the version and spec mismatches.

---

## 7. Verifier conformance

A verifier:

- MUST flag any value disagreement between the magic-header comment and a `cogHeader` field present in the same cog.
- MUST flag a `cogHeader` field that is not an object, or that is missing both `version` and `spec`.
- MUST flag a `cogHeader.version` value that does not match the syntax `^v\d+(\.\d+)*$`.
- SHOULD warn when `cogHeader.spec` is not a syntactically valid HTTPS URL.
- SHOULD warn when neither form is present on a cog declared as a `.cog.md` file (a cog circulating without identification metadata is hard for unfamiliar consumers to handle safely).
- MAY offer auto-fix suggestions when one form is present and the other is absent (generate the missing form from the present one).

---

## 8. Security and privacy considerations

- URLs declared in `cogHeader.runtime` may leak deployment topology. A cog declaring an internal runtime URL is exposing that URL to any reader of the cog. Operators concerned with topology privacy SHOULD either publish a public runtime URL or omit the field.
- A `cogHeader.spec` URL pointing at an attacker-controlled host is a phishing vector for runtimes that auto-fetch the spec. Runtimes SHOULD validate the spec URL against a trusted registry or operator-allowlist before fetching.
- The magic-header comment and `cogHeader` field are NOT authenticated — anyone who can write the cog can lie about which spec it claims to follow. Authentication is handled by the witness/signature model in [MXS-05](mxs-05-contract-fingerprinting.cog.md), not by this standard.

---

## 9. References

- [Cog Specification v1](https://mx.allabout.network/drafts/cog-spec.v1.md) — defines the cog file format and the magic-header comment
- [MXS-01 Core Metadata Standard](mxs-01-core-metadata.cog.md) — `schema`, `validatesAgainst`, the two-zone frontmatter model
- [MXS-05 Contract Fingerprinting Standard](mxs-05-contract-fingerprinting.cog.md) — signing model; `cogHeader` is typically `metadataFields`-scoped
- [Cog Runtime](https://mx.allabout.network/drafts/cog-runtime.md) — runtime companion document
- [BCP 14 / RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) — keywords for use in RFCs

---

## 10. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-proposed | 2026-04-27 | Initial draft. Defines `cogHeader` as the frontmatter equivalent of the cog magic-header comment, including the equivalence rule when both forms appear in the same cog. |
