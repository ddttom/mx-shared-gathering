---
title: "MX Core Metadata Standard"
version: "1.2-proposed"
created: 2026-04-02
modified: 2026-04-26
author: The Gathering
description: "Formal specification of core MX metadata fields — the foundational vocabulary every MX-aware document must, should, or may declare. Machine-readable form: mx-canon/ssot/fields-data.yaml (sanitised open-standard core, ~103 fields)."

mx:
  status: proposed
  license: MIT
  partOf: mx-the-gathering
  contentType: specification
  buildsOn: [cog-unified-spec]
  tags: [standard, metadata, core, conformance, specification]
  audience: [humans, machines]
  cacheability: permanent
  runbook: "This is the core MX metadata standard. It defines Zone 1 identity fields, Zone 2 operational fields, and cog structural fields. Use the conformance level tables to determine which fields are required at each level. Cross-reference companion standards for extensions, provenance, AI policy, and profile-specific fields."
---

# MX Core Metadata Standard

**Version:** 1.2-proposed
**Status:** Proposed (draft for Stream submission, awaiting community review)
**Date:** 26 April 2026
**Governing body:** The Gathering
**License:** MIT

---

## 1. Abstract

This document defines the core metadata vocabulary for the Machine Experience (MX) framework. It specifies the foundational fields that every MX-aware document — whether a cog file, a folder metadata file, or any carrier-format document — must, should, or may declare.

The core vocabulary is organised into three groups: Zone 1 identity fields (top-level document identity), Zone 2 operational fields (governance, classification, and distribution metadata under the `mx:` namespace), and cog structural fields (fields specific to the cog document format).

This standard establishes the conformance level framework (Level 1, Level 2, Level 3) that all companion MX standards adopt by reference.

**Machine-readable source.** The canonical field dictionary for this standard is [`mx-canon/ssot/fields-data.yaml`](../../../mx-canon/ssot/fields-data.yaml) — a sanitised core of ~103 fields covering identity, classification, relationships, lifecycle, machine-readability infrastructure, folder metadata, cog contract, and non-YAML markup carriers. Carrier-format schemas (code, database, media) live in the companion [`mx-canon/ssot/fields-data-carriers.yaml`](../../../mx-canon/ssot/fields-data-carriers.yaml) as specified in MXS-04. Vendor extensions (e.g. `cognovamx-fields.yaml`) are out of scope for this core standard.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

### 2.1 Conformance levels

This standard defines three conformance levels. All companion MX standards ([MX Extensions Standard](mxs-02-extensions.cog.md), [MX Provenance Standard](mxs-03-provenance.cog.md), [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md) *(deferred)*, [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) *(deferred)*) adopt these levels by reference.

| Level | Name | Requirement | Description |
|-------|------|-------------|-------------|
| Level 1 | **MX Core** | All MUST fields present and valid | Minimum viable MX metadata. A document at Level 1 is machine-identifiable and attributable. |
| Level 2 | **MX Standard** | All MUST and SHOULD fields present | Recommended for production use. A document at Level 2 is fully classified and lifecycle-managed. |
| Level 3 | **MX Complete** | All MUST, SHOULD, and applicable MAY fields present | Full metadata coverage. A document at Level 3 provides maximum context for agents and tools. |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels. A document claiming "MX Standard" (Level 2) MUST also satisfy all "MX Core" (Level 1) requirements.

### 2.2 Draft status

This document is a proposed standard under draft by The Gathering. It is authored for submission to the Stream public review process and awaits community ratification. Until ratified, the field definitions, conformance requirements, and normative rules in this document are the working draft — stable enough to build against, expected to evolve through review.

---

## 3. Scope and relationship to other standards

### 3.1 What this document covers

This document specifies:

- **Zone 1 identity fields** — top-level document identity (title, description, author, dates, version)
- **Zone 2 core operational fields** — classification, governance, and distribution metadata
- **Cog structural fields** — fields specific to the cog document format (.cog.md files)
- **The conformance level framework** — Level 1/2/3 definitions

### 3.2 What this document does not cover

The following are defined in companion standards:

| Topic | Standard |
|-------|----------|
| Namespace policy, carrier formats, extension mechanisms | [MX Extensions Standard](mxs-02-extensions.cog.md) |
| Trust, attribution, verification, decision records | [MX Provenance Standard](mxs-03-provenance.cog.md) |
| AI governance, agent policies, training controls | [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md) *(deferred)* |
| Content-type-specific fields (code, media, database, etc.) | [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) *(deferred)* |

### 3.3 Relationship to existing standards

This standard builds upon:

- **[Cog Unified Specification](../specifications/cog-unified-spec.cog.md)** — defines the cog file format that these fields populate
- **[MX Standards Alignment](../specifications/mx-standards-alignment.cog.md)** — documents how MX conventions align with Schema.org, Dublin Core, and Open Graph
- **[NDR-02: camelCase Naming](../naming-decisions/ndr-02-camelcase-naming.cog.md)** — all field names in this standard use camelCase
- **[NDR-03: Spelling Neutrality](../naming-decisions/ndr-03-spelling-neutrality.cog.md)** — field names avoid regional spelling variants
- **ISO 8601** — all date fields use YYYY-MM-DD format
- **SPDX License List** — licence identifiers follow the SPDX standard

---

## 4. Terminology

- **Cog** — A `.cog.md` file. The atomic unit of the MX document system. One file, many block types. Defined by the [Cog Unified Specification](../specifications/cog-unified-spec.cog.md).
- **Zone 1** — Top-level YAML frontmatter fields (outside the `mx:` object). Document identity fields.
- **Zone 2** — Fields nested under the `mx:` object in YAML frontmatter. MX-operational fields.
- **Profile** — A named set of fields applicable to a specific document type (e.g., `core`, `cog`, `folder`, `book`).
- **Carrier format** — The mechanism by which a non-markdown file carries MX metadata (HTML meta tags, JSDoc comments, CSS comments, etc.). Defined in the [MX Extensions Standard](mxs-02-extensions.cog.md).
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
title: "MX Core Metadata Standard"
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
- The field name uses the spelling-neutral form `license` (SPDX standard spelling), following [NDR-03](../naming-decisions/ndr-03-spelling-neutrality.cog.md).

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
- For folder-level field inheritance, use `inheritable` on the parent folder instead (see [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) *(deferred)*, folder profile).

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
    name: "MX Core Metadata Standard"
```

**Normative notes:**

- The `ld` field provides a YAML representation of JSON-LD structured data.
- Implementations rendering HTML from this metadata SHOULD convert the `ld` object to a `<script type="application/ld+json">` block.

---

## 8. Field definitions — Cog structural fields

The following fields are specific to cog documents (`.cog.md` files). They provide registry classification, dependency management, and structural composition. All cog structural fields reside in Zone 2 (the `mx:` object).

### 8.1 `category`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MUST (Level 1) for cog documents |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Cog category. Determines registry grouping.

**Example:**

```yaml
mx:
  category: standard
```

**Normative notes:**

- Common values: `mx-core`, `mx-tool`, `mx-contact`, `mx-ops`, `mx-content`.
- This field is REQUIRED for all cog files.

---

### 8.2 `partOf`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MUST (Level 1) for cog documents |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Parent collection, suite, or initiative.

**Example:**

```yaml
mx:
  partOf: mx-the-gathering
```

**Normative notes:**

- This field is REQUIRED for all cog files. Every cog MUST belong to a named collection.

---

### 8.3 `buildsOn`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Level 2) for cog documents |
| **Default** | *(empty array)* |

**Definition:** Context graph. Array of cog names this document builds upon. Soft dependency — provides context, not a hard requirement.

**Example:**

```yaml
mx:
  buildsOn: [what-is-a-cog, cog-unified-spec]
```

**Normative notes:**

- `buildsOn` forms the cog knowledge graph. It is a soft dependency — the referenced cogs provide context but are not required for this cog to function.
- Distinct from `requires` (hard dependency) and `refersTo` (informational link).

---

### 8.4 `requires`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Level 2) for cog documents |
| **Default** | *(empty array)* |

**Definition:** Hard dependencies. Array of cog names that MUST exist for this cog to function.

**Example:**

```yaml
mx:
  requires: [node-runtime, markdownlint-cli2]
```

**Normative notes:**

- A cog MUST NOT be considered functional if any of its `requires` entries are missing.
- Distinct from `buildsOn` (soft context dependency).

---

### 8.5 `refersTo`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(empty array)* |

**Definition:** Related cogs or external resources. Informational links, not dependencies.

**Example:**

```yaml
mx:
  refersTo: [field-dictionary, cog-unified-spec]
```

---

### 8.6 `includes`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(empty array)* |

**Definition:** Cog composition — content reuse without duplication. Array of include declarations, each specifying a source cog, optional block filter, and resolution mode.

**Example:**

```yaml
mx:
  includes:
    - source: "shared/validation-policy.cog.md"
      blocks: [sop]
      resolution: build
```

**Normative notes:**

- Structural composition. Included content is merged into the cog; the including cog's own content overrides included content of the same type.
- Distinct from `buildsOn` (soft recommendation) and `requires` (hard dependency).
- See the [Cog Unified Specification](../specifications/cog-unified-spec.cog.md) for the full inclusion specification.

---

### 8.7 `blocks`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Valid values** | prose, action, definition, essence, provenance, version, code, html, sop, security |
| **Default** | *(none)* |

**Definition:** Declares block types present in the document body.

**Example:**

```yaml
mx:
  blocks: [prose, definition, code]
```

**Normative notes:**

- Block types are defined in the [Cog Unified Specification](../specifications/cog-unified-spec.cog.md).
- The `prose` block is implicit in every cog and need not be declared.

---

### 8.8 `execute`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Action block. Contains runtime, command, actions, and policy. Its presence makes a cog an action-doc.

**Example:**

```yaml
mx:
  execute:
    runtime: node
    command: "node scripts/validate.js"
```

**Normative notes:**

- The presence of an `execute` field distinguishes action-docs from info-docs.
- Action cogs SHOULD live in the `scripts/` folder.

---

### 8.9 `policy`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Content handling rules for agents. Inherited from uber docs via effective doc resolution.

**Example:**

```yaml
mx:
  policy: "inherit from parent uber-doc"
```

---

### 8.10 `deliverable`

| Property | Value |
|----------|-------|
| **Type** | string-or-array |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** What this cog produces or delivers. Declares the tangible output — a report, a validated artefact, a trained team, a published page.

**Example:**

```yaml
mx:
  deliverable: "validated cog registered in REGINALD"
```

**Normative notes:**

- For action-docs, describes what running the actions produces.
- For info-docs, describes the knowledge or artefact the cog represents.
- The value MAY be an array for cogs with multiple deliverables.

---

### 8.11 `cogId`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Level 2) for cog documents |
| **Default** | *(derived from filename)* |

**Definition:** Unique cog identifier within the registry.

**Example:**

```yaml
mx:
  cogId: "field-dictionary"
```

**Normative notes:**

- Typically derived from the filename without the `.cog.md` extension.
- MUST be unique within the registry.

---

### 8.12 `cogType`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Level 2) for cog documents |
| **Valid values** | info, action, routing, certificate-of-genuineness |
| **Default** | *(none)* |

**Definition:** Cog type classification.

**Example:**

```yaml
mx:
  cogType: info
```

**Normative notes:**

- `info` — reference or documentation cog.
- `action` — has an `execute` block; performs operations.
- `routing` — agent navigation cog.
- `certificate-of-genuineness` — publisher-provenanced credential with a publisher block.

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

### 9.3 Cog structural fields

These conformance levels apply only to cog documents (`.cog.md` files).

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `category` | MUST | — | — |
| `partOf` | MUST | — | — |
| `buildsOn` | — | SHOULD | — |
| `requires` | — | SHOULD | — |
| `cogId` | — | SHOULD | — |
| `cogType` | — | SHOULD | — |
| `refersTo` | — | — | MAY |
| `includes` | — | — | MAY |
| `blocks` | — | — | MAY |
| `execute` | — | — | MAY |
| `policy` | — | — | MAY |
| `deliverable` | — | — | MAY |

---

## 10. Security and privacy considerations

- The `confidential` field controls whether a document is excluded from public outputs. Implementations that generate public artefacts MUST respect this field.
- The `author` field is immutable and establishes provenance. Implementations MUST NOT allow modification of this field after initial creation.
- The `ownership` field MAY contain contact information. Implementations that publish metadata publicly SHOULD consider whether `ownership.contact` values are appropriate for public exposure.

---

## 11. References

### 11.1 Normative references

- [Cog Unified Specification](../specifications/cog-unified-spec.cog.md) — the cog document format
- [NDR-02: camelCase Naming](../naming-decisions/ndr-02-camelcase-naming.cog.md) — field naming convention
- [NDR-03: Spelling Neutrality](../naming-decisions/ndr-03-spelling-neutrality.cog.md) — spelling-neutral vocabulary
- [ADR-02: Namespace Policy](../architecture-decisions/adr-02-namespace-policy.cog.md) — namespace governance
- [MX Extensions Standard](mxs-02-extensions.cog.md) — namespace policy, carrier formats, extension mechanisms
- [MX Provenance Standard](mxs-03-provenance.cog.md) — trust, attribution, verification
- [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md) *(deferred)* — AI governance and agent policies
- [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) *(deferred)* — content-type-specific fields

### 11.2 Informative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) — Date and time format
- [SPDX Licence List](https://spdx.org/licenses/) — Software Package Data Exchange licence identifiers
- [Schema.org Style Guide](https://schema.org/docs/styleguide.html) — Vocabulary naming conventions
- [Dublin Core DCMI Namespace](https://www.dublincore.org/specifications/dublin-core/dcmi-namespace/) — Namespace governance model
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (conformance level model)
- [MX Standards Alignment](../specifications/mx-standards-alignment.cog.md) — how MX conventions align with existing web standards

---

## 12. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-draft | 2026-04-02 | Initial draft. Initial draft. |
| 1.2-proposed | 2026-04-26 | Added Zone 1 fields `schema` (§6.7) and `validatesAgainst` (§6.8) — schema-reference and conformance-claim fields imported from cog-spec v1.0 (mx-upgraded-reginald). Both at conformance level MAY. |
