---
title: "MX Carrier Formats note"
docname: draft-cranstoun-mx-carrier-formats
date: 2026-04-27
consensus: false
keyword:
  - mx
  - carriers
  - markdown
  - html
  - jsdoc
  - css
  - xmp
  - sidecar
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Carrier Formats note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies how MX metadata is carried across file formats. It covers two layers:

1. **Carrier mechanisms** — the syntactic envelope each file format uses to host MX metadata: YAML frontmatter for markdown, `<meta>` tags for HTML, JSDoc comments for JavaScript, CSS comments, shell comment blocks, XMP for media, sidecar files, and SQL comment blocks.
2. **Code-specific provenance vocabulary** — a minimum viable set of fields specific to source code as a document and not covered by the universal identity vocabulary.

The note also defines the `mx:*` identity fields used by carriers that do not have YAML frontmatter (HTML, JavaScript, CSS).

What the code does — function signatures, API surfaces, test metadata, type systems, invariants, behaviour — is explicitly out of scope. Each language already has its own convention for this (JSDoc, Python docstrings, Doxygen, rustdoc, godoc). MX does not duplicate them.

Database-content vocabulary and media-content vocabulary are also out of scope; this note specifies only how a document of either kind carries the universal MX identity fields. The MX framework reuses existing standards for the content layer:

- **Databases** — defer to [DCAT v3](https://www.w3.org/TR/vocab-dcat-3/) (datasets, distributions, catalogues), [CSVW](https://www.w3.org/TR/tabular-metadata/) (tabular schemas, columns, keys), and [Dublin Core](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) (dataset identity).
- **Media** — defer to [Schema.org](https://schema.org/) (`ImageObject`, `VideoObject`, `AudioObject`, `CreativeWork`, `license`) and the native embedded-metadata standards (EXIF, IPTC, XMP, ID3).
- **Code behaviour** — defer to the language's own documentation convention.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field — heading, property table, definition prose, example — and is binding on this note. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

This note defines three conformance levels (modelled on the [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level A/AA/AAA pattern):

| Level | Name | Requirement |
|-------|------|-------------|
| Level 1 | **MX Core** | Universal identity fields present (`title`, `author`, `created`, `description`); the carrier syntax for the file's format is used correctly. |
| Level 2 | **MX Standard** | Adds Level 1 plus the operational fields applicable to the format (`status`, `contentType`, `license`); for non-YAML carriers, the `mx:*` identity fields in §4 are used where applicable. |
| Level 3 | **MX Complete** | Adds Level 2 plus the code-specific provenance fields in §5 where they apply, and any `MAY`-level identity fields. |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Selective adoption

An implementation is **carrier-conformant at Level N** if, for every artefact it emits, it satisfies the Level-N requirements for the artefact's carrier format. Implementations MAY claim conformance for a subset of carriers (e.g. markdown and HTML only).

### 2.3 Baseline identity contract

Every MX-aware document, regardless of carrier, carries the universal Level 1 identity contract:

- `title` — REQUIRED
- `author` — REQUIRED
- `created` — REQUIRED (ISO 8601 date format YYYY-MM-DD)
- `description` — RECOMMENDED

The carrier-specific syntax for these fields is defined in §3.

### 2.4 Database and media content conformance

For dataset, table, column, image, video, and audio content metadata, implementations look to the external standards cited in §1. MX makes no conformance claim for content vocabularies beyond the carrier mechanisms in §3.

### 2.5 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions, conformance requirements, and normative rules in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Carrier format mappings

This section defines how each supported file format carries MX metadata.

### 3.1 `.cog.md` — YAML frontmatter

| Property | Value |
|----------|-------|
| **File types** | `.cog.md`, `.md`, `.mx.yaml.md` |
| **Carrier mechanism** | YAML frontmatter between `---` delimiters |
| **Field naming** | camelCase (operational fields under `mx:` object) |
| **Conformance** | MUST (Level 1) for all markdown-based MX documents |

The primary carrier format. All MX-aware markdown documents carry metadata in YAML frontmatter using a two-zone model: top-level fields for document identity (title, description, author, dates, version) and the `mx:` object for MX-operational fields. Implementations MUST NOT place identity fields inside the `mx:` object, and MUST NOT place operational fields at the top level.

**Example:**

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
---
```

Extension fields appear inside the `mx:` object alongside standard operational fields.

---

### 3.2 `.cog.html` — HTML meta tags

| Property | Value |
|----------|-------|
| **File types** | `.cog.html`, `.html` |
| **Carrier mechanism** | `<meta name="mx:*">` tags in `<head>`, `data-mx-*` attributes on elements |
| **Field naming** | kebab-case with `mx:` prefix |
| **Conformance** | MUST (Level 2) for HTML documents claiming MX conformance |
| **Required fields** | `description` (standard HTML meta) + `author` (standard HTML meta) + at least one `<meta name="mx:*">` tag |

HTML documents carry MX metadata using `<meta>` tags in the document `<head>`. The `mx:` prefix in the `name` attribute distinguishes MX metadata from standard HTML meta tags.

**Example:**

```html
<head>
  <meta name="description" content="Brief summary for search and AI agents">
  <meta name="author" content="Tom Cranstoun">
  <meta name="mx:status" content="active">
  <meta name="mx:content-type" content="guide">
  <meta name="mx:tags" content="metadata, example">
</head>
```

- Standard HTML meta tags (`description`, `author`) MUST use their native HTML names without the `mx:` prefix.
- MX-specific fields MUST use the `mx:` prefix in the `name` attribute.
- Array values MUST be serialised as comma-separated strings.
- `data-mx-*` attributes MAY be used on body elements for element-level MX metadata.

---

### 3.3 `.cog.js` — JSDoc comments

| Property | Value |
|----------|-------|
| **File types** | `.cog.js`, `.js` |
| **Carrier mechanism** | JSDoc `/** */` block with `@mx:` tags |
| **Field naming** | kebab-case with `@mx:` tag prefix |
| **Conformance** | MUST (Level 2) for JavaScript documents claiming MX conformance |
| **Required fields** | `@description` + `@version`/`@author` + at least one `@mx:*` tag |

JavaScript files carry MX metadata in a JSDoc comment block at the top of the file. MX-specific fields use the `@mx:` tag prefix.

**Example:**

```javascript
/**
 * @description MX validation utility
 * @version 1.0
 * @author Tom Cranstoun
 * @mx:status active
 * @mx:content-type utility
 * @mx:runtime node
 * @mx:tags validation, metadata
 */
```

- Standard JSDoc tags (`@description`, `@version`, `@author`) MUST use their native JSDoc names.
- MX-specific fields MUST use the `@mx:` prefix.
- The JSDoc block SHOULD appear at the top of the file, before any code.

---

### 3.4 `.cog.css` — CSS comments

| Property | Value |
|----------|-------|
| **File types** | `.cog.css`, `.css` |
| **Carrier mechanism** | CSS comment `/* */` block with `@mx:` tags |
| **Field naming** | kebab-case with `@mx:` tag prefix |
| **Conformance** | MUST (Level 2) for CSS documents claiming MX conformance |
| **Required fields** | `@description` + `@version`/`@author` + at least one `@mx:*` tag |

CSS files carry MX metadata in a comment block at the top of the file. MX-specific fields use the `@mx:` tag prefix.

**Example:**

```css
/*
 * @description Global layout styles
 * @version 2.0
 * @author Tom Cranstoun
 * @mx:status active
 * @mx:type utility
 * @mx:tags layout, responsive
 */
```

The metadata comment block SHOULD appear at the top of the file, before any rules.

---

### 3.5 `.cog.png` / `.cog.jpg` — EXIF/XMP metadata

| Property | Value |
|----------|-------|
| **File types** | `.cog.png`, `.cog.jpg`, `.png`, `.jpg`, `.webp`, `.svg` |
| **Carrier mechanism** | EXIF/XMP metadata with `mx:` namespace |
| **Field naming** | `mx:` namespace prefix, original case preserved |
| **Conformance** | MAY (Level 3) |

Image files carry MX identity metadata in EXIF/XMP metadata blocks using the `mx:` XML namespace. When XMP embedding is not practical, a sidecar file (`.mx.yaml`) SHOULD be used instead. Image-content vocabulary (subject, alt text, EXIF camera fields) is governed by EXIF/IPTC/XMP/Schema.org and is not redefined here.

**Example (XMP):**

```xml
<rdf:Description xmlns:mx="https://allabout.network/ns/mx/1.0">
  <mx:status>active</mx:status>
  <mx:contentType>illustration</mx:contentType>
</rdf:Description>
```

The MX XMP namespace URI is `https://allabout.network/ns/mx/1.0`.

---

### 3.6 Shell scripts — comment block

| Property | Value |
|----------|-------|
| **File types** | `.sh`, `.bash`, `.zsh` |
| **Carrier mechanism** | `# ---` YAML block with `#` prefix on each line |
| **Field naming** | camelCase in `# key: value` lines |
| **Conformance** | MUST (Level 2) for shell scripts claiming MX conformance |
| **Required fields** | `title`, `description`, `status`, `author` |

Shell scripts carry MX metadata in a comment block using `# ---` delimiters, with each YAML line prefixed by `#`.

**Example:**

```bash
#!/bin/bash
# ---
# title: "Build and deploy"
# description: "Builds site artefacts and deploys to Cloudflare"
# author: "Tom Cranstoun"
# status: active
# contentType: script
# ---
```

- The metadata block MUST start and end with `# ---`.
- Field names use camelCase, matching YAML conventions.
- The metadata block SHOULD appear immediately after the shebang line.

---

### 3.7 `.mx.yaml` — media sidecar

| Property | Value |
|----------|-------|
| **File types** | `.mx.yaml` (placed alongside a media asset) |
| **Carrier mechanism** | Standalone YAML file |
| **Field naming** | camelCase |
| **Conformance** | MAY (Level 3) |

A sidecar metadata file placed alongside a media asset (image, video, audio, binary) that cannot carry embedded metadata. The sidecar file name matches the asset file name with `.mx.yaml` appended.

**Example (for `hero-image.webp`):**

File: `hero-image.webp.mx.yaml`

```yaml
title: "Hero banner image"
description: "Full-width hero image for the landing page"
author: "Tom Cranstoun"
created: 2026-04-02
modified: 2026-04-16

mx:
  status: active
  contentType: image
  tags: [hero, banner, landing]
```

The sidecar file MUST use the two-zone model and the file name MUST match the asset file name with `.mx.yaml` appended.

---

### 3.8 `mx.yaml` — code repository root

| Property | Value |
|----------|-------|
| **File types** | `mx.yaml` (at repository root) |
| **Carrier mechanism** | Standalone YAML file |
| **Field naming** | camelCase |
| **Conformance** | MAY (Level 3) |

A repository-level metadata file placed at the root of a code repository. Provides MX metadata for the repository as a whole.

**Example:**

```yaml
title: "MX Hub"
description: "Central hub for Machine Experience development"
author: "Tom Cranstoun"
created: 2026-01-15
modified: 2026-04-16

mx:
  status: active
  contentType: repository
  domain: machine-experience
  tags: [hub, mx, repository]
```

---

### 3.9 Database sidecars and inline SQL

| Property | Value |
|----------|-------|
| **File types** | `mx.database.yaml` (sidecar), `.sql` (inline) |
| **Carrier mechanism** | Standalone YAML sidecar or `-- @mx ... -- @mx:end` comment blocks in SQL |
| **Field naming** | camelCase |
| **Conformance** | MAY (Level 3) |

Database schemas carry MX identity metadata either in a standalone sidecar file (`mx.database.yaml`) or in inline SQL comments using the `-- @mx` / `-- @mx:end` delimiters. Database-content vocabulary (table relationships, column types, dataset semantics) is governed by DCAT, CSVW, and Dublin Core and is not redefined here.

**Example (sidecar):**

File: `mx.database.yaml`

```yaml
title: "Customer database schema"
description: "Schema metadata for the customer CRM database"
author: "Tom Cranstoun"

mx:
  status: active
  contentType: database-schema
  domain: crm
```

**Example (inline SQL):**

```sql
-- @mx
-- title: Customer contacts table
-- description: Primary contact records for CRM
-- status: active
-- contentType: database-table
-- @mx:end
CREATE TABLE contacts (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);
```

- Inline SQL metadata blocks MUST start with `-- @mx` and end with `-- @mx:end`.
- Each field line within the block MUST be prefixed with `--`.
- Field names use camelCase.

---

## 4. Non-YAML identity fields

The following fields provide MX identity and classification metadata in carrier formats that do not use YAML frontmatter (HTML, JavaScript, CSS, image XMP). All fields use the `mx:` namespace prefix in their respective carrier contexts.

When a field has a YAML equivalent (e.g. `title`, `contentType`), the YAML form is used in YAML-bearing carriers and the `mx:*` form is used elsewhere. The two are aliases for the same semantic field.

### 4.1 `mx:name`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | SHOULD (Level 2) |

Cog name, equivalent to `title` in YAML. Provides a machine-readable identifier in carrier formats that lack YAML frontmatter.

```html
<meta name="mx:name" content="validation-utility">
```

```javascript
/** @mx:name validation-utility */
```

### 4.2 `mx:version`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | SHOULD (Level 2) |

Version string, equivalent to `version` in YAML.

```html
<meta name="mx:version" content="2.0">
```

```css
/* @mx:version 2.0 */
```

### 4.3 `mx:type`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html |
| **Conformance** | SHOULD (Level 2) |

Domain type classification, equivalent to `contentType` in YAML. The shorter name `type` is used in HTML to follow web conventions.

```html
<meta name="mx:type" content="guide">
```

### 4.4 `mx:purpose`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |

Why this file exists. Equivalent to `purpose` in YAML.

```javascript
/** @mx:purpose validation */
```

```css
/* @mx:purpose layout */
```

### 4.5 `mx:audience`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |

Intended readership. Equivalent to `audience` in YAML. Array values are expressed as a comma-separated string.

```javascript
/** @mx:audience developers, agents */
```

### 4.6 `mx:stability`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |
| **Valid values** | stable, evolving, experimental, deprecated, archived |

Stability level. Communicates how reliable this file's interface is for external consumers.

- `stable` — interface is fixed; breaking changes require a major version bump.
- `evolving` — interface may change; consumers should pin versions.
- `experimental` — interface is subject to significant change without notice.
- `deprecated` — interface is scheduled for removal; consumers should migrate.
- `archived` — interface is no longer maintained.

```javascript
/** @mx:stability stable */
```

### 4.7 `mx:category`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | MAY (Level 3) |

Cog category, equivalent to `category` in YAML. Determines registry grouping in non-YAML carrier formats.

```html
<meta name="mx:category" content="mx-tool">
```

### 4.8 `mx:tags`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | MAY (Level 3) |

Discovery keywords as a comma-separated string. Equivalent to `tags` (array) in YAML. Values SHOULD be lowercase strings.

```html
<meta name="mx:tags" content="validation, metadata, utility">
```

### 4.9 `mx:builds-on`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |

Context dependencies as a comma-separated list of cog names. Equivalent to `buildsOn` (array) in YAML. Uses kebab-case in non-YAML contexts.

```javascript
/** @mx:builds-on fields, cog-unified-spec */
```

### 4.10 `mx:documented-in`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |

Path to companion documentation. Links a code file to its narrative documentation.

```javascript
/** @mx:documented-in ../docs/validation-guide.cog.md */
```

### 4.11 `mx:context-provides`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |

What context this file provides to agents. Declares the knowledge or capability an agent gains by reading this file.

```javascript
/** @mx:context-provides Field validation rules and error message templates */
```

---

## 5. Code-specific provenance vocabulary

This section adds two fields specific to source code as a document. They are not covered by the universal identity vocabulary or by the language's own documentation convention.

### 5.1 Fields

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| `sourceRepo` | string | OPTIONAL | URL or path of the source-control repository that holds this code file's authoritative history. Aligns with [Schema.org `codeRepository`](https://schema.org/codeRepository). |
| `derivedFromCommit` | string | OPTIONAL | Version-control commit identifier (typically a git SHA) that this artefact was derived from. Pairs with a generic `derivedFrom` provenance pointer to add commit-level specificity. |

Every other concern that a wider draft might cover — function annotations, API surface, test metadata, class invariants, dependency declarations, runtime config, build conventions — is either out of scope entirely (deferred to the language convention) or handled generically by universal identity fields.

### 5.2 Carrier syntax

Code carriers use the native metadata mechanism of their file format. Field names are identical across all carriers; only the host syntax changes.

| Carrier | Native mechanism | Example |
|---------|------------------|---------|
| JavaScript / TypeScript | JSDoc comments | `@mx:sourceRepo https://github.com/example/repo` |
| CSS | CSS block comments | `/* @mx:sourceRepo https://github.com/example/repo */` |
| Shell | Shell comment block | `# sourceRepo: https://github.com/example/repo` (after shebang) |
| SQL | SQL comment block | `-- @mx sourceRepo: https://github.com/example/repo` |
| Any | YAML frontmatter in `.cog.*` files | `sourceRepo: https://github.com/example/repo` |

A JavaScript file declaring `@mx:sourceRepo` and a `.cog.md` declaring `sourceRepo:` carry the same semantic value.

### 5.3 Profile

This note declares a single profile for code: `code`. An implementation handling code artefacts applies the `code` profile's fields in addition to the universal identity fields. There are no per-language sub-profiles — the vocabulary is language-agnostic.

---

## 6. Vendor extensions

Carrier formats accept vendor extension fields under a three-level namespace policy:

- **Standard** (no prefix): the universal identity vocabulary plus the fields in this note.
- **Vendor public** (`x-<vendor>-` prefix): vendor-specific carrier-format additions, openly published.
- **Vendor private** (`x-<vendor>-p-` prefix): vendor-specific additions, obfuscated or registry-decoded.

Implementations MUST NOT add a prefix to standard fields, and MUST NOT use `x-` prefixes for fields that already belong to the standard vocabulary.

The detailed namespace policy (governance, prefix-parsing rules, registration process) is specified in a separate note covering MX extensions.

---

## 7. Out of scope

This note is scoped to carrier mechanisms and the small code-specific provenance vocabulary. The following concerns are explicitly out of scope:

- **Function-level annotations** (`pure`, `idempotent`, `complexity`, `throws`, `returns`, `param`) — defer to JSDoc, Python docstring convention, Doxygen, rustdoc, godoc.
- **API surface specification** (HTTP method, path, auth, rate limits, cache semantics) — defer to [OpenAPI](https://www.openapis.org/).
- **Test metadata** (test type, coverage targets, fixtures) — defer to test framework conventions.
- **Class-level annotations** (design pattern, thread-safety, invariants) — defer to language convention and code reviews.
- **Dependency vocabulary** (purpose, criticality, upgrade policy, alternatives considered) — defer to package manifests (`package.json`, `pyproject.toml`, `Cargo.toml`) and Architecture Decision Records.
- **Runtime configuration** (interpreter, language version, framework versions) — defer to native config (`.node-version`, `.tool-versions`, shebang lines, package manifests).
- **Coding conventions** (style, testing conventions, documentation conventions) — defer to linter configs and community style guides.
- **Inline code annotations** (`mx:begin`, `mx:end`, `mx:sensitive`, `mx:intentional`, `mx:todo`, `mx:ai`) — each language already has its own TODO-comment and annotation conventions.
- **Database content vocabulary** (table relationships, column types, dataset semantics) — defer to DCAT v3, CSVW, Dublin Core.
- **Image content vocabulary** (subject, alt text, EXIF camera fields) — defer to EXIF, IPTC, XMP, Schema.org.
- **Audio/video content vocabulary** (duration, bitrate, captions, chapters) — defer to ID3, XMP, Schema.org.
- **Document identity field semantics** (what `title`, `author`, `created`, `description` mean) — governed by the MX Core Metadata note.
- **Cog file format internals** (the `.cog.md` body grammar, content blocks) — governed by the MX Cogs note.

If an implementation needs any of these, it uses the relevant external convention or sibling note. This note does not duplicate them.

---

## 8. Relationship to existing standards

MX is explicit about deferring to established vocabularies:

- **JSDoc / TSDoc / Python docstrings / Doxygen / rustdoc / godoc** — code-level documentation is each language's own convention.
- **OpenAPI** — full API surface specification.
- **Package manifests** (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`) — dependency declarations.
- **Dublin Core / Schema.org** — generic metadata (dates, rights, formats, display names).
- **Test framework conventions** — unit/integration/e2e classification, coverage targets, fixtures.
- **DCAT / CSVW** — dataset and tabular data metadata.
- **EXIF / IPTC / XMP / ID3** — embedded media metadata.
- **HTML `<meta>` standard** — base mechanism for `<meta name="mx:*">` carriers.
- **XMP namespace mechanism** — base mechanism for the `mx:` XMP namespace at `https://allabout.network/ns/mx/1.0`.

---

## 9. References

### 9.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels

### 9.2 Informative references — external standards MX defers to

- [DCAT v3](https://www.w3.org/TR/vocab-dcat-3/) — datasets and data catalogues
- [CSVW](https://www.w3.org/TR/tabular-metadata/) — tabular data schemas
- [Dublin Core](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) — generic resource metadata
- [Schema.org](https://schema.org/) — web content vocabulary
- [EXIF](https://www.cipa.jp/std/documents/e/DC-008-Translation-2019-E.pdf) / [IPTC](https://iptc.org/standards/photo-metadata/) / [XMP](https://developer.adobe.com/xmp/docs/XMPSpecifications/) / [ID3](https://id3.org/) — embedded media metadata
- [OpenAPI](https://www.openapis.org/) — API surface specification
- [JSDoc](https://jsdoc.app/) / [Python Docstring (PEP 257)](https://peps.python.org/pep-0257/) / [Doxygen](https://www.doxygen.nl/) / [rustdoc](https://doc.rust-lang.org/rustdoc/) / [godoc](https://pkg.go.dev/cmd/doc) — code behaviour documentation
- [IETF RFC format](https://www.rfc-editor.org/rfc/rfc7322) — standards-document authoring
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (conformance level model)
- [Schema.org `codeRepository`](https://schema.org/codeRepository) — alignment for `sourceRepo`

---

*End of MX Carrier Formats note draft.*
