---
title: "MX Provenance note"
docname: draft-cranstoun-mx-provenance
date: 2026-05-07
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
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-provenance.md
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

### 5.0 Identity-field boundaries

Six fields touch document identity, each marking a distinct role in the document lifecycle. Treat them as different concepts and do not conflate them:

| Field | Role | Defined in |
|-------|------|-----------|
| `author` | Immutable creator of the document. The person or entity who first wrote it. Set once at creation and never changed. | [MX Core Metadata](draft-core-metadata.md) (Zone 1, top-level) |
| `provenanceAuthor` | Originator of the document's content, recorded in Zone 2 attribution metadata. May differ from `author` when content was contributed by another party. | This note (§5.1) |
| `maintainer` | Person or team responsible for ongoing maintenance of the document. The accountable steward, not necessarily the original creator. | [MX Core Metadata](draft-core-metadata.md) (Zone 2) |
| `maintainedBy` | Person or team who performed the most recent accuracy confirmation. May differ from `maintainer` (`maintainer` is the accountable role; `maintainedBy` records who actually did the last review). | This note (§7.2) |
| `publisher` | Entity that publishes the document and stands behind its claims. Rich struct with name, url, and contact sub-keys. | This note (§5.5) |
| `provenancePublisher` | Flat entity-name string declaring who published. The simpler form for cogs that do not need the rich `publisher` struct. | This note (§5.2) |

When a document declares more than one of these, each carries its own role and the others are not redundant. Validators MUST NOT collapse them. Authors of the canonical reading-order documentation SHOULD describe the role each field plays rather than treating them as interchangeable synonyms.

---

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

### 6.5 `quality`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

The `quality` object groups three sub-keys that together describe document quality across complementary dimensions. Each sub-key MAY be a string (compliance level or summary) or an object with structured assessment data.

| Sub-key | Dimension | Aligns with |
|---------|-----------|-------------|
| `quality.accessibility` | How well content serves users with disabilities | [WCAG 2.1](https://www.w3.org/TR/WCAG21/), Pa11y |
| `quality.semantic` | How well content uses semantic HTML and structured data | [Schema.org](https://schema.org/) JSON-LD, microdata |
| `quality.convergence` | How well content aligns human and machine experiences | MX dual-experience principle |

```yaml
mx:
  quality:
    accessibility: "WCAG 2.1 AA"
    semantic: "Schema.org JSON-LD"
    convergence: high
```

Common values for `quality.convergence` when expressed as a string: `low`, `medium`, `high`.

The grouping into one parent object reflects that the three dimensions are complementary facets of a shared quality theme; readers and validators can probe `quality` once and iterate the sub-keys rather than checking three top-level fields. The detailed accessibility-conformance vocabulary (`accessibilityConformance`, `accessibilityFeature`, `accessibilityHazard`, `accessibilitySummary`) defined in the [MX Document Accessibility note](draft-document-accessibility.md) lives in a separate disclosure layer; `quality.accessibility` is the high-level signal, the document-accessibility fields are the detailed conformance claim.

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

`expires` is defined in the [MX Core Metadata note](draft-core-metadata.md) as the ISO-8601 date after which the document's content is no longer expected to be authoritative. It is a foundation-layer field; this provenance note refers to it but does not redefine it. Time-sensitive content (pricing PDFs, service-level agreements, compliance reports, regulatory text) MAY declare `expires` alongside the maintenance fields here so a single review window covers both maintenance and validity.

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
| `quality` | quality | — | — | MAY |
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

## 13. Minimum-viable provenance block (Informative)

A worked example an author can copy and adapt. The block sits in Zone 2 (the `mx:` object) of any document's frontmatter and extends the MX Core Metadata floor (governed by the [MX Core Metadata note](./draft-core-metadata.md)).

```yaml
mx:
  provenanceAuthor: "Tom Cranstoun"
  provenancePublisher:
    name: "Digital Domain Technologies Ltd"
    brand: "CogNovaMX"
    contact: "info@cognovamx.com"
  provenanceOrigin: "human-led, AI-assisted"
  reviewCycle: "quarterly"
  maintainedDate: 2026-05-07
```

This block declares: who wrote the original (`provenanceAuthor`), who publishes it under what brand (`provenancePublisher`), the human-machine balance of its authorship (`provenanceOrigin`), the review cadence (`reviewCycle`), and when it was last maintained (`maintainedDate`). Five fields, complete enough to verify provenance for most documents.

For higher trust requirements add `accuracyCommitment` and `correctionSla` from §6 (the quality triad); for full traceability add the decision-record references (`adr`, `ndr`, `bdr`) from §8; for migrated documents add `movedFrom` and `movedDate` from §9.

---

## 14. Common authoring mistakes (Informative)

Three shapes a fresh author often produces, each with the fix.

**Mistake 1: confusing identity with stewardship.**

```yaml
# WRONG — provenanceAuthor names the original creator, not the current maintainer
mx:
  provenanceAuthor: "Maxine"   # Maxine is the current maintainer, not the originator
```

`provenanceAuthor` is immutable identity, closely related to the `originator` field in MX Core Metadata §5.3. For ongoing maintenance, use `maintainedBy` (§7) or the `stewardship.steward` sub-key from MX Core Metadata §6.6.

**Mistake 2: claiming `provenanceOrigin: human-only` after AI-assisted authorship.**

```yaml
# WRONG — the document was edited with AI assistance
mx:
  provenanceOrigin: "human-only"
```

Declaring the human-machine balance honestly is part of what makes provenance verifiable. If an AI tool contributed to drafting, editing, summarising, or fact-checking the document, the value is not `human-only`. Use `human-led, AI-assisted` (or one of the other values defined in §5).

**Mistake 3: declaring `expires` without a review plan.**

```yaml
# WRONG — expires says "this document goes stale on a date" but reviewCycle is missing
mx:
  expires: 2026-12-31
```

`expires` (§7) marks an authoritative-content cutoff. A document that declares `expires` SHOULD also declare `reviewCycle` so that an author or steward knows when to re-validate the content before the cutoff. Otherwise the expiry date is arbitrary.

---

*End of MX Provenance note draft.*
