---
title: "MX Field Definition Pattern note"
docname: draft-cranstoun-mx-field-pattern
date: 2026-04-27
consensus: false
keyword:
  - mx
  - field-pattern
  - authoring
  - template
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Field Definition Pattern note

**Version:** 1.0-draft
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review — **primary note of the MX draft set**
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This is the **primary note** of the MX draft set. Every other note in the set proposes one or more frontmatter fields, and every one of those fields is defined under the pattern this note specifies. Readers new to the suite should read this note first; sister notes assume its rules and do not restate them.

Every MX draft note proposes one or more frontmatter fields. Today each note repeats the same structural choices in slightly different prose: a heading, a property table, a definition, an example, and (sometimes) extra rules. The minor variations make automated extraction harder than it needs to be, and they make a reviewer ask "is this field defined the same way as the last one?" every time.

This note proposes a single, vetted **pattern** for defining a frontmatter field inside any MX draft. Once the community ratifies the pattern, future drafts add new fields by following the template — author once, machine-read everywhere.

**Why field discipline matters.** Machine consumers — agents, validators, registries, signers, graph builders — cannot infer intent from prose. They need fields that are explicit, tightly typed, and constrained to a small set of legal values. The MX vocabulary is therefore expected to **grow, not shrink**: new fields are how machines come to understand what a document means. Field growth without authoring discipline produces drift, and drift compounds: every loose definition multiplies the work of every downstream consumer that has to reconcile it. The pattern in this note exists to absorb growth without producing drift, and to keep every new field tightly constrained from the moment it is proposed.

The pattern itself is the proposal. **No new frontmatter fields are introduced in this note.** This note specifies *how* a field should be defined; *which* fields exist is the work of the sister notes that propose them.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

This note's pattern is **normative for the entire MX draft set**. Every MX note that proposes one or more frontmatter fields MUST define each field using the structure in §4, the property-table rows in §5 (for standard fields), or the inventory-table form in §4.4 (for pass-through fields). Sister notes MUST NOT introduce alternative shapes, additional property-table rows, or supplementary section types beyond those defined here.

Existing field definitions that predate ratification of this pattern remain valid until the next revision of the host note; on that revision, all new and amended definitions MUST conform, and the host note SHOULD migrate any older definitions that are touched. A note that defines no fields makes no claim against this pattern and is unaffected by it.

### 2.1 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the structural rules below are working drafts — stable enough to author against, expected to evolve through review.

---

## 3. Scope

### 3.1 In scope

- The structural template for a **standard field** definition inside an MX draft.
- The structural template for **pass-through fields** — fields whose value semantics are owned by an external vocabulary (Dublin Core, Schema.org, BCP 47, SPDX, FOAF, ...).
- The required and optional rows of the property table.
- Naming and ordering conventions for field-definition headings and sub-sections.
- One worked example showing the pattern applied.

### 3.2 Out of scope

- **Specific field semantics.** What `title`, `provenanceAuthor`, `x-mx-thresholds`, or any other field *means* belongs in the relevant note. This note does not define any field.
- **Profile catalogues.** Which fields apply to which document type (`core`, `cog`, `report`, ...) is governed by the notes that own those profiles.
- **Schema languages.** JSON Schema, YAML schema, Schema.org, and similar machine-readable shape languages are unrelated; this note specifies authored-prose shape only.
- **Carrier-format mappings.** How a field is expressed in HTML, JSDoc, CSS, etc. is governed by the MX Carrier Formats note.
- **Conformance-level definitions.** What "Level 1 / 2 / 3" mean within a particular note is the host note's choice; this pattern only requires that one of those levels be cited.

### 3.3 Relationship to existing standards

This note follows the [IETF RFC format](https://www.rfc-editor.org/rfc/rfc7322) for standards-document authoring and uses the [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) keyword vocabulary. The pattern itself is a stylistic convention layered on top of those, not a redefinition of either.

---

## 4. The field-definition pattern

A field definition is a contiguous unit inside a host note. It has four required pieces, in order, plus optional supplementary material.

### 4.1 Required pieces

1. **Heading.** A section heading at the level appropriate to the host note's structure (typically `### N.M`), with the field name in backticks. Example: `### 5.3 ` `` `author` ``. Headings MUST contain only the bare field name; no prose, no qualifiers, no parenthetical types.
2. **Property table.** A two-column markdown table with the rows specified in §5, in the order given.
3. **Definition prose.** One or more paragraphs of plain prose that name what the field is, what it carries, and any constraint not expressible in the property table. Definition prose MUST come immediately after the property table. Definition prose MUST NOT restate values already given in the table.
4. **Example.** A fenced YAML code block showing the field in use, in the carrier the host note targets (typically `mx:` block or top-level frontmatter). Examples MUST be syntactically valid YAML and MUST illustrate the field's actual shape. A definition with multiple legal shapes (e.g. `string-or-object`) MAY include multiple example blocks.

### 4.2 Optional supplementary material

After the required pieces, a definition MAY include any of:

- **Bulleted constraints** — additional rules expressible as short bullets (e.g. "the value MUST be quoted in YAML to prevent numeric coercion").
- **Cross-reference** — a one-line pointer to a related field elsewhere in the same note (e.g. "Distinct from `maintainer` (§6.6)"). Cross-references to *other notes* MUST be by note name only, never by file path or URL, per the standalone-ness rule of the draft set.
- **Sub-table** — a further markdown table when the field's value is an object with named sub-keys (e.g. `cogHeader` with `version`, `spec`, `runtime`, `runtimeDoc`).

A definition MUST NOT include a "Normative notes" preamble whose body merely paraphrases the property table or the definition prose. If a putative note adds no new constraint, it is removed.

### 4.3 Group rules

When two or more fields share the same shape and conformance, a note MAY collapse them into a single section that lists the fields together, presents one shared property table, and explains the differences in a small differentiation table. The host note SHOULD use this form whenever the result is shorter than separate sections without loss of normative content.

A group section follows the same Required-pieces order as a single field, with the heading naming the group (e.g. "Decision record references: `adr`, `ndr`, `bdr`") instead of a single field.

### 4.4 Pass-through field pattern

A **pass-through field** is a YAML key whose value semantics belong to an established external vocabulary (Dublin Core, Schema.org, BCP 47, SPDX, FOAF, and similar). MX names the key so that authors do not have to switch syntax mid-frontmatter; the external standard owns the meaning.

Pass-through fields use a different shape from §4.1, because the host note is not the owner of the value. Instead of one full section per field, a host note groups pass-through fields into a single **inventory table** with these columns, in this order:

| Column | Required | Value |
|--------|:--------:|-------|
| **Field** | yes | The YAML key MX provides, in backticks. |
| **Aligns with** | yes | The external standard's canonical name plus the specific term, e.g. `Dublin Core dc:date, Schema.org Date`. The standard MUST be cited precisely (URL plus the specific field/property name) at least once in the note. |
| **Value** | yes | One sentence summarising what the value carries. |
| **Conformance** | yes | Always `MAY` for pass-through fields — they are optional surface for an external vocabulary. |

A pass-through inventory table appears in a dedicated pass-through section of the host note. The section MUST also state, in prose:

- That the listed fields are pass-through (definition above).
- That MX does not redefine or contradict the cited external standard.
- The rule for adding a new pass-through field: an established external vocabulary cleanly covers the value, the alignment is declared via canon's `alignsWith:`, and MX names the YAML key without governing its semantics. If the external standard does not cleanly cover the value, the field is **not** a pass-through and MUST be defined under the standard pattern in §4.1.

Pass-through fields MUST NOT use the property table in §5; the inventory table replaces it. A field definition that mixes the two forms is a conformance failure.

The MX Core Metadata note's pass-through inventory is the reference precedent. Sister notes that introduce additional pass-through fields (e.g. linguistic, rights, dataset metadata) MUST follow this same inventory-table form.

---

## 5. The property table

The property table is the standard-field shape. It immediately follows the heading and uses two columns — `Property` and `Value` — containing the rows below in this exact order. Rows that do not apply MUST be omitted entirely; rows that do apply MUST appear in the order given. Pass-through fields use the inventory-table form in §4.4 and do not use this property table.

| Row | When | Value |
|-----|------|-------|
| **Type** | always | One of: `string`, `boolean`, `number`, `array`, `object`, `array of <type>`, `string-or-object`, `string-or-array`, `string-or-boolean`. New compound types MUST be hyphenated and in lowercase. |
| **Zone** | always for fields that live in YAML frontmatter | `1 (top-level)` or `2 (mx:)`. Fields that have no zone (e.g. parameters of a sub-key) omit this row. |
| **Profile** | when the field is profile-restricted | A comma-separated list of profile names (`core`, `cog`, `report`, `migration`, `x-mx-public`, ...). Omitted when the field is universal. |
| **Conformance** | always | An [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) keyword (`MUST`, `SHOULD`, `MAY`) plus the host note's level in parentheses. Example: `MUST (Level 1)`. Multiple conformance entries MAY appear when the level depends on context, separated by semicolons. |
| **Valid values** | when the value is enumerated | A comma-separated list of permitted values, in lowercase, exactly as they appear in YAML. |
| **Default** | when omitting the field has a defined behaviour | The default value, or `*(none)*` when there is no default. Use `*(none — explicit declaration required)*` for MUST-level fields. |

The table MUST NOT contain rows beyond these. New rows MAY be proposed only by amending this note, not by individual field definitions.

---

## 6. Worked example

The following is a complete, conforming definition of a hypothetical field `riskLevel`. It is not a real proposal; it exists only to illustrate the pattern.

> ### 7.4 `riskLevel`
>
> | Property | Value |
> |----------|-------|
> | **Type** | string |
> | **Zone** | 2 (mx:) |
> | **Conformance** | MAY (Level 3) |
> | **Valid values** | low, medium, high, critical |
> | **Default** | *(none)* |
>
> Self-declared risk classification of the document's subject matter. Helps agents and maintainers prioritise review attention when a queue is large.
>
> ```yaml
> mx:
>   riskLevel: medium
> ```
>
> - `critical` SHOULD be reserved for content whose factual error could cause material harm; lighter classifications cover everyday content.
> - Distinct from `confidential`, which controls publication, not review urgency.

The example shows: a heading containing only the bare field name; the four required property rows in order; the two optional rows (`Valid values`, `Default`) where they apply; one paragraph of definition prose that does not restate the table; one YAML example block; and two bulleted constraints — one introducing a new rule, one cross-referencing a sibling field. No "Normative notes" preamble is used.

---

## 7. Authoring rules

These rules are normative for any field defined under this pattern.

### 7.1 Naming

Field names use camelCase in YAML contexts (matching the [Schema.org Style Guide](https://schema.org/docs/styleguide.html)). Vendor-extension fields use kebab-case after their prefix (`x-mx-deploy-target`, not `x-mx-deployTarget`). Field names MUST NOT use snake_case or PascalCase in YAML contexts.

### 7.2 Conformance keywords

Each definition cites exactly one of `MUST`, `SHOULD`, or `MAY` as its primary conformance keyword. When the level is conditional ("MUST when X applies, MAY otherwise"), the conditions MUST be explicit in the Conformance row, semicolon-separated.

### 7.3 Length

A typical single-field definition is 8–25 lines. Definitions that exceed roughly 40 lines SHOULD be reviewed for unnecessary prose, restated rules, or material that belongs in a sibling note.

### 7.4 What not to include

A field definition MUST NOT contain:

- Marketing language or assertions of importance ("this critical field", "the most useful").
- Implementation detail (which library reads it, which database column it maps to).
- Forward-looking statements ("we plan to extend this to ...").
- Examples that duplicate a previous example without adding new shape.
- Cross-references to draft files by path or URL.

### 7.5 Worked-example status

The example in §6 is illustrative, not normative. Implementations MUST NOT treat `riskLevel` as a registered field on the basis of this note alone.

---

## 8. Security and privacy considerations

This note specifies authoring style only and introduces no new fields, no carriers, and no signing surface. It carries no direct security or privacy implications.

A second-order consideration: a uniform field-definition pattern makes automated extraction tractable, which means tooling can mechanically enumerate every field a draft proposes. Authors SHOULD assume that anything written inside a property table or YAML example will be parsed by tools that treat the draft as canonical, and SHOULD NOT include sensitive sample values in examples.

---

## 9. References

### 9.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels

### 9.2 Informative references

- [RFC 7322](https://www.rfc-editor.org/rfc/rfc7322) — RFC Style Guide (style precedent for standards-document authoring)
- [Schema.org Style Guide](https://schema.org/docs/styleguide.html) — Vocabulary naming conventions

---

## 10. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-draft | 2026-04-27 | Initial draft. Proposes a single uniform pattern for defining a frontmatter field inside any MX draft note: heading, property table, definition prose, example, optional supplementary material. Defines the property table's required and optional rows, naming and ordering conventions, and authoring rules. Documents the pass-through field pattern as a recognised second variant (§4.4) for fields whose semantics are owned by external vocabularies. Makes the pattern normative for the entire MX draft set: every sister note proposing fields MUST adhere. Introduces no new frontmatter fields itself, and motivates the discipline by noting that machines need *more* fields, tightly constrained, to understand document intent. |

---

*End of MX Field Definition Pattern note draft.*
