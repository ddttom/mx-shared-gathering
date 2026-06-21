---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Temporal Stance note"
docname: draft-cranstoun-mx-temporal-stance
date: 2026-05-07
consensus: false
keyword:
  - mx
  - temporal
  - anchors
  - regulatory
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-temporal-stance.md
---

# MX Temporal Stance note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 7 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

Many MX-aware documents have a **temporal stance**: a relationship between the prose they carry and the dates they describe. A regulatory analysis published before a law's application date reads correctly today and reads stale eighteen months later. A pricing document declares a price that holds until a specific date. An SLA quotes uptime targets relative to a contract anchor. A blog post marking the EU AI Act's application date is anchored to a date the legislator could shift.

Without a shared vocabulary for declaring temporal stance, every author solves the problem differently — some bake dates into prose, some compute "days until X" from JavaScript at render time, some rely on the reader to know which dates are real and which are anchors. The result is documents that drift silently from their intended truth-relative-to-time as the dates they describe move.

This note defines four fields — `temporalStance`, `temporalAnchors`, `temporalComputedFields`, `temporalProseGuidance` — that let a document declare its relationship to time machine-readably. A consumer (a render-time agent, a static-site builder, a verifier, a downstream summariser) can then compute date-derived values from anchors at fetch time, surface stale prose to the steward when an anchor moves, and explain the document's temporal contract without parsing the prose.

The fields live in Zone 2 (the `mx:` object) of any MX-aware document's frontmatter and apply equally to regulatory analyses, contracts, SLAs, compliance reports, pricing pages, and tech-trend pieces.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A document declaring a temporal stance SHALL satisfy:

- **Level 1 (Stance declared):** The document declares `temporalStance` with a value drawn from the controlled vocabulary in §4.1.
- **Level 2 (Anchored):** Level 1 plus, when `temporalStance: anchored`, every anchor referenced in the prose is declared in `temporalAnchors`.
- **Level 3 (Verifiable):** Level 2 plus `temporalComputedFields` declares every value in the prose computed from an anchor at render time, and `temporalProseGuidance` describes what a steward should change in the prose if any anchor moves.

A document not concerned with time MAY omit all four fields. A document declaring `temporalStance` other than `frozen` SHOULD declare at least Level 2.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Implementations of this draft SHOULD note the draft version number they implemented against.

---

## 3. Scope

### 3.1 In scope

- The four temporal-stance fields and their semantics.
- The render-time and fetch-time obligations on consumers that resolve anchors.
- The prose-guidance fields stewards use to keep documents honest as anchors move.

### 3.2 Out of scope

- **Calendar systems and timezones.** Anchors are ISO 8601 dates ([ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)); time-of-day, timezone, and non-Gregorian calendars are beyond this note.
- **Versioning.** When an anchor moves materially, the document SHOULD be re-versioned per the MX Core Metadata note's `version` field; that note governs the lifecycle.
- **Display formatting.** How a render-time agent presents a computed value (e.g. "in 87 days" vs. "on 2 August 2026") is a presentation concern, not specified here.

### 3.3 Relationship to existing standards

This note relies on:

- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) for date format.
- [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986) for URI form when anchors reference external sources.

It builds on:

- The **MX Core Metadata note** for the Zone 2 (`mx:` object) frontmatter model and the `created` / `modified` / `version` lifecycle fields.
- The **MX Provenance note** for `expires` and `reviewCycle`, which declare lifecycle endpoints distinct from temporal stance.

This note adds vocabulary; it does not duplicate or override either dependency.

---

## 4. Field definitions

### 4.1 `temporalStance`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 1) for any document referring to dated events; MAY otherwise |
| **Valid values** | frozen, living, anchored |

**Definition:** Declares the document's relationship between its prose and the dates the prose describes. Three values:

- **`frozen`** — the prose reflects state at publication. Dates in the prose are historical record; they do not move when the underlying world moves. A retrospective, a historical study, a release-notes archive.
- **`living`** — the prose reflects state at fetch time. Dates and date-derived values are recomputed against the consumer's clock when the document is read. A status page, a "current state" report, a now-page.
- **`anchored`** — the prose contains values computed from declared anchors. When an anchor moves, the document's intended meaning moves with it; stewards re-render or re-author per `temporalProseGuidance`. Regulatory analyses, contract terms, SLAs, compliance reports.

```yaml
mx:
  temporalStance: anchored
```

- A document declaring `temporalStance: anchored` SHOULD declare `temporalAnchors`.
- A document declaring `temporalStance: living` MAY omit anchors; a render-time agent uses the consumer's clock as the implicit anchor.
- A document declaring `temporalStance: frozen` MUST NOT declare anchors that move; if a date the prose mentions is genuinely an anchor (likely to move), the stance is not `frozen`.

### 4.2 `temporalAnchors`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) when `temporalStance: anchored`; MAY otherwise |

**Definition:** Named date anchors the document's prose depends on. Each entry pairs a stable identifier with the anchor's current value and a short human description of what the anchor is. The identifier is the key by which `temporalComputedFields` and `temporalProseGuidance` reference the anchor.

**Sub-keys** (per anchor entry):

| Sub-key | Type | Definition |
|---------|------|-----------|
| `date` | string (ISO 8601) | Current value of the anchor. MUST be present. |
| `description` | string | One-line explanation of what the anchor is, including the source if external (a law, a contract clause, a release calendar). |
| `source` | string (URL) | Optional canonical URL where the anchor's authoritative value is published. |

```yaml
mx:
  temporalAnchors:
    eu-ai-act-application:
      date: "2026-08-02"
      description: "EU AI Act provisions for general-purpose AI become applicable."
      source: "https://eur-lex.europa.eu/eli/reg/2024/1689"
    contract-renewal:
      date: "2027-04-30"
      description: "Master service agreement renewal date."
```

- Anchor identifiers SHOULD be lowercase kebab-case and stable across versions of the document.
- An anchor whose `date` value changes is a substantive editorial event; the document's `modified` date MUST be bumped, and the steward SHOULD review against `temporalProseGuidance` (§4.4).

### 4.3 `temporalComputedFields`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

**Definition:** Fields whose values are computed from anchors at render time. Each entry pairs a field name with an expression naming the anchor and the computation. Render-time agents MUST NOT cache the computed value beyond a single render; the value is by definition relative to the moment of fetch.

**Sub-keys** (per computed-field entry):

| Sub-key | Type | Definition |
|---------|------|-----------|
| `from` | string | Anchor identifier (must match a key in `temporalAnchors` or the literal `now`). |
| `compute` | string | Computation: one of `daysUntil`, `daysSince`, `monthsUntil`, `monthsSince`, `quartersUntil`, `quartersSince`, `dateValue` (returns the anchor date verbatim). |
| `format` | string | Optional presentation hint (e.g. `integer`, `iso-date`, `human-readable`). Implementations MAY ignore. |

```yaml
mx:
  temporalComputedFields:
    daysUntilApplication:
      from: eu-ai-act-application
      compute: daysUntil
      format: integer
    contractMonthsRemaining:
      from: contract-renewal
      compute: monthsUntil
      format: integer
```

- Render-time agents MUST treat unknown anchor identifiers as a render failure rather than substituting a default.
- A field name in `temporalComputedFields` SHOULD be referenced in the document's prose using a templating convention agreed with the publishing toolchain (this note does not mandate a template syntax).

### 4.4 `temporalProseGuidance`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 3) for documents declaring `temporalStance: anchored`; MAY otherwise |

**Definition:** Author-supplied notes telling future agents (human or machine) how to update the prose when an anchor moves. Each entry pairs an anchor identifier with prose-update instructions. The field is the bridge between the structural temporal-stance vocabulary and the unstructured prose that depends on it.

**Sub-keys** (per anchor entry):

| Sub-key | Type | Definition |
|---------|------|-----------|
| `onMove` | string | What a steward should review or rewrite in the prose if this anchor's date changes. |
| `affectedSections` | array | Optional list of section anchors (URL fragments) that depend on the anchor. |
| `softCutoff` | boolean | When `true`, a small move in the anchor (within a configurable tolerance) requires no prose update; only a material move does. Tools SHOULD surface what "material" means with a one-line description in `onMove`. |

```yaml
mx:
  temporalProseGuidance:
    eu-ai-act-application:
      onMove: "Re-read §3 'What changes on day one'. If the anchor moves more than 30 days, re-write the run-up timetable."
      affectedSections: ["#what-changes-on-day-one", "#run-up-timetable"]
      softCutoff: true
    contract-renewal:
      onMove: "Update the renewal callout in §1; reissue if the new date is more than 90 days away."
      softCutoff: false
```

- A document declaring `temporalProseGuidance` SHOULD list every anchor whose movement could leave the prose factually wrong.
- Anchors whose movement is purely cosmetic (the prose just quotes the date) MAY be omitted from the guidance.

---

## 5. Implementation guidance (Informative)

*This section is informative; it is not a normative part of the note.*

### 5.1 Static-site builders

A static-site builder reading an `anchored` document SHOULD resolve `temporalComputedFields` against the build clock and either:

- Inline the computed values into the rendered HTML at build time (and re-build when an anchor moves), or
- Emit the values as `data-mx-temporal-*` attributes on the relevant elements so a client-side renderer can recompute against the reader's clock without re-fetching the document.

The choice is a presentation decision, not a conformance one. Both approaches are conformant.

### 5.2 Runtime agents

A render-time agent reading a `living` document recomputes against the agent's clock at fetch time. A render-time agent reading an `anchored` document recomputes against the anchor `date` values declared in the document, not the agent's clock — the anchor is authoritative even if the agent's clock is later.

### 5.3 Stewardship loop

A steward editing a document with declared anchors SHOULD:

1. Update the anchor's `date` in `temporalAnchors`.
2. Bump `modified` in Zone 1.
3. Walk `temporalProseGuidance` for the moved anchor and either rewrite the affected sections or, if the move is within `softCutoff`, leave them.
4. If any computed-field value materially changes, bump `version` per the MX Core Metadata note's `version` field and re-sign per the MX Contract Fingerprinting and Signing note if the document is signed.

---

## 6. Security and privacy considerations

- A document whose anchors point to external sources discloses what dates the publisher considers load-bearing. Authors handling commercially sensitive timelines SHOULD consider whether the anchor identifiers and source URLs are appropriate for public exposure.
- A render-time agent that fetches an anchor's `source` URL to verify the current value introduces a network call and a potential privacy leak (the publisher of the source learns who is verifying). Agents SHOULD honour the consumer's privacy expectations and SHOULD NOT issue verification fetches without consent.
- A `living` document's computed values move every fetch. Verifiers signing such a document MUST exclude `temporalComputedFields` values from the contract surface (the anchor itself MAY be in the contract; the values derived from it at render time are by definition outside the signature).

---

## 7. IANA considerations

This note proposes no new IANA registrations.

---

## 8. References

### 8.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) — Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) — Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) — Date and time format.
- [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986) — Uniform Resource Identifier generic syntax.

### 8.2 Informative

- **MX Core Metadata note** — Zone 2 frontmatter model and the `version` lifecycle this note builds on.
- **MX Provenance note** — `expires` and `reviewCycle` fields, distinct from temporal stance.
- **MX Contract Fingerprinting and Signing note** — signing model that excludes `temporalComputedFields` values from the contract surface.

---

## 9. Author quickstart (Informative)

A minimum-viable temporal-stance block for a regulatory blog post anchored to the EU AI Act application date.

```yaml
---
title: "What changes on day one of the EU AI Act"
description: "A run-up reading of the EU AI Act, anchored to the application date for general-purpose AI."
originator: "Tom Cranstoun"
created: 2026-05-07
modified: 2026-05-07
version: "1.0"
type: info-doc
mx:
  status: published
  canonicalUri: https://example.org/blog/eu-ai-act-day-one
  summary: "What the EU AI Act changes on day one for general-purpose AI providers and users."
  conformsTo: [https://mx.allabout.network/cog.html]
  trainingDataPolicy: "Permitted with attribution."
  temporalStance: anchored
  temporalAnchors:
    eu-ai-act-application:
      date: "2026-08-02"
      description: "EU AI Act provisions for general-purpose AI become applicable."
      source: "https://eur-lex.europa.eu/eli/reg/2024/1689"
  temporalComputedFields:
    daysUntilApplication:
      from: eu-ai-act-application
      compute: daysUntil
      format: integer
  temporalProseGuidance:
    eu-ai-act-application:
      onMove: "Re-read the run-up timetable in §3. If the anchor moves more than 30 days, rewrite it."
      affectedSections: ["#run-up-timetable"]
      softCutoff: true
---
```

---

## 10. Common authoring mistakes (Informative)

Three shapes a fresh author often produces, each with the fix.

**Mistake 1: declaring `temporalStance: frozen` on a document with anchors.**

```yaml
# WRONG — frozen documents do not have anchors that move
mx:
  temporalStance: frozen
  temporalAnchors:
    contract-renewal: { date: "2027-04-30", description: "..." }
```

If the document depends on a date that could move, the stance is `anchored`, not `frozen` (§4.1). Use `frozen` only for retrospectives where every date is historical record.

**Mistake 2: missing `temporalProseGuidance` on an `anchored` document with prose dependencies.**

```yaml
# WRONG — anchors declared, but no guidance on what to update if they move
mx:
  temporalStance: anchored
  temporalAnchors:
    eu-ai-act-application: { date: "2026-08-02", description: "..." }
```

Without `temporalProseGuidance`, a future steward facing a moved anchor has no instructions and the document drifts silently. Add at minimum an `onMove` line per anchor.

**Mistake 3: signing `temporalComputedFields` values into the contract.**

```yaml
# WRONG — daysUntilApplication moves every render; a signature over it is invalidated immediately
contractFields:
  - title
  - schema
  - daysUntilApplication
```

`temporalComputedFields` values are by definition outside the signature (§6). The anchor itself (`temporalAnchors.eu-ai-act-application`) MAY be in the contract; the rendered value is not.

---

*End of MX Temporal Stance note draft.*
