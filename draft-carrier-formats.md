---
title: "MX Carrier Formats note"
docname: draft-cranstoun-mx-carrier-formats
date: 2026-04-27
consensus: false
keyword:
  - mx
  - carriers
  - code
  - provenance
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Carrier Formats note

**Version:** 1.1-draft
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies the MX metadata vocabulary for source code as a carrier format. It is deliberately narrow. Code files are documents, and as documents they carry the same general identity vocabulary that any MX-aware document does (title, author, dates, description, licence). This note adds **only the fields that are specific to code as a carrier and are not already covered by that universal vocabulary** — a minimum viable set of code-specific provenance fields.

What the code does — function signatures, API surfaces, test metadata, type systems, invariants, behaviour — is explicitly out of scope. Each language already has its own convention for this (JSDoc, Python docstrings, Doxygen, rustdoc, godoc). MX does not duplicate them.

Database and media carriers remain out of scope. The MX framework reuses existing standards:

- **Databases** — defer to [DCAT v3](https://www.w3.org/TR/vocab-dcat-3/) (datasets, distributions, catalogues), [CSVW](https://www.w3.org/TR/tabular-metadata/) (tabular schemas, columns, keys), and [Dublin Core](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) (dataset identity).
- **Media** — defer to [Schema.org](https://schema.org/) (`ImageObject`, `VideoObject`, `AudioObject`, `CreativeWork`, `license`) and the native embedded-metadata standards (EXIF, IPTC, XMP, ID3).
- **Code behaviour** — defer to the language's own documentation convention.

The tight scope is deliberate. Every candidate field must pass the triage rule: *is this specific to code as a carrier, and not already covered by the universal identity vocabulary or by an existing language convention?* If the answer is no on either count, the field does not belong in this note.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

### 2.1 Conformance levels

This note defines three conformance levels (modelled on the [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level A/AA/AAA pattern):

| Level | Name | Requirement for code carriers |
|-------|------|-------------------------------|
| Level 1 | **MX Core** | Universal identity fields present (`title`, `author`, `created`, `description`) |
| Level 2 | **MX Standard** | Adds Level 1 plus the operational fields applicable to code (`status`, `contentType`, `license`) |
| Level 3 | **MX Complete** | Adds Level 2 plus the code-specific provenance fields in §3 where they apply |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Selective adoption

An implementation is **code-conformant at Level N** if, for every code artefact it emits, it satisfies the Level-N identity fields and, where applicable, declares the code-specific provenance fields defined in §3. Implementations MAY claim code conformance without adopting every field; `sourceRepo` and `derivedFromCommit` are both OPTIONAL.

### 2.3 Baseline identity contract

Every code-carrier document carries the universal Level 1 identity contract:

- `title` — REQUIRED
- `author` — REQUIRED
- `created` — REQUIRED (ISO 8601 date format YYYY-MM-DD)
- `description` — RECOMMENDED

A code-carrier document MUST satisfy these identity fields regardless of whether it declares any of the code-specific fields in §3.

### 2.4 Database and media conformance

Implementations seeking metadata conformance for datasets, tables, columns, images, video, or audio look to the external standards cited in §1 Abstract. MX makes no conformance claim for those carriers.

---

## 3. The code vocabulary

Applies to source code files. The vocabulary is two fields, each carrying provenance that is specific to code as a document and not already covered by the universal identity vocabulary.

### 3.1 Fields

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| `sourceRepo` | string | OPTIONAL | URL or path of the source-control repository that holds this code file's authoritative history. Lets a reader locate the canonical copy and its change history. Aligns with [Schema.org `codeRepository`](https://schema.org/codeRepository). |
| `derivedFromCommit` | string | OPTIONAL | Version-control commit identifier (typically a git SHA) that this artefact was derived from. Gives precise pedigree when the artefact is a build output, a transpiled copy, or a distributed snapshot of a specific repository state. Pairs with a generic `derivedFrom` provenance pointer to add commit-level specificity. |

Every other concern that a wider draft might cover — function annotations, API surface, test metadata, class invariants, dependency declarations, runtime config, build conventions — is either out of scope entirely (deferred to the language convention) or handled generically by universal identity fields.

### 3.2 Carrier syntax

Code carriers use the native metadata mechanism of their file format. Field names are identical across all carriers; only the host syntax changes.

| Carrier | Native mechanism | Example |
|---------|------------------|---------|
| JavaScript / TypeScript | JSDoc comments | `@mx:sourceRepo https://github.com/example/repo` |
| CSS | CSS block comments | `/* @mx:sourceRepo https://github.com/example/repo */` |
| Shell | Shell comment block | `# sourceRepo: https://github.com/example/repo` (after shebang) |
| SQL | SQL comment block | `-- @mx sourceRepo: https://github.com/example/repo` |
| Any | YAML frontmatter in `.cog.*` files | `sourceRepo: https://github.com/example/repo` |

A JavaScript file declaring `@mx:sourceRepo` and a `.cog.md` declaring `sourceRepo:` carry the same semantic value.

### 3.3 Profiles

This note declares a single profile: `code`. An implementation handling code artefacts applies the `code` profile's fields in addition to the universal identity fields. There are no per-language sub-profiles — the vocabulary is language-agnostic.

---

## 4. What this note does NOT cover

This note is scoped tightly. The following concerns are explicitly out of scope:

- **Function-level annotations** (`pure`, `idempotent`, `complexity`, `throws`, `returns`, `param`) — defer to JSDoc, Python docstring convention, Doxygen, rustdoc, godoc.
- **API surface specification** (HTTP method, path, auth, rate limits, cache semantics) — defer to [OpenAPI](https://www.openapis.org/).
- **Test metadata** (test type, coverage targets, fixtures) — defer to test framework conventions.
- **Class-level annotations** (design pattern, thread-safety, invariants) — defer to language convention and code reviews.
- **Dependency vocabulary** (purpose, criticality, upgrade policy, alternatives considered) — defer to package manifests (`package.json`, `pyproject.toml`, `Cargo.toml`) and Architecture Decision Records.
- **Runtime configuration** (interpreter, language version, framework versions) — defer to native config (`.node-version`, `.tool-versions`, shebang lines, package manifests).
- **Coding conventions** (style, testing conventions, documentation conventions) — defer to linter configs and community style guides.
- **Inline code annotations** (`mx:begin`, `mx:end`, `mx:sensitive`, `mx:intentional`, `mx:todo`, `mx:ai`) — each language already has its own TODO-comment and annotation conventions.

If an implementation needs any of these, it uses the relevant external convention. This note does not duplicate them.

---

## 5. Vendor extensions

Vendor extensions to code carriers follow a three-level namespace policy:

- **Standard** (no prefix): the universal identity vocabulary plus the fields in §3 of this note.
- **Vendor public** (`x-<vendor>-` prefix): vendor-specific code-carrier additions, openly published.
- **Vendor private** (`x-<vendor>-p-` prefix): vendor-specific additions, obfuscated or registry-decoded.

Implementations MUST NOT add a prefix to standard fields, and MUST NOT use `x-` prefixes for fields that already belong to the standard vocabulary.

---

## 6. Relationship to existing standards

MX is explicit about deferring to established vocabularies. Within code carriers specifically:

- **JSDoc / TSDoc / Python docstrings / Doxygen / rustdoc / godoc** — MX does not redefine `@param`, `@returns`, `@throws`, `@example`, or any other documentation-convention tag. Code-level documentation is each language's own convention.
- **OpenAPI** — for APIs that a code file implements, full API surface specification (request/response schemas, error codes, examples) uses [OpenAPI](https://www.openapis.org/). This note does not duplicate OpenAPI.
- **Package manifests** (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`) — dependency declarations are the native manifest's responsibility; this note does not add parallel dependency vocabulary.
- **Dublin Core / Schema.org** — generic metadata (dates, rights, formats, display names) uses pass-through fields aligned with `dc:*` and `schema:*`, not code-specific redefinitions.
- **Test framework conventions** — unit/integration/e2e classification, coverage targets, and fixtures are owned by the test framework (mocha, pytest, cargo test). This note does not declare test metadata fields.

---

## 7. References

### 7.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels

### 7.2 Informative references — external standards MX defers to

- [DCAT v3](https://www.w3.org/TR/vocab-dcat-3/) — datasets and data catalogues
- [CSVW](https://www.w3.org/TR/tabular-metadata/) — tabular data schemas
- [Dublin Core](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) — generic resource metadata
- [Schema.org](https://schema.org/) — web content vocabulary
- [EXIF](https://www.cipa.jp/std/documents/e/DC-008-Translation-2019-E.pdf) / [IPTC](https://iptc.org/standards/photo-metadata/) / [XMP](https://developer.adobe.com/xmp/docs/XMPSpecifications/) / [ID3](https://id3.org/) — embedded media metadata
- [OpenAPI](https://www.openapis.org/) — API surface specification
- [JSDoc](https://jsdoc.app/) / [Python Docstring (PEP 257)](https://peps.python.org/pep-0257/) / [Doxygen](https://www.doxygen.nl/) / [rustdoc](https://doc.rust-lang.org/rustdoc/) / [godoc](https://pkg.go.dev/cmd/doc) — code behaviour documentation
- [IETF RFC format](https://www.rfc-editor.org/rfc/rfc7322) — standards-document authoring
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (conformance level model that inspired the Level 1/2/3 framework in §2.1)
- [Schema.org `codeRepository`](https://schema.org/codeRepository) — alignment for `sourceRepo`

---

## 8. Changelog

- **2026-04-27 (v1.1-draft)** — Renamed from "MX Carrier Formats Standard" to "MX Carrier Formats note" to clarify these are draft notes by Tom Cranstoun, not ratified standards. Made the note standalone — removed cross-references to other Gathering drafts and inlined required material.
- **2026-04-17 (v1.1-proposed)** — cut from forty fields to two. Scope tightened to code-specific provenance only; behavioural vocabulary removed and deferred to language conventions.
- **2026-04-15 (v1.0-proposed)** — initial draft. Code-only scope. Databases and media explicitly deferred to DCAT/CSVW/Schema.org/EXIF/XMP per the "reuse existing standards, do not duplicate" principle.

---

*End of MX Carrier Formats note draft.*
