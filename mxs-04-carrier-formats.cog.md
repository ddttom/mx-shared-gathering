---
title: "MX Carrier Formats Standard"
version: "1.0-proposed"
created: 2026-04-15
modified: 2026-04-16
author: The Gathering
description: "Formal specification of MX metadata fields for code carrier formats. Companion to MXS-01 Core Metadata. Databases and media defer to existing standards (DCAT, CSVW, Schema.org, EXIF/XMP) rather than being redefined here."

mx:
  status: proposed
  license: MIT
  category: standard
  partOf: mx-the-gathering
  contentType: specification
  buildsOn: [mxs-01-core-metadata, mxs-02-extensions]
  tags: [standard, carriers, code, specification]
  audience: [humans, machines]
  cacheability: permanent
  runbook: "This is the code-carrier companion to MXS-01. It defines the MX metadata vocabulary for code files (functions, classes, APIs, tests, inline annotations, dependencies, repositories). Database and media carriers are explicitly out of scope; implementations defer to DCAT, CSVW, Dublin Core, Schema.org, and EXIF/XMP for those. An implementation that handles source code SHOULD adopt the relevant code profiles; other implementations MAY ignore this standard."
---

# MX Carrier Formats Standard

**Version:** 1.0-proposed
**Status:** Proposed (draft for Stream submission, awaiting community review)
**Date:** 16 April 2026
**Governing body:** The Gathering
**License:** MIT

---

## 1. Abstract

This document specifies the MX metadata vocabulary for source code as a carrier format. Where [MXS-01 Core Metadata](mxs-01-core-metadata.cog.md) defines the universal field dictionary any MX implementation needs, this standard adds schemas for **code** — the one carrier format for which no existing standard covers AI-governance-focused annotation.

Database and media carriers are deliberately out of scope. The MX framework reuses existing standards:

- **Databases** — defer to [DCAT v3](https://www.w3.org/TR/vocab-dcat-3/) (datasets, distributions, catalogues), [CSVW](https://www.w3.org/TR/tabular-metadata/) (tabular schemas, columns, keys), and [Dublin Core](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) (dataset identity).
- **Media** — defer to [Schema.org](https://schema.org/) (`ImageObject`, `VideoObject`, `AudioObject`, `CreativeWork`, `license`) and the native embedded-metadata standards (EXIF, IPTC, XMP, ID3).

An MX-aware document describing a dataset or an image declares its metadata using those vocabularies directly; MX does not reinvent them. For code, no single equivalent standard covers the AI-agent-focused governance annotations MX needs (`mx:sensitive`, `mx:intentional`, `ai.doNotModify`, rate-limit declarations on API surfaces) — so this standard fills that gap.

**Machine-readable source.** The canonical field dictionary for this standard is [`mx-canon/ssot/fields-data-carriers.yaml`](../../../mx-canon/ssot/fields-data-carriers.yaml) — ~40 code-family fields. Together with [`mx-canon/ssot/fields-data.yaml`](../../../mx-canon/ssot/fields-data.yaml) (the core) they constitute The Gathering's complete open-standard metadata vocabulary.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

This standard adopts the Level 1 / Level 2 / Level 3 conformance framework defined in [MXS-01 §2.1](mxs-01-core-metadata.cog.md#2-conformance) by reference.

### 2.1 Selective adoption

An implementation is **code-conformant at Level N for profile P** if, for every code artefact it emits that matches profile P, it declares the Level-N-required fields of profile P. Implementations MAY claim conformance for any subset of code profiles. The claim is per-profile, not global.

Example: a documentation CMS that renders syntax-highlighted code blocks inside prose MAY claim Level 2 conformance for profile `code-inline` without claiming conformance for `code-api` or `code-repository`.

### 2.2 Baseline dependency

Every code-carrier document inherits the Level 1 identity fields from MXS-01 (`title`, `author`, `created`, `description`). A code-carrier document MUST satisfy those identity fields regardless of which code profile it targets.

### 2.3 Database and media conformance

Implementations seeking metadata conformance for datasets, tables, columns, images, video, or audio look to the external standards cited in §1 Abstract. MX makes no conformance claim for those carriers.

---

## 3. Code profile family

Applies to source code files — functions, classes, API endpoint definitions, tests, inline annotations, dependency declarations, and repository-level metadata. Used when a document IS a code artefact or embeds code as first-class content.

### 3.1 Canonical carrier syntax

Per MXS-02, code carriers use the native metadata mechanism of their file format:

| Carrier | Native mechanism | Syntax |
|---------|------------------|--------|
| JavaScript / TypeScript | JSDoc comments | `@mx:<field> <value>` |
| CSS | CSS block comments | `/* @mx:<field> <value> */` |
| Shell | Shell block comments | `# <field>: <value>` (after shebang) |
| Any | YAML frontmatter in `.cog.*` files | camelCase field names |

Inline code annotations (`mx:begin` / `mx:end` pairs, `mx:intentional`, `mx:sensitive`, `mx:ai`, `mx:todo`) use the same mechanism.

### 3.2 Profiles

| Profile | Purpose | Representative fields |
|---------|---------|----------------------|
| `code-file` | File-level metadata | `owner`, `pattern`, `moduleType` |
| `code-function` | Function-level metadata | `pure`, `idempotent`, `complexity`, `throws`, `pattern` |
| `code-class` | Class-level metadata | `pattern`, `extends`, `implements` |
| `code-api` | API endpoint metadata | `endpoint`, `method`, `auth`, `rateLimit`, `cache.*` |
| `code-test` | Test file metadata | `testType`, `fixtures`, `coverageTarget` |
| `code-inline` | Inline annotations | `mx:begin`, `mx:end`, `mx:intentional`, `mx:sensitive`, `mx:todo`, `mx:ai` |
| `code-dependency` | Dependency metadata | `dependencyOf`, `peerDependencies` |
| `code-repository` | Repository-level | `primaryLanguages`, `branchConvention`, `releaseCadence` |
| `script` | Executable scripts | `script-specific primitives` |

Full field inventory lives in [`fields-data-carriers.yaml`](../../../mx-canon/ssot/fields-data-carriers.yaml).

---

## 4. Interaction with MXS-01

Every code-carrier document inherits the Zone 1 identity contract from MXS-01 §3:

- `title` REQUIRED
- `author` REQUIRED
- `created` REQUIRED
- `description` RECOMMENDED

These apply regardless of profile. A `.cog.js` file MUST declare these in its top-level JSDoc or an explicit frontmatter block.

Zone 2 operational fields from MXS-01 §4 (`status`, `contentType`, `tags`, `audience`, `license`, `runbook`) remain available to code documents and SHOULD be used where they apply.

---

## 5. Vendor extensions

Vendor extensions to code profiles follow the same three-level namespace policy as MXS-02:

- **Standard** (no prefix): this document.
- **Vendor public** (`x-<vendor>-` prefix): vendor-specific additions, openly published.
- **Vendor private** (`x-<vendor>-p-` prefix): vendor-specific additions, obfuscated or registry-decoded.

CogNovaMX's code-related extensions live in `cognovamx-fields.yaml` and are not part of this standard.

---

## 6. Relationship to existing standards

MX is explicit about deferring to established vocabularies. Within code carriers specifically:

- **JSDoc / TSDoc** — MX does not redefine `@param`, `@returns`, `@throws`, or other JSDoc tags. MX adds `@mx:*` tags for governance concerns JSDoc does not cover (AI policy, security annotations, pipeline stage).
- **OpenAPI** — `code-api` fields describe a function that serves an API endpoint. For full API surface specification (request/response schemas, error codes, examples), implementations use [OpenAPI](https://www.openapis.org/). MXS-04 does not duplicate OpenAPI.
- **package.json / pyproject.toml** — `code-dependency` fields supplement, not replace, native dependency manifests.
- **Dublin Core / Schema.org** — generic metadata (dates, rights, formats, display names) uses the pass-through fields declared in MXS-01 (aligned with `dc:*` and `schema:*`), not code-specific redefinitions.

---

## 7. Registry & references

- **Machine-readable:** [`mx-canon/ssot/fields-data-carriers.yaml`](../../../mx-canon/ssot/fields-data-carriers.yaml)
- **Human-readable:** Appendix M §25 (Carrier Format Metadata Map), §26 (HTML Carrier Writing Guide)
- **Companion standards:** [MXS-01 Core Metadata](mxs-01-core-metadata.cog.md), [MXS-02 Extensions](mxs-02-extensions.cog.md)
- **External standards MX defers to:**
  - [DCAT v3](https://www.w3.org/TR/vocab-dcat-3/) — datasets and data catalogues
  - [CSVW](https://www.w3.org/TR/tabular-metadata/) — tabular data schemas
  - [Dublin Core](https://www.dublincore.org/specifications/dublin-core/dcmi-terms/) — generic resource metadata
  - [Schema.org](https://schema.org/) — web content vocabulary (ImageObject, VideoObject, AudioObject, CreativeWork)
  - [EXIF](https://www.cipa.jp/std/documents/e/DC-008-Translation-2019-E.pdf) / [IPTC](https://iptc.org/standards/photo-metadata/) / [XMP](https://developer.adobe.com/xmp/docs/XMPSpecifications/) — embedded media metadata

---

## 8. Open questions

These are flagged for Gathering review before 1.0 publication:

1. **Aspirational vs ratified fields.** The code dictionary currently contains several fields with zero observed usage (e.g. `fixtures`, `coverageTarget`, `rateLimit`). Should the 1.0 standard ratify them (forward contract) or defer until documented usage exists?
2. **Overlap with JSDoc / TSDoc extensions.** Several JavaScript/TypeScript ecosystems define their own JSDoc extensions (Google Closure, TypeDoc, etc.). Should MXS-04 document known overlaps and preferred resolutions, or stay orthogonal?
3. **Inline annotation delimiters.** The `mx:begin` / `mx:end` pair requires paired parsing. Would single-statement annotations (one comment per governed line) be simpler for implementers?

---

## 9. Changelog

- **2026-04-15** — 1.0-draft initial draft. Code-only scope. Databases and media explicitly deferred to DCAT/CSVW/Schema.org/EXIF/XMP per The Gathering's "reuse existing standards, do not duplicate" principle.

---

*End of MX Carrier Formats Standard draft.*
