---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Core Metadata note"
docname: draft-cranstoun-mx-core-metadata
date: 2026-05-23
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
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-core-metadata.md
---

# MX Core Metadata note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note defines the core machine-readable document-metadata vocabulary for the Machine Experience (MX) framework. It specifies the foundational fields that every MX-aware document — whether a markdown file, an HTML page, a YAML sidecar, or any other text-bearing artefact — must, should, or may declare.

The core vocabulary is organised into three groups: **Zone 1 identity fields** (top-level document identity), **Zone 2 operational fields** (governance, classification, and distribution metadata under the `mx:` namespace), and **externally-aligned fields** (YAML keys whose semantics are owned by, or modelled after, established external vocabularies — Dublin Core, Schema.org, BCP 47, SPDX, RFC 3986). The relationship between MX and each external standard is recorded explicitly per field; see §7.

The cog file format is **not** described here. Cogs are an optional layer on top of MX, covered by the MX Cogs note in the same draft set. A document can carry MX metadata without ever being a cog.

**Accessibility (Normative).** An MX-compliant document MUST itself be accessible. For HTML, EPUB, and other web-style carriers, the artefact MUST conform to [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level AA. For PDF, the artefact MUST be tagged for accessibility with at least the conformance level cited in the MX Document Accessibility note. For markdown and other plain-text carriers, the structural conventions covered by the MX Document Accessibility note (heading hierarchy, alt text on images, descriptive link text, plain-language summaries) apply. MX adds machine-readable governance on top of standards-conformant content; it does not replace or relax the underlying accessibility requirement.

**Provenance.** Authorship, stewardship, derivation, and review history are governed by the **MX Provenance note**. That note extends this floor with the attribution and trust fields that make a document's origin verifiable. It is part of the recommended reading order for any author writing for an MX-aware audience.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field — heading, property table, definition prose, example — and the inventory-table form used for the externally-aligned fields in §7. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

| Level | Name | Requirement | Description |
|-------|------|-------------|-------------|
| Level 1 | **MX Core** | All MUST fields present and valid | Minimum viable MX metadata. A document at Level 1 is machine-identifiable and attributable. |
| Level 2 | **MX Standard** | All MUST and SHOULD fields present | Recommended for production use. A document at Level 2 is fully classified and lifecycle-managed. |
| Level 3 | **MX Complete** | All MUST, SHOULD, and applicable MAY fields present | Full metadata coverage. |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions, conformance requirements, and normative rules in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Scope

### 3.1 In scope

- **Zone 1 identity fields** — top-level document identity (title, description, author, dates, version)
- **Zone 2 core operational fields** — classification, governance, and distribution metadata
- **Externally-aligned fields** — fields where MX provides a YAML key whose value semantics are owned by, or modelled after, an established external vocabulary (§7)
- **The conformance level framework** — Level 1/2/3 definitions

### 3.2 Out of scope

- The cog file format (an optional layer over MX). A separate note covers cogs.
- Any signing or fingerprint contract a document might choose to carry.
- Carrier-format mappings (how the same field is expressed in HTML meta tags, JSDoc, CSS comments, etc.).

### 3.3 Relationship to existing standards

All field names in this note use camelCase, follow spelling-neutral forms (e.g. `license`, not `licence`, when an SPDX-aligned identifier is required), and use ISO 8601 date format (YYYY-MM-DD) for all date fields:

- **ISO 8601** — date and time format
- **SPDX License List** — licence identifiers follow the SPDX standard
- **Schema.org Style Guide** — vocabulary naming conventions (camelCase, lowercase first character)
- **Dublin Core DCMI Namespace** — namespace governance model

---

## 4. The two-zone frontmatter model

MX metadata in YAML frontmatter is organised into two zones. **Zone 1** (top-level) carries document identity fields. **Zone 2** (under the `mx:` object) carries MX-operational fields. **Externally-aligned fields** are YAML keys whose value semantics are owned by, or modelled after, an established external vocabulary; the inventory and the alignment relationships are in §7, and the fields otherwise behave as Zone 1 or Zone 2 depending on which zone the field declaration places them in.

**Zone 1 example:**

```yaml
---
title: "Document Title"
description: "Brief summary for search engines and machines"
author: "Author Name"
created: 2026-04-02
modified: 2026-04-16
version: "1.0"
---
```

**Zone 2 example:**

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

## 5. Zone 1 identity fields

### 5.1 `title`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |

Human-readable document title. The canonical identity field for all documents.

```yaml
title: "MX Core Metadata note"
```

If both `title` in frontmatter and an H1 heading exist in the document body, authors SHOULD avoid duplication by omitting `title` from frontmatter and relying on the H1 heading, or vice versa.

---

### 5.2 `description`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |

One-line summary. Used by search engines, machines of every class, and registry listings. The value SHOULD NOT exceed 160 characters and MUST be a single sentence or phrase summarising the document's purpose.

```yaml
description: "Single source of truth for every YAML frontmatter field."
```

---

### 5.3 `originator`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |

Creator of the document. Person name or collaborative attribution. Identity, not stewardship — names who first wrote the document, not who currently looks after it.

```yaml
originator: "Tom Cranstoun"
```

- The `originator` field is immutable after creation. Implementations MUST NOT change this value after the document is first committed.
- For collaborative work, all contributors SHOULD be listed.
- `originator` is distinct from the `stewardship` object (§6.6). The originator is the original creator (immutable, Zone 1); stewardship covers ongoing maintenance, contact, and ownership (mutable, Zone 2).

**Alias.** Documents written before this rename used the field name `author`. Implementations MUST treat a top-level `author` field as an alias of `originator` and SHOULD copy the value across when migrating. The `author` alias is supported for one major version after this note ratifies, then retires per the field-deprecation lifecycle in the MX Extensions note §10.4.

---

### 5.4 `created`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |

Creation date in ISO 8601 format (YYYY-MM-DD). This field is immutable — once set, it MUST NOT be changed.

```yaml
created: 2026-04-02
```

---

### 5.5 `modified`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MUST (Level 1) |

Last modification date in ISO 8601 format (YYYY-MM-DD). Implementations MUST update this field every time the document's content changes.

```yaml
modified: 2026-04-16
```

---

### 5.6 `version`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | SHOULD (Level 2) |

Semantic version string. The value MUST be quoted in YAML to prevent numeric coercion (e.g., `"1.0"` not `1.0`). Version numbers live in frontmatter, never in filenames.

```yaml
version: "2.0"
```

---

### 5.7 `schema`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Conformance** | MAY (Level 3) |

Schema reference identifier — pointer to the schema document that defines and validates this document's contract. May be a Schema.org type URL, a JSON Schema `$id`, a relative path to a YAML/JSON schema file, or a database schema name.

```yaml
schema: ./schemas/invoice-approval.v1.yaml
```

- When present, validators SHOULD attempt to resolve the reference and apply the referenced schema before checking field-level conformance.
- The reference MAY be a relative path, an absolute path, a URL, or a registry-resolvable name.
- Implementations SHOULD treat unresolved references as a warning, not a failure, since the schema may live in an external system.

---

### 5.8 `validatesAgainst`

| Property | Value |
|----------|-------|
| **Type** | array of string |
| **Zone** | 1 (top-level) |
| **Conformance** | MAY (Level 3) |

Named validators (dotted notation) this document claims conformance to. A document MAY declare conformance to multiple validators (typically one meta-validator plus one or more domain validators).

```yaml
validatesAgainst:
  - cog.meta.v1
  - invoice-approval.v1
```

- Each entry SHOULD be a dotted name resolvable via the cog registry or via a `schema` pointer.
- Validators MUST treat the array as additive — a document conforming to `cog.meta.v1` AND `invoice-approval.v1` must satisfy both.
- Implementations SHOULD warn when a referenced validator name cannot be resolved; they MUST NOT silently skip validation.

---

## 6. Zone 2 core operational fields

### 6.1 `status`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |
| **Valid values** | draft, active, published, deprecated, archived, unknown, proposed, accepted, rejected, superseded, pending, review, approved, planning, open, closed, sent, canonical |

Lifecycle state. The valid values span three lifecycles — document, decision-record, and workflow — and the value's meaning depends on the document's `contentType` (§6.8). The matrix in §6.1.1 governs which values are valid for which content types; an out-of-matrix value is a conformance failure.

```yaml
mx:
  status: active
```

#### 6.1.1 Valid `status` by `contentType` (Normative)

A document MUST declare a `status` value drawn from the row matching its `contentType`. The matrix below is normative; a `status` value that does not appear in the row for the document's declared `contentType` is a conformance failure.

| `contentType`            | Valid `status` values                                                |
|--------------------------|----------------------------------------------------------------------|
| `info-doc`, `manuscript`, `guide`, `reference`, `report`, `field-dictionary`, `standards-alignment` | draft, active, published, archived, canonical, unknown |
| `specification`, `cog`, `cogs`              | draft, active, canonical, deprecated, archived, unknown              |
| `decision-record`, `adr`                    | proposed, accepted, rejected, superseded                             |
| `workflow`, `task`, `ticket`                | pending, review, approved, planning, open, closed, sent              |
| `identity`                                  | active, archived, unknown                                            |

Notes on the matrix:

- A document whose `contentType` does not appear in the matrix MAY use any value in the document lifecycle row (`draft`, `active`, `published`, `archived`, `canonical`, `unknown`) as a default; sister notes adding new content types SHOULD extend the matrix in lockstep.
- `unknown` is a conformance-safe placeholder for documents whose lifecycle is genuinely undetermined; tools SHOULD warn when it persists past initial authorship.
- `canonical` denotes an authoritative reference document — the present-tense source of truth for its subject. Distinct from `published` (a release point) and `active` (currently maintained).
- A document that legitimately spans lifecycles (rare — for example, a decision-record blog post) MUST pick a single `contentType` and draw its `status` from that lifecycle's row.

---

### 6.2 `tags`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Discovery keywords. Array of strings for search, filtering, and agent matching. Values SHOULD be lowercase strings.

```yaml
mx:
  tags: [metadata, yaml, frontmatter, reference]
```

---

### 6.3 `audience`

| Property | Value |
|----------|-------|
| **Type** | string-or-array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | tech, business, humans, machines, agents, both |

Intended readership.

```yaml
mx:
  audience: [humans, machines]
```

---

### 6.4 `purpose`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Why this document exists. Distinct from `contentType` (§6.8), which classifies what a document is, not why it exists. The field accepts two shapes:

- **String form** — a single value drawn from the controlled high-level vocabulary below. Sufficient for most documents.
- **Object form** — `{ kind: <controlled value>, subPurpose: "<free-form>" }` — for documents whose purpose has a recognisable major axis but a specific genre that the controlled vocabulary does not name (a position paper, a press release, a regulatory filing, a contract, a case study).

The controlled high-level vocabulary (used as the bare string value or as the `kind` sub-key in the object form):

| Value | Definition |
|-------|-----------|
| `specification` | Defines a contract, schema, or normative rule that other artefacts implement. |
| `reference` | A lookup resource: catalogue, dictionary, registry, atlas. Read by index, not cover-to-cover. |
| `guide` | Walks a reader through an unfamiliar process or domain. Pedagogical. |
| `operational` | Operational instructions for an actor (human or machine) to execute. Includes runbooks, SOPs, manuals. |
| `narrative` | Argument-bearing prose. Includes position papers, blog posts, essays, manifestos, press releases, opinion pieces. |
| `record` | Captures a state, event, or decision for the historical record. Includes minutes, decision records, audit logs, reports. |

A document declaring `purpose` SHOULD pick the value whose definition fits best; when the genre is recognisable but specific, the object form names it via `subPurpose`.

**Examples:**

```yaml
# String form — most documents
mx:
  purpose: reference
```

```yaml
# Object form — narrative, named genre
mx:
  purpose:
    kind: narrative
    subPurpose: "position paper"
```

```yaml
# Object form — record, named genre
mx:
  purpose:
    kind: record
    subPurpose: "regulatory filing"
```

- The legacy values `operational manual`, `dispatcher`, and `configuration` (from earlier drafts of this note) are no longer in the controlled vocabulary. Implementations MUST treat `operational manual` as an alias of `operational`; `dispatcher` and `configuration` SHOULD be migrated to the closest fit (typically `operational` with a `subPurpose`) and the alias supported for one major version.
- `subPurpose` is free-form and lower-case-with-spaces. Tools SHOULD index `subPurpose` values for discoverability but MUST NOT treat them as a controlled vocabulary.

---

### 6.5 `license`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

SPDX licence identifier. Values MUST use SPDX licence identifiers as defined in the [SPDX Licence List](https://spdx.org/licenses/). Common values: `proprietary`, `MIT`, `Apache-2.0`, `CC-BY-4.0`. The field name uses the spelling-neutral form `license` (SPDX standard spelling) so that the YAML key matches the SPDX identifier exactly across regional spellings.

```yaml
mx:
  license: MIT
```

---

### 6.6 `stewardship`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Mutable stewardship details. A single object that captures ongoing responsibility for the document — who maintains it, who answers questions about it, which legal entity owns it, what brand it is published under. Distinct from `originator` (§5.3), which is the immutable original creator and lives at the top level.

**Sub-keys** (all optional; declare the ones the document needs):

| Sub-key | Type | Definition |
|---------|------|-----------|
| `steward` | string | Person or team currently responsible for ongoing updates and corrections. May change as stewardship transfers. |
| `accountableContact` | string | Mailto, URL, or organisation reference that an outside reader uses when something is wrong with the document. The single point of contact for follow-up. |
| `legalEntity` | string | The legal owner of the document and its underlying rights. Typically a registered company or trading entity. |
| `brand` | string | Public-facing trading name or imprint under which the document is published. May differ from `legalEntity`. |

```yaml
mx:
  stewardship:
    steward: "Maxine + Tom"
    accountableContact: "info@cognovamx.com"
    legalEntity: "Digital Domain Technologies Ltd"
    brand: "CogNovaMX"
```

- A document SHOULD declare at least one sub-key when `stewardship` is present; an empty object is permitted but unusual.
- When `steward` is omitted, the `originator` is assumed to also be the steward.
- Implementations that publish metadata publicly SHOULD consider whether `accountableContact` values are appropriate for public exposure.

**Aliases.** Documents written before this consolidation used `maintainer` (string) and `ownership` (string-or-object). Implementations MUST treat them as aliases:

| Legacy field | Maps to |
|--------------|---------|
| `mx.maintainer: "<name>"` | `mx.stewardship.steward: "<name>"` |
| `mx.ownership: "<name>"` (string form) | `mx.stewardship.legalEntity: "<name>"` |
| `mx.ownership.owner` | `mx.stewardship.legalEntity` |
| `mx.ownership.delegate` | `mx.stewardship.steward` |
| `mx.ownership.contact` | `mx.stewardship.accountableContact` |

The legacy fields are supported for one major version after this note ratifies, then retire per the field-deprecation lifecycle in the MX Extensions note §10.4. The polymorphic string-or-object shape of `ownership` is retired entirely; the explicit object on `stewardship` replaces it.

---

### 6.7 `domain`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Business domain or subject area. Values are context-specific and not constrained to an enumeration. In folder metadata, `domain` is an identity field and MUST NOT be inherited by child folders.

```yaml
mx:
  domain: "machine-experience"
```

---

### 6.8 `contentType`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |

Machine-readable content type classification. Free-form (not constrained to an enumeration) to allow domain-specific content types. Common values: `specification`, `guide`, `reference`, `info-doc`, `identity`, `report`, `field-dictionary`, `standards-alignment`.

```yaml
mx:
  contentType: field-dictionary
```

---

### 6.9 `segment`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | developer, author, agent, business, morning, afternoon, evening |

Segment classifier. Used in two contexts: audience segment (developer, author, agent, business) and time segment for session reports (morning, afternoon, evening).

```yaml
mx:
  segment: developer
```

---

### 6.10 `cacheability`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | ephemeral, short-lived, medium, long-lived, permanent |
| **Default** | medium |

How long an agent or resolver may cache this document's content before re-fetching.

- `ephemeral` — MUST NOT be cached; content changes on every request.
- `short-lived` — cache for up to 1 hour.
- `medium` — cache for up to 1 day.
- `long-lived` — cache for up to 1 week.
- `permanent` — cache indefinitely until the document changes.

A custom duration string (e.g., `4h`, `30d`) is also accepted. Format: number + unit (`s`, `m`, `h`, `d`, `w`). Distinct from `stability` (content reliability) and `lifecycle` (development phase).

```yaml
mx:
  cacheability: long-lived
```

---

### 6.11 `readingLevel`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Valid values** | beginner, intermediate, advanced, expert |

Content reading level. Helps agents recommend content based on user expertise.

```yaml
mx:
  readingLevel: intermediate
```

---

### 6.12 `runbook`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |

Operational instructions for agents. Describes how to interpret and act on this document. The runbook SHOULD be written in imperative voice and SHOULD be specific enough that an agent can act on it without reading the full document body. Particularly valuable for machine-facing documents where the document structure is not self-evident.

```yaml
mx:
  runbook: "Parse the fields array. Each entry has name, type, definition, status, and profile."
```

---

### 6.13 `confidential`

| Property | Value |
|----------|-------|
| **Type** | boolean |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | false |

Whether this record must be excluded from public outputs. Authors SHOULD only declare this field when marking content as confidential.

```yaml
mx:
  confidential: true
```

---

### 6.14 `inherits`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Path to the file this document extends. The inheriting file adds MX metadata on top of the target's content. The target can be any file type (`.md`, `.html`, `.json`, `.yaml`, etc.). The inheriting file extends the target; it does not replace it. The path MAY be relative or absolute. For folder-level field inheritance, an `inheritable` field on the parent folder's metadata (out of scope for this note) is the preferred mechanism.

```yaml
mx:
  inherits: "README.md"
```

---

### 6.15 `ld`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Inline JSON-LD (Schema.org) expressed in YAML frontmatter. Enables structured data without a separate `<script>` block. Implementations rendering HTML from this metadata SHOULD convert the `ld` object to a `<script type="application/ld+json">` block.

```yaml
mx:
  ld:
    "@context": "https://schema.org"
    "@type": "TechArticle"
    name: "MX Core Metadata note"
```

---

### 6.16 `hold`

| Property | Value |
|----------|-------|
| **Type** | boolean |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |
| **Default** | false |

Publication-block flag. When `true`, the document is otherwise ready but is being held back from release, send, or merge pending an external decision. Distinct from `status: draft`, which means "still in progress"; `hold: true` says "complete or near-complete, but blocked from going out".

The flag is for human-readable workflow signalling. Implementations MAY use it to suppress publication, send, or notification automations; consumers SHOULD treat the absence of `hold` (and `hold: false`) as "not held". When `hold` is `true`, `holdReason` SHOULD be provided.

```yaml
mx:
  hold: true
```

---

### 6.17 `holdReason`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Free-text companion to `hold`. Records why the document is being held back and, where possible, the condition that will release it. Single sentence in present tense.

`holdReason` is meaningful only when `hold: true`. Setting `holdReason` without `hold` is permitted but SHOULD be treated by implementations as a hold not yet activated.

```yaml
mx:
  hold: true
  holdReason: "Tom holding the draft set overnight; release decision 1 May."
```

---

## 7. External alignments

Some MX fields carry values whose semantics are owned, in whole or in part, by an external standard. The `Owns semantics` column in the inventory below records the relationship:

- **`external`** — the external vocabulary fully owns the value's meaning. MX provides the YAML key as a convenience so authors don't have to switch syntax mid-frontmatter; tooling treats the field as the aligned external field. (Older drafts called this "pass-through".)
- **`mx`** — MX names both the field and the value semantics, but models them after a recognised external standard so existing tools can interoperate. The field name is MX's; the alignment is informational.
- **`aligned`** — MX names the field and its abstract semantics, with a normative requirement that the value be expressible in the cited external vocabulary. The field is MX's; the value's structure must agree with the standard.

### 7.1 Inventory

| Field | Aligns with | Owns semantics | Definition | Conformance |
|-------|-------------|:--------------:|-----------|:-----------:|
| `date` | Dublin Core `dc:date`, Schema.org `Date` | external | ISO 8601 date for a generic reference event distinct from `created` / `modified`. | MAY |
| `duration` | Schema.org `duration` | external | ISO 8601 duration (e.g. `PT1H30M`). For time-based content. | MAY |
| `format` | Dublin Core `dc:format`, Schema.org `encodingFormat` | external | Media type or file format string. | MAY |
| `rights` | Dublin Core `dc:rights`, Schema.org `license` | external | Free-form rights statement when an SPDX identifier does not fit. | MAY |
| `displayName` | FOAF `displayName`, Schema.org `alternateName` | external | Citation form when it differs from `title`. | MAY |
| `usage` | Schema.org `usageInfo` | external | Guidance on how the document is meant to be consumed. | MAY |
| `proofOfAuthorship` | W3C Verifiable Credentials, Signed Exchanges | mx | Cryptographic claim that the named author wrote this. | MAY |
| `integritySignature` | RFC 9421 HTTP Message Signatures, Subresource Integrity | mx | Hash or signature over the artefact's content. | MAY |
| `provenancePedigree` | W3C PROV-O, C2PA Content Credentials | mx | Derivation chain back to the source. | MAY |
| `canonicalUri` | RFC 3986, Schema.org `mainEntityOfPage`, Dublin Core `identifier` | aligned | Canonical location of this document, expressed as a URI (see §7a.1). | MUST (Level 2) |
| `conformsTo` | Dublin Core `conformsTo` | aligned | Standards this document declares conformance to (see §7a.4). | MUST (Level 2) |

### 7.2 Adding a new externally-aligned field

A new field in this inventory SHOULD be proposed only when there is value in surfacing the key in MX frontmatter and the alignment relationship is clear. The proposal MUST:

- Cite the external standard precisely (URL plus the specific field/property name).
- Declare which `Owns semantics` value applies (`external`, `mx`, or `aligned`).
- Declare the alignment in canon via `alignsWith:` so converters and validators can route the value to the external standard's schema.
- For `external` semantics: NOT redefine the value's meaning. MX's role is to carry the key.
- For `mx` and `aligned` semantics: name the alignment explicitly so a future deviation is documented at the proposal stage rather than discovered at integration.

### 7.3 Tiebreaker — when external semantics almost fit (Normative)

A recurring authoring question: an external standard's field nearly captures the value MX needs, but not exactly. The rule:

- **Exact fit** — declare the field with `Owns semantics: external` and defer entirely. MX MUST NOT redefine the value's meaning.
- **Close fit, no deviation needed** — declare the field with `Owns semantics: aligned`. MX names the abstract semantics; values must be expressible in the external vocabulary. No re-interpretation.
- **Close fit, deviation needed** — declare the field with `Owns semantics: mx` and document the deviation in the field's definition. The proposal MUST name the specific point at which MX departs from the external standard, the reason for the deviation, and the migration path if the external standard later closes the gap.
- **No fit** — the field is not externally aligned. Define it in §5 (Zone 1) or §6 (Zone 2) with full MX semantics; do not list it in this inventory.

MX MUST NOT silently override an external standard's semantics. A deviation is documented at the proposal stage or it is not a deviation — it is a bug.

---

## 7a. Document discovery and lifecycle

This section covers document discovery, version chains, lifecycle dates, machine-readable affordances, semantic identifiers, and consumption policy. Each field has a direct analogue in Dublin Core or Schema.org and applies to any MX-governed document regardless of vendor. The fields are written into the YAML frontmatter under Zone 2 (the `mx:` object) and, where the carrier supports XMP metadata (e.g. tagged PDFs under ISO 14289-1), should be propagated into the carrier's metadata packet so they survive copying and reformatting.

### 7a.1 Identity and provenance

| Field | Zone | Type | Definition | Analogue |
|-------|:-:|------|-----------|----------|
| `canonicalUri` | 2 | URI (RFC 3986) | Canonical location of this document, expressed as a URI. The scheme tells the receiver what kind of location it is: `file://` for a carrier that lives on disk, `smb://` / `nfs://` / internal `https://` for a carrier reachable inside a network, a publicly resolvable `https://` for a carrier on the public internet. | Schema.org `mainEntityOfPage`, Dublin Core `identifier` |
| `supersedes` | 2 | string-or-array | URL or identifier of an earlier document this one replaces. | Dublin Core `replaces` |
| `supersededBy` | 2 | string | URL of the later document that replaces this one. The inverse of `supersedes`. | Dublin Core `isReplacedBy` |

`canonicalUri` carries the carrier's authoritative location at every tier; the URI scheme tells the receiver how to interpret it. A cog that lives only on disk SHOULD use a `file://` URI naming the carrier. A cog served from a network volume or internal corporate application SHOULD use the URI reachable inside the relevant network (`smb://`, `nfs://`, or an internal `https://`). When a cog is hosted in a publicly reachable source repository or website, the value MUST be that public URI: without it, an agent holding a copy cannot find the authoritative version.

**Agreement with HTML's `<link rel="canonical">` (Normative).** For HTML carriers, the URL declared by the page's `<link rel="canonical">` element and the value of the `mx:canonical-uri` `<meta>` tag (the kebab-case HTML rendering of `mx.canonicalUri` per the MX Carrier Formats note) MUST agree. Verifiers MAY treat either form as authoritative. A page that declares both with disagreeing URLs is a conformance failure: tools cannot reconcile two contradictory canonical claims and SHOULD flag the page as ambiguous rather than guess. When only one form is present, that form is authoritative.

### 7a.2 Lifecycle dates

| Field | Zone | Type | Definition | Analogue |
|-------|:-:|------|-----------|----------|
| `expires` | 2 | string | ISO-8601 date after which the document's content is no longer authoritative. Pricing PDFs, SLAs, compliance reports benefit from explicit expiry. | Dublin Core `valid` (end date) |
| `reviewBy` | 2 | string | ISO-8601 date by which the document is scheduled for the next editorial review. Distinct from `expires`; review may confirm continued validity. | (none direct) |

### 7a.3 Action affordances

| Field | Zone | Type | Definition | Analogue |
|-------|:-:|------|-----------|----------|
| `relatedDocs` | 2 | array | List of URLs an agent should consider fetching for full context on this document. | Dublin Core `relation` |
| `supportContact` | 2 | string | Contact address (mailto, URL, or organisation reference) for follow-up questions. | Schema.org `contactPoint` |

### 7a.4 Semantics and structure

| Field | Zone | Type | Definition | Analogue |
|-------|:-:|------|-----------|----------|
| `summary` | 2 | string | One-to-two-sentence machine-summary of the document. Targets agents that need to decide whether the artefact is relevant before reading the body. | Schema.org `abstract` |
| `topic` | 2 | array | Controlled-vocabulary identifiers for the topics the document covers (Wikidata QIDs, Schema.org Concept URLs). | Schema.org `about`, Dublin Core `subject` |
| `entities` | 2 | array | Named entities (people, organisations, products) the document is about, given as stable identifiers (Wikidata QIDs, ORCID IDs, organisation domain names). | Schema.org `mentions` |
| `speakable` | 2 | string | Optional inline summary suitable for a voice agent to read aloud. | Schema.org SpeakableSpecification |
| `conformsTo` | 2 | array | Standards this document declares conformance to, as URIs or stable identifiers. Centralises what would otherwise be spread across many separate fields. | Dublin Core `conformsTo` |

### 7a.5 Consumption policy (negative space)

| Field | Zone | Type | Definition |
|-------|:-:|------|-----------|
| `trainingDataPolicy` | 2 | string | Whether this artefact may be included in AI training corpora. The flip side of robots.txt, embedded so it survives copying and syndication. |
| `doNotIndex` | 2 | boolean | When `true`, the document should not be added to public indices (search, llms.txt, sitemap) even where it is technically reachable. Embedded equivalent of robots.txt `noindex`. |

### 7a.6 Conformance levels

Four of the fourteen fields are MUST at Level 2 of the MX Core profile: `canonicalUri`, `summary`, `conformsTo`, `trainingDataPolicy`. The reasoning, in each case:

- `canonicalUri` — without it, agents holding a copy of the artefact cannot find the current authoritative version. The URI scheme also tells the receiver what kind of location it is: `file://` for on-disk, an internal scheme for network-reachable, a publicly resolvable `https://` for public. The single highest-value field of the standard.
- `summary` — without it, agents read the whole document to decide if it is relevant. The energy and inference saving is direct.
- `conformsTo` — without it, conformance claims are scattered across many implicit fields. Centralising them lets an agent read one field and know which contracts the document claims.
- `trainingDataPolicy` — AI training corpora routinely ingest public documents. An explicit policy is the only way for a publisher to express a position the consumer can act on.

The remaining ten fields are SHOULD at Level 2 and MAY at Level 1.

### 7a.7 XMP rendering

For carriers that support XMP metadata (PDFs, audio, video), the same fields should be written into the XMP packet using a registered namespace. The reference implementation in CogNovaMX uses `https://schemas.cognovamx.com/mx/1.0/` with the prefix `mx`, so a tagged PDF carries XMP properties such as `mx:CanonicalUri`, `mx:Summary`, `mx:ConformsTo`. The Gathering may wish to propose a vendor-neutral namespace once the field set is ratified.

---

## 8. Conformance summary

| Field | Zone | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|:-:|
| `title` | 1 | MUST | — | — |
| `description` | 1 | MUST | — | — |
| `originator` | 1 | MUST | — | — |
| `created` | 1 | MUST | — | — |
| `modified` | 1 | MUST | — | — |
| `version` | 1 | — | SHOULD | — |
| `schema` | 1 | — | — | MAY |
| `validatesAgainst` | 1 | — | — | MAY |
| `status` | 2 | — | SHOULD | — |
| `contentType` | 2 | — | SHOULD | — |
| `runbook` | 2 | — | SHOULD | — |
| `tags` | 2 | — | — | MAY |
| `audience` | 2 | — | — | MAY |
| `purpose` | 2 | — | — | MAY |
| `license` | 2 | — | — | MAY |
| `stewardship` | 2 | — | — | MAY |
| `domain` | 2 | — | — | MAY |
| `segment` | 2 | — | — | MAY |
| `cacheability` | 2 | — | — | MAY |
| `readingLevel` | 2 | — | — | MAY |
| `confidential` | 2 | — | — | MAY |
| `inherits` | 2 | — | — | MAY |
| `ld` | 2 | — | — | MAY |
| External alignments (§7) | varies | — | — | MAY |
| `canonicalUri` | 2 | — | MUST | — |
| `summary` | 2 | — | MUST | — |
| `conformsTo` | 2 | — | MUST | — |
| `trainingDataPolicy` | 2 | — | MUST | — |
| Legacy aliases: `author` → `originator`; `maintainer` → `stewardship.steward`; `ownership` → `stewardship.legalEntity` (and sub-keys per §6.6). Supported for one major version after this note ratifies. | | | | |

---

## 9. Security and privacy considerations

- The `confidential` field controls whether a document is excluded from public outputs. Implementations that generate public artefacts MUST respect this field.
- The `originator` field is immutable and establishes provenance. Implementations MUST NOT allow modification of this field after initial creation. The legacy `author` alias inherits the same immutability.
- The `stewardship.accountableContact` sub-key (§6.6) is typically a public contact address. Implementations that publish metadata publicly SHOULD verify the value is appropriate for public exposure (an organisational mailbox or generic role address rather than a personal one) before rendering it.

---

## 10. References

### 10.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) — Date and time format
- [SPDX Licence List](https://spdx.org/licenses/) — Software Package Data Exchange licence identifiers

### 10.2 Informative references

- [Schema.org Style Guide](https://schema.org/docs/styleguide.html) — Vocabulary naming conventions
- [Dublin Core DCMI Namespace](https://www.dublincore.org/specifications/dublin-core/dcmi-namespace/) — Namespace governance model
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (conformance level model)

---

## 11. Author quickstart (Informative)

Three minimum-viable frontmatter blocks an author can copy and adapt.

### 11.1 Minimum-viable Level 1 — a blog post

The smallest valid Level 1 frontmatter. Identifies and attributes; does not classify lifecycle.

```yaml
---
title: "How we hit Level 2 in a fortnight"
description: "What it took to retrofit MX core metadata onto an existing site."
originator: "Tom Cranstoun"
created: 2026-04-15
modified: 2026-05-07
---
```

### 11.2 Minimum-viable Level 2 — a production document

Level 2 adds lifecycle, classification, the four MUST-at-Level-2 web fields, and stewardship.

```yaml
---
title: "How we hit Level 2 in a fortnight"
description: "What it took to retrofit MX core metadata onto an existing site."
originator: "Tom Cranstoun"
created: 2026-04-15
modified: 2026-05-07
version: "1.0"
mx:
  status: published
  contentType: info-doc
  runbook: "Read top to bottom; the table in section 3 carries the punch list."
  canonicalUri: https://mx.allabout.network/blog/level-2-in-a-fortnight
  summary: "A retrofit playbook from a real adoption: which fields landed first, which failed, and why."
  conformsTo: [https://mx.allabout.network/cog.html]
  trainingDataPolicy: "Permitted with attribution."
  stewardship:
    steward: "Tom Cranstoun"
    accountableContact: "info@cognovamx.com"
---
```

### 11.3 Minimum-viable Level 3 — a fully-described reference

Level 3 adds the discretionary classification fields. The example is a field-dictionary entry in a registry.

```yaml
---
title: "MX Core field — originator"
description: "Specification entry for the originator field in MX Core."
originator: "Tom Cranstoun"
created: 2026-05-07
modified: 2026-05-07
version: "1.0"
schema: ./schemas/mx-field-entry.v1.yaml
mx:
  status: canonical
  contentType: field-dictionary
  runbook: "The field-dictionary entry for originator. See §5.3 of the MX Core Metadata note for the full definition."
  canonicalUri: https://mx.allabout.network/fields/originator
  summary: "Immutable Zone 1 field naming the document's original creator."
  conformsTo: [https://mx.allabout.network/cog.html]
  trainingDataPolicy: "Permitted with attribution."
  tags: [mx-core, field, identity]
  audience: [humans, machines]
  purpose:
    kind: reference
    subPurpose: "field dictionary entry"
  license: MIT
  domain: "machine-experience"
  cacheability: long-lived
  readingLevel: intermediate
  stewardship:
    steward: "Tom Cranstoun"
    accountableContact: "info@cognovamx.com"
    legalEntity: "Digital Domain Technologies Ltd"
    brand: "CogNovaMX"
---
```

---

## 12. Common authoring mistakes (Informative)

Five shapes a fresh author often produces, each with the fix.

**Mistake 1: keeping `author` after rename.**

```yaml
# WRONG — author is a deprecated alias
author: "Tom Cranstoun"
```

The canonical Zone 1 identity field is `originator` (§5.3). `author` remains as a one-major-version alias, but new documents SHOULD declare `originator` directly.

**Mistake 2: using `ownership` as a string.**

```yaml
# WRONG — string-or-object polymorph is retired
mx:
  ownership: "Tom Cranstoun"
```

The polymorphic shape is gone. Use the `stewardship` object with explicit sub-keys (§6.6):

```yaml
# RIGHT
mx:
  stewardship:
    legalEntity: "Tom Cranstoun"
```

**Mistake 3: out-of-matrix `status`.**

```yaml
# WRONG — `closed` is not a valid status for a blog post
mx:
  contentType: info-doc
  status: closed
```

`closed` is workflow lifecycle (§6.1.1). For an info-doc, draw from the document lifecycle row: `draft`, `active`, `published`, `archived`, `canonical`, `unknown`.

**Mistake 4: silent override of an external standard.**

```yaml
# WRONG — `format` is externally-aligned (Owns semantics: external); MX does not redefine its values
mx:
  format: "long-form"   # Not a valid Schema.org / Dublin Core format value
```

Externally-aligned fields with `Owns semantics: external` (§7.1) defer entirely to the external vocabulary. `format` expects a media type or recognised file format identifier. If a value the external standard does not accept is genuinely needed, the field is the wrong field — see §7.3 (Tiebreaker).

**Mistake 5: claiming Level 2 without the four MUST web fields.**

```yaml
# WRONG — Level 2 requires canonicalUri, summary, conformsTo, trainingDataPolicy
mx:
  status: published
  contentType: info-doc
  runbook: "..."
```

A document at Level 2 MUST declare `canonicalUri`, `summary`, `conformsTo`, and `trainingDataPolicy` (§7a.6). Without them, the document is at Level 1, not Level 2 — declaring `status: published` is not the conformance signal.

---

*End of MX Core Metadata note draft.*
