---
title: "MX Provenance Standard"
version: "1.0-proposed"
created: 2026-04-02
modified: 2026-04-16
author: The Gathering
description: "Formal specification of MX provenance, trust, and verification metadata — attribution, quality assurance, maintenance tracking, and decision record references."

mx:
  status: proposed
  license: MIT
  partOf: mx-the-gathering
  contentType: specification
  buildsOn: [cog-unified-spec, mxs-01-core-metadata]
  tags: [standard, provenance, trust, attribution, verification, specification]
  audience: [humans, machines]
  cacheability: permanent
  runbook: "This is the MX provenance standard. It defines attribution fields, quality and trust fields, maintenance fields, decision record references, and migration fields. Use the conformance level tables to determine which fields are required at each level. Cross-reference the MX Core Metadata Standard for the conformance level framework and Zone 1/Zone 2 identity fields."
---

# MX Provenance Standard

**Version:** 1.0-proposed
**Status:** Proposed (draft for Stream submission, awaiting community review)
**Date:** 16 April 2026
**Governing body:** The Gathering
**License:** MIT

---

## 1. Abstract

This document defines the provenance, trust, and verification metadata vocabulary for the Machine Experience (MX) framework. It specifies the fields that establish who created content, how it was created, how trustworthy it is, who maintains it, and what governance decisions shaped it.

The provenance vocabulary is organised into five groups: attribution fields (authorship and origin), quality and trust fields (reliability commitments), maintenance fields (ongoing stewardship), decision record references (governance links), and migration fields (content relocation tracking).

This standard adopts the conformance level framework (Level 1, Level 2, Level 3) defined by the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md).

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

### 2.1 Conformance levels

This standard adopts the three conformance levels defined by the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md):

| Level | Name | Provenance requirement |
|-------|------|------------------------|
| Level 1 | **MX Core** | Author attribution present (`provenanceAuthor` or `author` from Core standard) |
| Level 2 | **MX Standard** | Adds `provenanceOrigin`, `maintainedDate`, and `reviewCycle` declared |
| Level 3 | **MX Complete** | Full trust chain including `publisher`, `accuracyCommitment`, `correctionSla`, decision records, and quality triad |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Draft status

This document is a proposed standard under draft by The Gathering. It is authored for submission to the Stream public review process and awaits community ratification. Until ratified, the field definitions, conformance requirements, and normative rules in this document are the working draft — stable enough to build against, expected to evolve through review.

---

## 3. Scope and relationship to other standards

### 3.1 What this document covers

This document specifies:

- **Attribution fields** — authorship, publishing entity, content origin, source material, and generation instructions
- **Quality and trust fields** — accuracy commitments, review cycles, compliance levels, and the quality triad
- **Maintenance fields** — freshness tracking, maintainer identity, expiry, and update triggers
- **Decision record references** — links to architecture, naming, and business decision records
- **Migration fields** — content relocation tracking

### 3.2 What this document does not cover

The following are defined in companion standards:

| Topic | Standard |
|-------|----------|
| Zone 1 identity fields, Zone 2 core operational fields, cog structural fields | [MX Core Metadata Standard](mxs-01-core-metadata.cog.md) |
| Namespace policy, carrier formats, extension mechanisms | [MX Extensions Standard](mxs-02-extensions.cog.md) |
| AI governance, agent policies, training controls | [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md) |
| Content-type-specific fields (code, media, database, etc.) | [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) |

### 3.3 Relationship to existing standards

This standard builds upon:

- **[MX Core Metadata Standard](mxs-01-core-metadata.cog.md)** — defines the conformance level framework and foundational fields (including `author` in Zone 1)
- **[Cog Unified Specification](../specifications/cog-unified-spec.cog.md)** — defines the cog file format that these fields populate
- **[NDR-02: camelCase Naming](../naming-decisions/ndr-02-camelcase-naming.cog.md)** — all field names in this standard use camelCase
- **[NDR-03: Spelling Neutrality](../naming-decisions/ndr-03-spelling-neutrality.cog.md)** — field names avoid regional spelling variants
- **ISO 8601** — all date fields use YYYY-MM-DD format
- **WCAG 2.1** — accessibility compliance referenced in the quality triad

---

## 4. Terminology

- **Provenance** — The chain of authorship, origin, and publication that establishes who created content and how.
- **Trust chain** — The complete set of provenance, quality, and maintenance metadata that allows an agent to assess content reliability.
- **Quality triad** — The three quality dimensions: accessibility, semantic quality, and convergence.
- **Decision record** — A formal record (ADR, NDR, or BDR) documenting an architectural, naming, or business decision that governs this content.
- **Zone 1** — Top-level YAML frontmatter fields (outside the `mx:` object). Defined by the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md).
- **Zone 2** — Fields nested under the `mx:` object in YAML frontmatter.
- **Profile** — A named set of fields applicable to a specific document type (e.g., `core`, `cog`, `report`, `migration`).

---

## 5. Provenance in the two-zone model

Provenance fields reside in Zone 2 (the `mx:` object), complementing the Zone 1 `author` field defined by the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md).

The Zone 1 `author` field identifies the document creator. The Zone 2 provenance fields extend this with richer attribution: how the content was created (`provenanceOrigin`), who published it (`provenancePublisher`), and the original author if distinct from the Zone 1 `author` (`provenanceAuthor`).

```yaml
---
title: "Example Document"
author: "Tom Cranstoun"
created: 2026-04-02
modified: 2026-04-16

mx:
  provenanceAuthor: "Tom Cranstoun"
  provenancePublisher: "CogNovaMX Ltd"
  provenanceOrigin: human-directed
  maintainedDate: 2026-04-02
  maintainedBy: "Tom Cranstoun"
  reviewCycle: quarterly
  accuracyCommitment: "All factual claims verified against source documentation"
---
```

---

## 6. Field definitions — Attribution fields

### 6.1 `provenanceAuthor`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | core |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Person or entity who originally authored the document content.

**Example:**

```yaml
mx:
  provenanceAuthor: "Tom Cranstoun"
```

**Normative notes:**

- Part of the provenance group.
- At Level 1, a document MUST declare either `provenanceAuthor` or the Zone 1 `author` field from the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md). Both MAY be present.
- When both are present, `provenanceAuthor` identifies the original content author, while `author` identifies the document creator. These are often the same person.

---

### 6.2 `provenancePublisher`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | core |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Entity that published the document.

**Example:**

```yaml
mx:
  provenancePublisher: "CogNovaMX Ltd"
```

**Normative notes:**

- Part of the provenance group.
- Distinct from `publisher` (Section 6.5), which is a richer object used in Certificate of Genuineness cogs.

---

### 6.3 `provenanceOrigin`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | core |
| **Conformance** | SHOULD (Level 2) |
| **Valid values** | human-directed, ai-assisted, automated-fact-check |
| **Default** | *(none)* |

**Definition:** How the content was created. Declares the human-machine authorship balance.

**Example:**

```yaml
mx:
  provenanceOrigin: human-directed
```

**Normative notes:**

- Part of the provenance group.
- `human-directed` — content authored by a human, possibly with AI tooling assistance.
- `ai-assisted` — content substantially generated by AI, with human review and direction.
- `automated-fact-check` — content generated and verified by automated processes.
- This field relates to the AI generation fields in the [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md). Where `provenanceOrigin` declares how content was created, AI policy fields govern how agents may interact with it.

---

### 6.4 `source`

| Property | Value |
|----------|-------|
| **Type** | string-or-array |
| **Zone** | 2 (mx:) |
| **Profile** | cog, report |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Origin source material or input files. Declares the content provenance chain — what materials were used to create this document.

**Example:**

```yaml
mx:
  source: "brainstorm.md"
```

```yaml
mx:
  source:
    - "quarterly-report.csv"
    - "customer-feedback.md"
```

**Normative notes:**

- Content provenance, distinct from the provenance block type (which declares authorship).
- The value MAY be a single string or an array of strings.
- Paths MAY be relative or absolute.

---

### 6.5 `publisher`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Entity that publishes and stands behind this cog. Can be a string (name) or an object with `name`, `url`, and `contact` sub-fields.

**Example:**

```yaml
mx:
  publisher: "CogNovaMX Ltd"
```

```yaml
mx:
  publisher:
    name: "CogNovaMX Ltd"
    url: "https://allabout.network"
    contact: "info@cognovamx.com"
```

**Normative notes:**

- Distinct from `provenancePublisher` (Section 6.2), which is a flat provenance field. The `publisher` field supports richer structured data and is used in Certificate of Genuineness cogs.
- Used alongside `accuracyCommitment`, `reviewCycle`, and `mxCompliance` to form a complete trust declaration.

---

### 6.6 `generate`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Generation instructions for AI agents. Describes how to regenerate this content if lost or outdated.

**Example:**

```yaml
mx:
  generate:
    prompt: "Regenerate from source data"
    source: "quarterly-report.csv"
```

**Normative notes:**

- Part of MX OS metadata. If a document is lost, an agent with access to the `generate` instructions and source material can recreate it.
- The `prompt` sub-field SHOULD contain instructions specific enough for an agent to produce a faithful reconstruction.
- The `source` sub-field identifies the input data required for regeneration.

---

## 7. Field definitions — Quality and trust fields

### 7.1 `accuracyCommitment`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Publisher's commitment to content accuracy. A human-readable statement of the accuracy standard the publisher holds themselves to.

**Example:**

```yaml
mx:
  accuracyCommitment: "All factual claims verified against source documentation"
```

**Normative notes:**

- Part of the trust metadata group.
- Used alongside `publisher`, `reviewCycle`, and `correctionSla` to form a complete trust declaration.
- The value SHOULD be a clear, actionable statement that an agent or reader can use to assess the publisher's reliability.

---

### 7.2 `reviewCycle`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** How often cog content is reviewed for accuracy.

**Example:**

```yaml
mx:
  reviewCycle: quarterly
```

**Normative notes:**

- Part of the trust metadata group.
- Common values: `weekly`, `monthly`, `quarterly`, `annually`, `on-change`.
- Agents MAY use this field to assess whether content is likely to be current.

---

### 7.3 `correctionSla`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Service-level agreement for correcting errors. Sets expectations for error correction urgency.

**Example:**

```yaml
mx:
  correctionSla: "24 hours"
```

**Normative notes:**

- The value SHOULD be a human-readable duration string.
- This field communicates the publisher's responsiveness to reported errors, helping agents and consumers assess trust.

---

### 7.4 `mxCompliance`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** MX compliance level or detailed compliance metadata.

**Example:**

```yaml
mx:
  mxCompliance: "level-3"
```

```yaml
mx:
  mxCompliance:
    level: "level-3"
    lastAudit: 2026-04-01
    auditor: "Maxine"
```

**Normative notes:**

- Used alongside `publisher`, `accuracyCommitment`, and `reviewCycle` to form a complete trust declaration.
- When expressed as a string, the value corresponds to the conformance levels defined in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md).

---

### 7.5 `accessibility`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Accessibility compliance level. Part of the quality triad: accessibility, semantic, convergence.

**Example:**

```yaml
mx:
  accessibility: "WCAG 2.1 AA"
```

**Normative notes:**

- The quality triad measures three dimensions of document quality that are central to the MX philosophy.
- Accessibility measures how well the content serves users with disabilities, following established standards (WCAG 2.1, Pa11y).
- The value MAY be a string (compliance level) or an object with structured assessment data.

---

### 7.6 `semantic`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Semantic quality level. How well the cog uses semantic HTML and structured data.

**Example:**

```yaml
mx:
  semantic: "Schema.org JSON-LD"
```

**Normative notes:**

- Part of the quality triad: accessibility, semantic, convergence.
- Semantic quality measures how well the content uses structured data, semantic HTML, and machine-readable formats.
- The value MAY be a string (summary) or an object with structured assessment data.

---

### 7.7 `convergence`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Convergence score. How well the cog aligns human and machine experiences.

**Example:**

```yaml
mx:
  convergence: high
```

**Normative notes:**

- Part of the quality triad: accessibility, semantic, convergence.
- Core to MX's dual-experience philosophy. Convergence measures whether the same content serves both human readers and AI agents effectively.
- Common values when expressed as a string: `low`, `medium`, `high`.
- The value MAY be a string (summary) or an object with structured assessment data.

---

## 8. Field definitions — Maintenance fields

### 8.1 `maintainedDate`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Date the cog was last confirmed accurate by its maintainer. Helps agents assess content freshness.

**Example:**

```yaml
mx:
  maintainedDate: 2026-04-02
```

**Normative notes:**

- The value MUST use ISO 8601 date format (YYYY-MM-DD).
- Distinct from `modified` (Zone 1, defined in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md)). The `modified` field tracks when the document was last changed; `maintainedDate` tracks when it was last confirmed to still be accurate — even if no changes were needed.
- Agents SHOULD treat documents with a `maintainedDate` older than the `reviewCycle` as potentially stale.

---

### 8.2 `maintainedBy`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Person or team who last confirmed accuracy.

**Example:**

```yaml
mx:
  maintainedBy: "Tom Cranstoun"
```

**Normative notes:**

- Distinct from `maintainer` (defined in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md)), which identifies the person responsible for ongoing maintenance. `maintainedBy` records who performed the last accuracy confirmation — these may differ.

---

### 8.3 `expires`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Expiry date for time-sensitive cogs. Content SHOULD be reviewed or archived after this date.

**Example:**

```yaml
mx:
  expires: 2026-12-31
```

**Normative notes:**

- The value MUST use ISO 8601 date format (YYYY-MM-DD).
- Agents encountering expired content SHOULD flag it for review rather than treating it as authoritative.
- Distinct from `cacheability` (how long to cache a fetch) — `expires` indicates when the content itself is no longer current.

---

### 8.4 `updateTriggers`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(empty array)* |

**Definition:** Conditions that trigger a cog update. Events that should prompt a content review or revision.

**Example:**

```yaml
mx:
  updateTriggers:
    - specification-change
    - new-field-added
```

**Normative notes:**

- Values are context-specific strings describing trigger conditions.
- Agents and tooling MAY use these values to automate review notifications when trigger conditions are detected.

---

## 9. Field definitions — Decision record references

### 9.1 `adr`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Architecture Decision Record reference. Links to the ADR governing architectural choices that affect this document.

**Example:**

```yaml
mx:
  adr: "../architecture-decisions/adr-02-namespace-policy.cog.md"
```

```yaml
mx:
  adr:
    path: "../architecture-decisions/adr-02-namespace-policy.cog.md"
    date: 2026-03-15
    status: accepted
```

**Normative notes:**

- The value MAY be a string (path to the ADR) or an object with `path`, `date`, and `status` sub-fields.
- Decision records provide governance traceability — they explain why certain choices were made.

---

### 9.2 `ndr`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Naming Decision Record reference. Links to the NDR governing naming conventions that affect this document.

**Example:**

```yaml
mx:
  ndr: "../naming-decisions/ndr-02-camelcase-naming.cog.md"
```

```yaml
mx:
  ndr:
    path: "../naming-decisions/ndr-02-camelcase-naming.cog.md"
    date: 2026-03-10
    status: accepted
```

**Normative notes:**

- The value MAY be a string (path to the NDR) or an object with `path`, `date`, and `status` sub-fields.

---

### 9.3 `bdr`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Business Decision Record reference. Links to the BDR governing business decisions that affect this document.

**Example:**

```yaml
mx:
  bdr: "../business-decisions/bdr-003-hub-mount-table.cog.md"
```

```yaml
mx:
  bdr:
    path: "../business-decisions/bdr-003-hub-mount-table.cog.md"
    date: 2026-03-20
    status: accepted
```

**Normative notes:**

- The value MAY be a string (path to the BDR) or an object with `path`, `date`, and `status` sub-fields.

---

## 10. Field definitions — Migration fields

### 10.1 `movedFrom`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | migration |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Previous file path before content was relocated. Used for redirect generation and link maintenance.

**Example:**

```yaml
mx:
  movedFrom: "datalake/knowledge/reference/field-definitions.md"
```

**Normative notes:**

- The value SHOULD be the full relative path from the repository root.
- Tooling MAY use this field to generate redirects or update internal links automatically.
- SHOULD be paired with `movedDate` to record when the relocation occurred.

---

### 10.2 `movedDate`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | migration |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Date content was relocated.

**Example:**

```yaml
mx:
  movedDate: 2026-03-15
```

**Normative notes:**

- The value MUST use ISO 8601 date format (YYYY-MM-DD).
- SHOULD be paired with `movedFrom` to provide a complete migration record.

---

## 11. Conformance levels summary

### 11.1 Attribution fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `provenanceAuthor` | MUST | — | — |
| `provenancePublisher` | — | SHOULD | — |
| `provenanceOrigin` | — | SHOULD | — |
| `source` | — | — | MAY |
| `publisher` | — | — | MAY |
| `generate` | — | — | MAY |

### 11.2 Quality and trust fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `reviewCycle` | — | SHOULD | — |
| `accuracyCommitment` | — | — | MAY |
| `correctionSla` | — | — | MAY |
| `mxCompliance` | — | — | MAY |
| `accessibility` | — | — | MAY |
| `semantic` | — | — | MAY |
| `convergence` | — | — | MAY |

### 11.3 Maintenance fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `maintainedDate` | — | SHOULD | — |
| `maintainedBy` | — | — | MAY |
| `expires` | — | — | MAY |
| `updateTriggers` | — | — | MAY |

### 11.4 Decision record references

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `adr` | — | — | MAY |
| `ndr` | — | — | MAY |
| `bdr` | — | — | MAY |

### 11.5 Migration fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `movedFrom` | — | — | MAY |
| `movedDate` | — | — | MAY |

---

## 12. Security and privacy considerations

- The `provenanceAuthor` and `provenancePublisher` fields contain personally identifiable information. Implementations that publish metadata publicly SHOULD consider whether these values are appropriate for public exposure.
- The `publisher.contact` sub-field MAY contain email addresses or other contact information. Implementations SHOULD apply the same privacy considerations as for `ownership.contact` in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md).
- The `provenanceOrigin` field declares the human-machine authorship balance. Publishers SHOULD declare this field honestly to maintain trust with consumers and agents.
- Decision record references (`adr`, `ndr`, `bdr`) MAY link to internal governance documents. Implementations SHOULD verify that referenced documents are accessible to the intended audience before publishing these links externally.
- The `generate` field contains instructions for content regeneration. Implementations SHOULD ensure that generation prompts do not expose sensitive business logic or proprietary processes in publicly accessible metadata.

---

## 13. References

### 13.1 Normative references

- [MX Core Metadata Standard](mxs-01-core-metadata.cog.md) — conformance level framework, Zone 1/Zone 2 identity fields
- [Cog Unified Specification](../specifications/cog-unified-spec.cog.md) — the cog document format
- [NDR-02: camelCase Naming](../naming-decisions/ndr-02-camelcase-naming.cog.md) — field naming convention
- [NDR-03: Spelling Neutrality](../naming-decisions/ndr-03-spelling-neutrality.cog.md) — spelling-neutral vocabulary
- [MX Extensions Standard](mxs-02-extensions.cog.md) — namespace policy, carrier formats, extension mechanisms
- [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md) — AI governance and agent policies (related to `provenanceOrigin`)
- [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) — content-type-specific fields

### 13.2 Informative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) — Date and time format
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (referenced in quality triad)
- [Schema.org](https://schema.org/) — Structured data vocabulary (referenced in semantic quality)
- [MX Standards Alignment](../specifications/mx-standards-alignment.cog.md) — how MX conventions align with existing web standards

---

## 14. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-draft | 2026-04-02 | Initial draft. Initial draft. |
