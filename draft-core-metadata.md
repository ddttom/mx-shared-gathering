---
title: "MX Core Metadata note"
docname: draft-cranstoun-mx-core-metadata
date: 2026-04-27
consensus: false
keyword:
  - mx
  - metadata
  - core
  - conformance
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Core Metadata note

**Version:** 1.3-draft
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note defines the core machine-readable document-metadata vocabulary for the Machine Experience (MX) framework. It specifies the foundational fields that every MX-aware document — whether a markdown file, an HTML page, a YAML sidecar, or any other text-bearing artefact — must, should, or may declare.

The core vocabulary is organised into three groups: Zone 1 identity fields (top-level document identity), Zone 2 operational fields (governance, classification, and distribution metadata under the `mx:` namespace), and pass-through fields (YAML keys whose semantics MX borrows from established external vocabularies — Dublin Core, Schema.org, BCP 47, SPDX — without redefinition).

The cog file format is **not** described here. Cogs are an optional layer on top of MX, covered by the MX Cogs draft note in the same draft set. A document can carry MX metadata without ever being a cog.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

### 2.1 Conformance levels

This note defines three conformance levels.

| Level | Name | Requirement | Description |
|-------|------|-------------|-------------|
| Level 1 | **MX Core** | All MUST fields present and valid | Minimum viable MX metadata. A document at Level 1 is machine-identifiable and attributable. |
| Level 2 | **MX Standard** | All MUST and SHOULD fields present | Recommended for production use. A document at Level 2 is fully classified and lifecycle-managed. |
| Level 3 | **MX Complete** | All MUST, SHOULD, and applicable MAY fields present | Full metadata coverage. A document at Level 3 provides maximum context for agents and tools. |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels. A document claiming "MX Standard" (Level 2) MUST also satisfy all "MX Core" (Level 1) requirements.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions, conformance requirements, and normative rules in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Scope

### 3.1 What this note covers

This note specifies:

- **Zone 1 identity fields** — top-level document identity (title, description, author, dates, version)
- **Zone 2 core operational fields** — classification, governance, and distribution metadata
- **Pass-through fields** — fields where MX provides a YAML key but defers the value semantics to a published external vocabulary
- **The conformance level framework** — Level 1/2/3 definitions

### 3.2 What this note does not cover

- The cog file format (an optional layer over MX). A separate draft note covers cogs.
- Any signing or fingerprint contract a document might choose to carry.
- Carrier-format mappings (how the same field is expressed in HTML meta tags, JSDoc, CSS comments, etc.).

### 3.3 Relationship to existing standards

This note draws on the following published standards. All field names in this note use camelCase, follow spelling-neutral forms (e.g. `license`, not `licence`, when an SPDX-aligned identifier is required), and use ISO 8601 date format (YYYY-MM-DD) for all date fields:

- **ISO 8601** — date and time format
- **SPDX License List** — licence identifiers follow the SPDX standard
- **Schema.org Style Guide** — vocabulary naming conventions (camelCase, lowercase first character)
- **Dublin Core DCMI Namespace** — namespace governance model

---

## 4. Terminology

- **Zone 1** — Top-level YAML frontmatter fields (outside the `mx:` object). Document identity fields.
- **Zone 2** — Fields nested under the `mx:` object in YAML frontmatter. MX-operational fields.
- **Pass-through field** — A field name MX provides as a YAML key whose value semantics are owned by an established external vocabulary (Dublin Core, Schema.org, BCP 47, SPDX, ...). MX does not redefine; it simply carries the key.
- **Profile** — A named set of fields applicable to a specific document type (e.g., `core`, `folder`, `book`).
- **SSOT** — Single Source of Truth. The authoritative source for a given piece of information.

---

## 5. The two-zone frontmatter model

MX metadata in YAML frontmatter is organised into two zones:

**Zone 1 (top-level):** Document identity fields. These MUST appear at the top level of the YAML frontmatter, never inside the `mx:` object.

```yaml
---
title: "Document Title"
description: "Brief summary for search and AI agents"
author: "Author Name"
created: 2026-04-02
modified: 2026-04-16
version: "1.0"
---
```

**Zone 2 (mx: object):** All other MX-operational fields. These MUST appear inside the `mx:` object.

```yaml
---
title: "Document Title"
description: "Brief summary"
author: "Author Name"
created: 2026-04-02
modified: 2026-04-16
version: "1.0"

mx:
  status: active
  contentType: guide
  tags: [metadata, example]
  audience: [humans]
---
```

Implementations MUST NOT place Zone 1 fields inside the `mx:` object. Implementations MUST NOT place Zone 2 fields at the top level.

---

## 6. Field definitions — Zone 1 identity fields

### 6.1 `title`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Human-readable document title. The canonical identity field for all documents including cog files.

**Example:**

```yaml
title: "MX Core Metadata note"
```

**Normative notes:**

- Every MX-aware document MUST declare a `title`.
- If both `title` in frontmatter and an H1 heading exist in the document body, authors SHOULD avoid duplication by omitting `title` from frontmatter and relying on the H1 heading, or vice versa.

---

### 6.2 `description`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none — explicit declaration required)* |

**Definition:** One-line summary. Used by search engines, AI agents, and registry listings.

**Example:**

```yaml
description: "Single source of truth for every YAML frontmatter field."
```

**Normative notes:**

- The value SHOULD NOT exceed 160 characters.
- The description MUST be a single sentence or phrase summarising the document's purpose.

---

### 6.3 `author`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Creator of the document. Person name or collaborative attribution.

**Example:**

```yaml
author: "Tom Cranstoun"
```

**Normative notes:**

- The `author` field is immutable after creation. Implementations MUST NOT change this value after the document is first committed.
- For collaborative work, all contributors SHOULD be listed.
- `author` is distinct from `maintainer` (see Section 7.5). The author is the original creator; the maintainer handles ongoing updates.

---

### 6.4 `created`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Creation date in ISO 8601 format (YYYY-MM-DD).

**Example:**

```yaml
created: 2026-04-02
```

**Normative notes:**

- The value MUST use ISO 8601 date format (YYYY-MM-DD).
- This field is immutable — once set, it MUST NOT be changed.

---

### 6.5 `modified`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Last modification date in ISO 8601 format (YYYY-MM-DD).

**Example:**

```yaml
modified: 2026-04-16
```

**Normative notes:**

- The value MUST use ISO 8601 date format (YYYY-MM-DD).
- Implementations MUST update this field every time the document's content changes.

---

### 6.6 `version`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Semantic version string.

**Example:**

```yaml
version: "2.0"
```

**Normative notes:**

- The value MUST be quoted in YAML to prevent numeric coercion (e.g., `"1.0"` not `1.0`).
- Version numbers live in frontmatter, never in filenames.

---

### 6.7 `schema`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Schema reference identifier — pointer to the schema document that defines and validates this document's contract. May be a Schema.org type URL, a JSON Schema `$id`, a relative path to a YAML/JSON schema file, or a database schema name. Context-dependent.

**Example:**

```yaml
schema: ./schemas/invoice-approval.v1.yaml
```

**Normative notes:**

- When present, validators SHOULD attempt to resolve the reference and apply the referenced schema before checking field-level conformance.
- The reference MAY be a relative path, an absolute path, a URL, or a registry-resolvable name.
- Implementations SHOULD treat unresolved references as a warning, not a failure, since the schema may live in an external system.

---

### 6.8 `validatesAgainst`

| Property | Value |
|----------|-------|
| **Type** | array of string |
| **Zone** | 1 (top-level) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Named validators (dotted notation) this document claims conformance to. A document MAY declare conformance to multiple validators (typically one meta-validator plus one or more domain validators).

**Example:**

```yaml
validatesAgainst:
  - cog.meta.v1
  - invoice-approval.v1
```

**Normative notes:**

- Each entry SHOULD be a dotted name resolvable via the cog registry or via a `schema` pointer.
- Validators MUST treat the array as additive — a document conforming to `cog.meta.v1` AND `invoice-approval.v1` must satisfy both.
- Implementations SHOULD warn when a referenced validator name cannot be resolved; they MUST NOT silently skip validation.

---

## 7. Field definitions — Zone 2 core operational fields

### 7.1 `status`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |
| **Valid values** | draft, active, published, deprecated, archived, unknown, proposed, accepted, rejected, superseded, pending, review, approved, planning, open, closed, sent, canonical |
| **Default** | *(none — explicit declaration recommended)* |

**Definition:** Lifecycle state. Different document contexts use different subsets of the valid values.

**Example:**

```yaml
mx:
  status: active
```

**Normative notes:**

- Standard lifecycle values: `draft`, `active`, `published`, `deprecated`, `archived`, `unknown`.
- Decision record values: `proposed`, `accepted`, `rejected`, `superseded`.
- Workflow values: `pending`, `review`, `approved`, `planning`, `open`, `closed`, `sent`.
- Special value: `canonical` (indicates an authoritative reference document).
- Implementations SHOULD always declare `status` to communicate the document's lifecycle stage.

---

### 7.2 `tags`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(empty array)* |

**Definition:** Discovery keywords. Array of strings for search, filtering, and agent matching.

**Example:**

```yaml
mx:
  tags: [metadata, yaml, frontmatter, reference]
```

**Normative notes:**

- Values SHOULD be lowercase strings.
- Tags enable discovery; they are distinct from `words` (spell-checker dictionary) and `keyFields` (agent attention routing).

---

### 7.3 `audience`

| Property | Value |
|----------|-------|
| **Type** | string-or-array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | tech, business, humans, machines, agents, both |
| **Default** | *(none)* |

**Definition:** Intended readership.

**Example:**

```yaml
mx:
  audience: [humans, machines]
```

**Normative notes:**

- The value MAY be a single string or an array of strings.

---

### 7.4 `purpose`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | specification, reference, guide, operational manual, dispatcher, configuration |
| **Default** | *(none)* |

**Definition:** Why this document exists. Distinct from `contentType` (which classifies what a document is, not why it exists).

**Example:**

```yaml
mx:
  purpose: reference
```

---

### 7.5 `license`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** SPDX licence identifier.

**Example:**

```yaml
mx:
  license: MIT
```

**Normative notes:**

- Values MUST use SPDX licence identifiers as defined in the [SPDX Licence List](https://spdx.org/licenses/).
- Common values: `proprietary`, `MIT`, `Apache-2.0`, `CC-BY-4.0`.
- The field name uses the spelling-neutral form `license` (SPDX standard spelling) so that the YAML key matches the SPDX identifier exactly across regional spellings.

---

### 7.6 `maintainer`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Person or team responsible for maintaining this document. Distinct from `author` — the maintainer handles ongoing updates while the author is the immutable creator.

**Example:**

```yaml
mx:
  maintainer: "Maxine"
```

**Normative notes:**

- This field is mutable and MAY change as stewardship transfers.
- When omitted, the `author` is assumed to also be the maintainer.

---

### 7.7 `ownership`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Ownership details. Can be a string (owner name) or an object with `owner`, `delegate`, and `contact` sub-fields.

**Example:**

```yaml
mx:
  ownership: "Tom Cranstoun"
```

```yaml
mx:
  ownership:
    owner: "Tom Cranstoun"
    delegate: "Maxine"
    contact: "tom@example.com"
```

---

### 7.8 `domain`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Business domain or subject area.

**Example:**

```yaml
mx:
  domain: "machine-experience"
```

**Normative notes:**

- Values are context-specific and not constrained to an enumeration.
- In folder metadata, `domain` is an identity field and MUST NOT be inherited by child folders.

---

### 7.9 `contentType`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Machine-readable content type classification.

**Example:**

```yaml
mx:
  contentType: field-dictionary
```

**Normative notes:**

- Common values include: `specification`, `guide`, `reference`, `info-doc`, `identity`, `report`, `field-dictionary`, `standards-alignment`.
- This is a free-form classification, not constrained to an enumeration, to allow domain-specific content types.

---

### 7.10 `segment`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | developer, author, agent, business, morning, afternoon, evening |
| **Default** | *(none)* |

**Definition:** Segment classifier. Used in two contexts: audience segment (developer, author, agent, business) and time segment for session reports (morning, afternoon, evening).

**Example:**

```yaml
mx:
  segment: developer
```

---

### 7.11 `cacheability`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | ephemeral, short-lived, medium, long-lived, permanent |
| **Default** | medium |

**Definition:** How long an agent or resolver may cache this document's content before re-fetching.

**Example:**

```yaml
mx:
  cacheability: long-lived
```

**Normative notes:**

- Named tiers map to durations:
  - `ephemeral` — MUST NOT be cached; content changes on every request
  - `short-lived` — cache for up to 1 hour
  - `medium` — cache for up to 1 day
  - `long-lived` — cache for up to 1 week
  - `permanent` — cache indefinitely until the document changes
- A custom duration string (e.g., `4h`, `30d`) is also accepted. Format: number + unit (`s`, `m`, `h`, `d`, `w`).
- When omitted, implementations SHOULD treat the value as `medium`.
- Distinct from `stability` (content reliability) and `lifecycle` (development phase).

---

### 7.12 `readingLevel`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | beginner, intermediate, advanced, expert |
| **Default** | *(none)* |

**Definition:** Content reading level. Helps agents recommend content based on user expertise.

**Example:**

```yaml
mx:
  readingLevel: intermediate
```

---

### 7.13 `runbook`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Operational instructions for agents. Describes how to interpret and act on this document.

**Example:**

```yaml
mx:
  runbook: "Parse the fields array. Each entry has name, type, definition, status, and profile."
```

**Normative notes:**

- The runbook SHOULD be written in imperative voice.
- The runbook SHOULD be specific enough that an agent can act on it without reading the full document body.
- This field is particularly valuable for machine-facing documents where the document structure is not self-evident.

---

### 7.14 `confidential`

| Property | Value |
|----------|-------|
| **Type** | boolean |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | false |

**Definition:** Whether this record must be excluded from public outputs.

**Example:**

```yaml
mx:
  confidential: true
```

**Normative notes:**

- When omitted, the default is `false` (document is public).
- Authors SHOULD only declare this field when marking content as confidential.

---

### 7.15 `inherits`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Path to the file this document extends. The inheriting file adds MX metadata on top of the target's content. The target can be any file type.

**Example:**

```yaml
mx:
  inherits: "README.md"
```

**Normative notes:**

- File-type agnostic — the target can be `.md`, `.cog.md`, `.html`, `.json`, `.yaml`, or any other file type.
- The inheriting file extends the target; it does not replace it.
- The path MAY be relative or absolute.
- For folder-level field inheritance, an `inheritable` field on the parent folder's metadata (out of scope for this note) is the preferred mechanism.

---

### 7.16 `ld`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Inline JSON-LD (Schema.org) expressed in YAML frontmatter. Enables structured data without a separate `<script>` block.

**Example:**

```yaml
mx:
  ld:
    "@context": "https://schema.org"
    "@type": "TechArticle"
    name: "MX Core Metadata note"
```

**Normative notes:**

- The `ld` field provides a YAML representation of JSON-LD structured data.
- Implementations rendering HTML from this metadata SHOULD convert the `ld` object to a `<script type="application/ld+json">` block.

---

## 8. Pass-through fields

A **pass-through field** is a YAML key MX provides whose value semantics belong to an established external vocabulary. MX does not redefine the value's meaning. The key exists so authors don't have to switch syntax mid-frontmatter to use the external vocabulary; tooling treats the field as the aligned external field.

The pattern is:

1. The external vocabulary owns the value semantics.
2. MX names a YAML key (camelCase, in keeping with the rest of the vocabulary).
3. Canon entries declare the alignment via `alignsWith:` so that converters and validators can route the value to the external standard's schema.

### 8.1 Inventory

The current pass-through inventory (matching the `alignsWith:` declarations in `fields-data.yaml`):

| Field | Aligns with | Value | Conformance |
|-------|-------------|-------|:-----------:|
| `date` | Dublin Core `dc:date`, Schema.org `Date` | ISO 8601 date for a generic reference event distinct from `created` / `modified`. | MAY |
| `duration` | Schema.org `duration` | ISO 8601 duration (e.g. `PT1H30M`). For time-based content. | MAY |
| `format` | Dublin Core `dc:format`, Schema.org `encodingFormat` | Media type or file format string. | MAY |
| `rights` | Dublin Core `dc:rights`, Schema.org `license` | Free-form rights statement when an SPDX identifier does not fit. | MAY |
| `displayName` | FOAF `displayName`, Schema.org `alternateName` | Citation form when it differs from `title`. | MAY |
| `usage` | Schema.org `usageInfo` | Guidance on how the document is meant to be consumed. | MAY |
| `url` | Schema.org `url`, Dublin Core `identifier` | Canonical URL where the document lives or is published. | MAY |

### 8.2 Externally-aligned fields (genuineness family)

The following MX-defined fields carry semantics MX itself names but model after established standards. They are not strict pass-through (the field name is MX's), but they are alignment-anchored and tooling SHOULD interoperate with the cited standards:

| Field | Aligns with | Purpose |
|-------|-------------|---------|
| `proofOfAuthorship` | W3C Verifiable Credentials, Signed Exchanges | Cryptographic claim that the named author wrote this. |
| `integritySignature` | RFC 9421 HTTP Message Signatures, Subresource Integrity | Hash or signature over the artefact's content. |
| `provenancePedigree` | W3C PROV-O, C2PA Content Credentials | Derivation chain back to the source. |

### 8.3 Adding a new pass-through field

A new pass-through field SHOULD be proposed only when an established external vocabulary cleanly covers the value semantics and there is value in surfacing the key in MX frontmatter. The proposal MUST:

- Cite the external standard precisely (URL plus the specific field/property name).
- Declare the alignment in canon via `alignsWith:`.
- NOT redefine the value's meaning. MX's role is to carry the key, not to govern its semantics.

If the external standard does not cleanly cover the value, the field is not a pass-through and belongs in the proper Zone 1 / Zone 2 section instead, with full MX semantics.

---

---

## 9. Conformance levels summary

### 9.1 Zone 1 identity fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `title` | MUST | — | — |
| `description` | MUST | — | — |
| `author` | MUST | — | — |
| `created` | MUST | — | — |
| `modified` | MUST | — | — |
| `version` | — | SHOULD | — |

### 9.2 Zone 2 core operational fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `status` | — | SHOULD | — |
| `contentType` | — | SHOULD | — |
| `runbook` | — | SHOULD | — |
| `tags` | — | — | MAY |
| `audience` | — | — | MAY |
| `purpose` | — | — | MAY |
| `license` | — | — | MAY |
| `maintainer` | — | — | MAY |
| `ownership` | — | — | MAY |
| `domain` | — | — | MAY |
| `segment` | — | — | MAY |
| `cacheability` | — | — | MAY |
| `readingLevel` | — | — | MAY |
| `confidential` | — | — | MAY |
| `inherits` | — | — | MAY |
| `ld` | — | — | MAY |

### 9.3 Pass-through fields

All pass-through fields (§8) are MAY at Level 3. They appear in MX frontmatter at the author's discretion when an external vocabulary applies.

---

## 10. Security and privacy considerations

- The `confidential` field controls whether a document is excluded from public outputs. Implementations that generate public artefacts MUST respect this field.
- The `author` field is immutable and establishes provenance. Implementations MUST NOT allow modification of this field after initial creation.
- The `ownership` field MAY contain contact information. Implementations that publish metadata publicly SHOULD consider whether `ownership.contact` values are appropriate for public exposure.

---

## 11. References

### 11.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) — Date and time format
- [SPDX Licence List](https://spdx.org/licenses/) — Software Package Data Exchange licence identifiers

### 11.2 Informative references

- [Schema.org Style Guide](https://schema.org/docs/styleguide.html) — Vocabulary naming conventions
- [Dublin Core DCMI Namespace](https://www.dublincore.org/specifications/dublin-core/dcmi-namespace/) — Namespace governance model
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (conformance level model that inspired the Level 1/2/3 framework in §2.1)

---

## 12. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-draft | 2026-04-02 | Initial draft. |
| 1.2-proposed | 2026-04-26 | Added Zone 1 fields `schema` (§6.7) and `validatesAgainst` (§6.8) — schema-reference and conformance-claim fields. Both at conformance level MAY. |
| 1.3-draft | 2026-04-27 | Removed all cog content (the 12 cog structural fields and the cog terminology entry); cogs are an optional layer covered by the MX Cogs draft note. Added §8 "Pass-through fields" naming the pattern, listing the canonical inventory (`date`, `duration`, `format`, `rights`, `displayName`, `usage`, `url`), and documenting the rule for adding new pass-through fields. |
