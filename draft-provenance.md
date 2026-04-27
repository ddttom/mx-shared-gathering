---
title: "MX Provenance note"
docname: draft-cranstoun-mx-provenance
date: 2026-04-27
consensus: false
keyword:
  - mx
  - provenance
  - trust
  - attribution
  - verification
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Provenance note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note defines the provenance, trust, and verification metadata vocabulary for the Machine Experience (MX) framework. It specifies the fields that establish who created content, how it was created, how trustworthy it is, who maintains it, and what governance decisions shaped it.

The provenance vocabulary is organised into five groups: **attribution fields** (authorship and origin), **quality and trust fields** (reliability commitments), **maintenance fields** (ongoing stewardship), **decision record references** (governance links), and **migration fields** (content relocation tracking).

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field — heading, property table, definition prose, example — and is binding on this note. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

This note defines three conformance levels for provenance metadata. References to a top-level `author` field describe a string identifying the document's creator, located in the YAML frontmatter outside the `mx:` object — a near-universal convention in markdown-based authoring systems.

| Level | Name | Provenance requirement |
|-------|------|------------------------|
| Level 1 | **MX Core** | Author attribution present (`provenanceAuthor`, or a top-level `author` field) |
| Level 2 | **MX Standard** | Adds `provenanceOrigin`, `maintainedDate`, and `reviewCycle` declared |
| Level 3 | **MX Complete** | Full trust chain including `publisher`, `accuracyCommitment`, `correctionSla`, decision records, and quality triad |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions, conformance requirements, and normative rules in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Scope

### 3.1 In scope

- **Attribution fields** — authorship, publishing entity, content origin, source material, and generation instructions
- **Quality and trust fields** — accuracy commitments, review cycles, compliance levels, and the quality triad
- **Maintenance fields** — freshness tracking, maintainer identity, expiry, and update triggers
- **Decision record references** — links to architecture, naming, and business decision records
- **Migration fields** — content relocation tracking

### 3.2 Out of scope

- **Document identity fields** — `title`, `description`, `author`, `created`, `modified`, `version` are governed by the MX Core Metadata note. This note refers to the top-level `author` but does not redefine it.
- **AI/agent governance** — how agents may interact with content (read, edit, regenerate, attribute) is a separate concern; this note declares only how content was created (`provenanceOrigin`).
- **Cryptographic verification** — fingerprinting, signing, and signature verification are governed by a separate note. The `provenancePedigree`, `proofOfAuthorship`, and `integritySignature` fields are named in core metadata; their cryptographic mechanics are out of scope here.
- **Carrier-format mappings** — how these fields are expressed in HTML, JSDoc, CSS, etc. is governed by the MX Carrier Formats note.

### 3.3 Relationship to existing standards

All field names in this note use camelCase (matching the [Schema.org Style Guide](https://schema.org/docs/styleguide.html)) and avoid regional spelling variants where an established identifier system supplies a canonical spelling. All date fields use [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) format (YYYY-MM-DD). The quality triad references [WCAG 2.1](https://www.w3.org/TR/WCAG21/) for accessibility and [Schema.org](https://schema.org/) JSON-LD for semantic structure.

---

## 4. Provenance in the two-zone model

Provenance fields reside in Zone 2 (the `mx:` object), complementing the top-level `author` field that markdown-based authoring conventionally places in Zone 1.

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

## 5. Attribution fields

### 5.1 `provenanceAuthor`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST (Level 1) |

Person or entity who originally authored the document content.

```yaml
mx:
  provenanceAuthor: "Tom Cranstoun"
```

- At Level 1, a document MUST declare either `provenanceAuthor` or a top-level `author` field. Both MAY be present.
- When both are present, `provenanceAuthor` identifies the original content author, while `author` identifies the document creator. These are often the same person.

---

### 5.2 `provenancePublisher`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |

Entity that published the document. Distinct from `publisher` (§5.5), which is a richer object used in trust-declaration cogs.

```yaml
mx:
  provenancePublisher: "CogNovaMX Ltd"
```

---

### 5.3 `provenanceOrigin`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |
| **Valid values** | human-directed, ai-assisted, automated-fact-check |

How the content was created. Declares the human-machine authorship balance.

- `human-directed` — content authored by a human, possibly with AI tooling assistance.
- `ai-assisted` — content substantially generated by AI, with human review and direction.
- `automated-fact-check` — content generated and verified by automated processes.

```yaml
mx:
  provenanceOrigin: human-directed
```

---

### 5.4 `source`

| Property | Value |
|----------|-------|
| **Type** | string-or-array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Origin source material or input files. Declares the content provenance chain — what materials were used to create this document. Paths MAY be relative or absolute.

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

---

### 5.5 `publisher`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Entity that publishes and stands behind this document. Can be a string (name) or an object with `name`, `url`, and `contact` sub-fields. Distinct from `provenancePublisher` (§5.2), which is a flat provenance field. The `publisher` field supports richer structured data and is used alongside `accuracyCommitment`, `reviewCycle`, and `mxCompliance` to form a complete trust declaration.

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

---

### 5.6 `generate`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Generation instructions. Describes how to regenerate this content if lost or outdated. If a document is lost, an agent with access to the `generate` instructions and source material can recreate it.

```yaml
mx:
  generate:
    prompt: "Regenerate from source data"
    source: "quarterly-report.csv"
```

- The `prompt` sub-field SHOULD contain instructions specific enough to produce a faithful reconstruction.
- The `source` sub-field identifies the input data required for regeneration.

---

## 6. Quality and trust fields

### 6.1 `accuracyCommitment`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Publisher's commitment to content accuracy. A human-readable statement of the accuracy standard the publisher holds themselves to. The value SHOULD be a clear, actionable statement that a reader can use to assess the publisher's reliability. Used alongside `publisher`, `reviewCycle`, and `correctionSla` to form a complete trust declaration.

```yaml
mx:
  accuracyCommitment: "All factual claims verified against source documentation"
```

---

### 6.2 `reviewCycle`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |

How often content is reviewed for accuracy. Common values: `weekly`, `monthly`, `quarterly`, `annually`, `on-change`.

```yaml
mx:
  reviewCycle: quarterly
```

---

### 6.3 `correctionSla`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Service-level agreement for correcting errors. The value SHOULD be a human-readable duration string. Communicates the publisher's responsiveness to reported errors.

```yaml
mx:
  correctionSla: "24 hours"
```

---

### 6.4 `mxCompliance`

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

MX compliance level or detailed compliance metadata. When expressed as a string, the value corresponds to the conformance levels in §2.1 (`level-1`, `level-2`, `level-3`). Used alongside `publisher`, `accuracyCommitment`, and `reviewCycle` to form a complete trust declaration.

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

---

### 6.5 The quality triad: `accessibility`, `semantic`, `convergence`

The quality triad is three optional MAY-level fields that together describe document quality across complementary dimensions. Each field MAY be a string (compliance level or summary) or an object with structured assessment data.

| Field | Dimension | Aligns with |
|-------|-----------|-------------|
| `accessibility` | How well content serves users with disabilities | [WCAG 2.1](https://www.w3.org/TR/WCAG21/), Pa11y |
| `semantic` | How well content uses semantic HTML and structured data | [Schema.org](https://schema.org/) JSON-LD, microdata |
| `convergence` | How well content aligns human and machine experiences | MX dual-experience principle |

```yaml
mx:
  accessibility: "WCAG 2.1 AA"
  semantic: "Schema.org JSON-LD"
  convergence: high
```

Common values for `convergence` when expressed as a string: `low`, `medium`, `high`.

---

## 7. Maintenance fields

### 7.1 `maintainedDate`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |

Date the content was last confirmed accurate by its maintainer. Distinct from a top-level `modified` field. `modified` tracks when the document was last changed; `maintainedDate` tracks when it was last confirmed to still be accurate — even if no changes were needed. Agents SHOULD treat documents with a `maintainedDate` older than the `reviewCycle` as potentially stale.

```yaml
mx:
  maintainedDate: 2026-04-02
```

---

### 7.2 `maintainedBy`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Person or team who last confirmed accuracy. Distinct from a `maintainer` field (the person responsible for ongoing maintenance). `maintainedBy` records who performed the last accuracy confirmation — these may differ.

```yaml
mx:
  maintainedBy: "Tom Cranstoun"
```

---

### 7.3 `expires`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Expiry date for time-sensitive content. Content SHOULD be reviewed or archived after this date. Agents encountering expired content SHOULD flag it for review rather than treating it as authoritative. Distinct from `cacheability` (how long to cache a fetch) — `expires` indicates when the content itself is no longer current.

```yaml
mx:
  expires: 2026-12-31
```

---

### 7.4 `updateTriggers`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Conditions that trigger a content update. Events that should prompt a content review or revision. Values are context-specific strings describing trigger conditions. Tooling MAY use these values to automate review notifications when trigger conditions are detected.

```yaml
mx:
  updateTriggers:
    - specification-change
    - new-field-added
```

---

## 8. Decision record references

The fields `adr`, `ndr`, and `bdr` link a document to the formal decision record that governs an aspect of its content. Decision records provide governance traceability — they explain why certain choices were made.

All three fields share the same shape: a string path to the decision record, or an object with `path`, `date`, and `status` sub-fields.

| Field | Decision type | Typical content |
|-------|---------------|-----------------|
| `adr` | Architecture Decision Record | Architectural choices: storage strategy, transport protocol, deployment topology |
| `ndr` | Naming Decision Record | Naming conventions: identifier style, file naming, vocabulary policy |
| `bdr` | Business Decision Record | Business decisions: pricing, market positioning, partnership terms |

| Property | Value |
|----------|-------|
| **Type** | string-or-object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

```yaml
mx:
  adr: "decisions/adr-007-storage-strategy.md"
  ndr: "decisions/ndr-002-camelcase-naming.md"
  bdr: "decisions/bdr-012-launch-pricing.md"
```

```yaml
mx:
  adr:
    path: "decisions/adr-007-storage-strategy.md"
    date: 2026-03-15
    status: accepted
```

---

## 9. Migration fields

### 9.1 `movedFrom`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Previous file path before content was relocated. Used for redirect generation and link maintenance. The value SHOULD be the full relative path from the repository root. Tooling MAY use this field to generate redirects or update internal links automatically. SHOULD be paired with `movedDate`.

```yaml
mx:
  movedFrom: "datalake/knowledge/reference/field-definitions.md"
```

---

### 9.2 `movedDate`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

Date content was relocated. SHOULD be paired with `movedFrom` to provide a complete migration record.

```yaml
mx:
  movedDate: 2026-03-15
```

---

## 10. Conformance summary

| Field | Group | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|-------|:-:|:-:|:-:|
| `provenanceAuthor` | attribution | MUST | — | — |
| `provenancePublisher` | attribution | — | SHOULD | — |
| `provenanceOrigin` | attribution | — | SHOULD | — |
| `source` | attribution | — | — | MAY |
| `publisher` | attribution | — | — | MAY |
| `generate` | attribution | — | — | MAY |
| `reviewCycle` | quality | — | SHOULD | — |
| `accuracyCommitment` | quality | — | — | MAY |
| `correctionSla` | quality | — | — | MAY |
| `mxCompliance` | quality | — | — | MAY |
| `accessibility` | quality | — | — | MAY |
| `semantic` | quality | — | — | MAY |
| `convergence` | quality | — | — | MAY |
| `maintainedDate` | maintenance | — | SHOULD | — |
| `maintainedBy` | maintenance | — | — | MAY |
| `expires` | maintenance | — | — | MAY |
| `updateTriggers` | maintenance | — | — | MAY |
| `adr` | decisions | — | — | MAY |
| `ndr` | decisions | — | — | MAY |
| `bdr` | decisions | — | — | MAY |
| `movedFrom` | migration | — | — | MAY |
| `movedDate` | migration | — | — | MAY |

---

## 11. Security and privacy considerations

- The `provenanceAuthor` and `provenancePublisher` fields contain personally identifiable information. Implementations that publish metadata publicly SHOULD consider whether these values are appropriate for public exposure.
- The `publisher.contact` sub-field MAY contain email addresses or other contact information. Implementations SHOULD consider whether such values are appropriate for public exposure before publishing this metadata externally.
- The `provenanceOrigin` field declares the human-machine authorship balance. Publishers SHOULD declare this field honestly to maintain trust with consumers and agents.
- Decision record references (`adr`, `ndr`, `bdr`) MAY link to internal governance documents. Implementations SHOULD verify that referenced documents are accessible to the intended audience before publishing these links externally.
- The `generate` field contains instructions for content regeneration. Implementations SHOULD ensure that generation prompts do not expose sensitive business logic or proprietary processes in publicly accessible metadata.

---

## 12. References

### 12.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) — Date and time format

### 12.2 Informative references

- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (referenced in quality triad and the conformance level model)
- [Schema.org](https://schema.org/) — Structured data vocabulary (referenced in semantic quality)
- [Schema.org Style Guide](https://schema.org/docs/styleguide.html) — Vocabulary naming conventions
- [Dublin Core DCMI Namespace](https://www.dublincore.org/specifications/dublin-core/dcmi-namespace/) — Provenance and stewardship metadata model

---

*End of MX Provenance note draft.*
