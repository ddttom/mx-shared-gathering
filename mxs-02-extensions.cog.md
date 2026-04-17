---
title: "MX Extensions Standard"
version: "1.0-proposed"
created: 2026-04-02
modified: 2026-04-16
author: The Gathering
description: "Formal specification of the MX namespace policy, carrier format mappings, and extension mechanisms for non-standard metadata fields."

mx:
  status: proposed
  license: MIT
  x-mx-category: standard
  partOf: mx-the-gathering
  contentType: specification
  buildsOn: [cog-unified-spec, mxs-01-core-metadata]
  tags: [standard, extensions, namespace, carrier-formats, specification]
  audience: [humans, machines]
  cacheability: permanent
  runbook: "This is the MX extensions standard. It defines the three-level namespace policy (standard, x-mx-, x-mx-p-), context-specific naming rules for different carrier formats, carrier format mappings for all supported file types, and the extension registration process. Cross-reference the MX Core Metadata Standard for conformance level definitions."
---

# MX Extensions Standard

**Version:** 1.0-proposed
**Status:** Proposed (draft for Stream submission, awaiting community review)
**Date:** 16 April 2026
**Governing body:** The Gathering
**License:** MIT

---

## 1. Abstract

This document defines the namespace policy, carrier format mappings, and extension mechanisms for the Machine Experience (MX) framework. It specifies how MX metadata is carried across different file types, how field names adapt to different syntactic contexts, and how vendors extend the MX vocabulary without polluting the standard namespace.

The namespace policy establishes three levels: standard fields (no prefix, owned by The Gathering), public extensions (`x-mx-`, owned by CogNovaMX), and private extensions (`x-mx-p-`, owned by CogNovaMX, values obfuscated). The carrier format mappings define how MX metadata is expressed in markdown, HTML, JavaScript, CSS, images, shell scripts, sidecar files, and database contexts.

This standard adopts the conformance level framework defined in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md).

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

### 2.1 Conformance levels

This standard adopts the three conformance levels defined in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md), Section 2.1:

| Level | Name | Requirement for this document |
|-------|------|-------------------------------|
| Level 1 | **MX Core** | Valid namespace usage — standard fields have no prefix, extensions use `x-mx-` or `x-mx-p-`. No prefix pollution. |
| Level 2 | **MX Standard** | Carrier format metadata present for all applicable file types. |
| Level 3 | **MX Complete** | Full `mx:*` coverage across all carrier contexts. All applicable extension fields present. |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Draft status

This document is a proposed standard under draft by The Gathering. It is authored for submission to the Stream public review process and awaits community ratification. Until ratified, the field definitions, conformance requirements, and normative rules in this document are the working draft — stable enough to build against, expected to evolve through review.

---

## 3. Scope and relationship to other standards

### 3.1 What this document covers

This document specifies:

- **Namespace policy** — the three-level prefix hierarchy and governance rules
- **Context-specific naming** — how the same field appears in YAML, HTML, JSDoc, CSS, shell, XMP, sidecar, and SQL contexts
- **Carrier format mappings** — how MX metadata is carried in each supported file type
- **Non-YAML identity fields** — the `mx:*` equivalents of standard cog fields for non-YAML carrier formats
- **Public extension fields** — the `x-mx-` vendor extension vocabulary
- **Private extension mechanism** — the `x-mx-p-` obfuscated extension system
- **Extension registration process** — how to add new extension fields

### 3.2 What this document does not cover

The following are defined in companion standards:

| Topic | Standard |
|-------|----------|
| Zone 1/Zone 2 core fields, conformance level framework | [MX Core Metadata Standard](mxs-01-core-metadata.cog.md) |
| Trust, attribution, verification, decision records | [MX Provenance Standard](mxs-03-provenance.cog.md) |
| AI governance, agent policies, training controls | [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md) |
| Content-type-specific fields (code, media, database, etc.) | [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) |

### 3.3 Relationship to existing standards

This standard builds upon:

- **[MX Core Metadata Standard](mxs-01-core-metadata.cog.md)** — defines the conformance level framework this document adopts
- **[Cog Unified Specification](../specifications/cog-unified-spec.cog.md)** — defines the cog file format and carrier format details
- **[ADR-02: Namespace Policy](../architecture-decisions/adr-02-namespace-policy.cog.md)** — architecture decision establishing the three-level namespace
- **[NDR-02: camelCase Naming](../naming-decisions/ndr-02-camelcase-naming.cog.md)** — field naming convention for YAML contexts
- **[NDR-03: Spelling Neutrality](../naming-decisions/ndr-03-spelling-neutrality.cog.md)** — field names avoid regional spelling variants
- **HTTP Extension Header Convention** — the `x-` prefix convention for non-standard headers, adapted for metadata namespaces

---

## 4. Terminology

- **Cog** — A `.cog.md` file. The atomic unit of the MX document system. Defined by the [Cog Unified Specification](../specifications/cog-unified-spec.cog.md).
- **Carrier format** — The mechanism by which a non-markdown file carries MX metadata (HTML meta tags, JSDoc comments, CSS comments, etc.).
- **Namespace** — A prefix that identifies the owner and governance policy of a metadata field.
- **Standard field** — A field with no prefix, owned by The Gathering, universal across all MX implementations.
- **Extension field** — A field with an `x-mx-` or `x-mx-p-` prefix, owned by a vendor (currently CogNovaMX).
- **Sidecar file** — A standalone metadata file (`.mx.yaml` or `.mx.yaml.md`) placed alongside a media asset or at a directory root.
- **$MX_HOME** — The local MX OS registry directory. Contains decode tables for private extension values.
- **SSOT** — Single Source of Truth. The authoritative source for a given piece of information.

---

## 5. Design principles

The namespace and carrier format system follows these principles:

1. **The prefix IS the policy.** No documentation lookup is needed to determine governance — `x-` means extension, `mx-` means CogNovaMX, `p-` means private.
2. **Standard fields are universal.** Every MX implementation uses the same core vocabulary with no prefix.
3. **Extensions are opt-in.** No implementation is required to support `x-mx-` or `x-mx-p-` fields.
4. **Carrier formats are lossless.** The same metadata can be expressed in any supported format without information loss.
5. **Context determines syntax, not semantics.** A field means the same thing regardless of whether it appears in YAML, HTML, or JSDoc.

---

## 6. Namespace policy

### 6.1 The three-level namespace

MX metadata fields are organised into three namespace levels. Each level has a distinct prefix, owner, and governance policy.

| Level | Prefix | Owner | Governance | Visibility |
|-------|--------|-------|------------|------------|
| Standard | *(none)* | The Gathering | Community-governed. Changes require Gathering approval. | Universal — all implementations use these fields. |
| Public extension | `x-mx-` | CogNovaMX | Vendor-governed. CogNovaMX decides. | Visible in published cogs. Implementation extension, not the open standard. |
| Private extension | `x-mx-p-` | CogNovaMX | Vendor-governed. CogNovaMX decides. | Obfuscated. Only `$MX_HOME` registry holders can decode values. |

### 6.2 Standard fields (no prefix)

Standard fields are the core MX vocabulary defined in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md) and this document. They have no prefix and are owned by The Gathering.

**Rule:** Implementations MUST NOT add a prefix to standard fields. A field defined in the core standard MUST always appear without a prefix in all contexts.

**Example (correct):**

```yaml
mx:
  status: active
  contentType: guide
```

**Example (incorrect — prefix pollution):**

```yaml
mx:
  x-mx-status: active     # WRONG — status is a standard field
  mx-contentType: guide    # WRONG — contentType is a standard field
```

### 6.3 Public extension fields (`x-mx-`)

Public extension fields use the `x-mx-` prefix. They are owned by CogNovaMX and follow the HTTP extension header convention (`x-` = extension, `mx-` = CogNovaMX's namespace).

**Rule:** Public extension fields MUST use the `x-mx-` prefix. They MUST use kebab-case for the field name portion (unlike standard fields which use camelCase in YAML).

**Rule:** Public extension fields are visible in published cogs and public outputs. They are implementation-specific features, not part of the open standard.

**Example:**

```yaml
mx:
  x-mx-mount-type: personal
  x-mx-mount-swappable: true
```

### 6.4 Private extension fields (`x-mx-p-`)

Private extension fields use the `x-mx-p-` prefix. They are owned by CogNovaMX and their values are hashed, encoded, or tokenised.

**Rule:** Private extension field values MUST be opaque to any system that does not hold the decode registry at `$MX_HOME`.

**Rule:** Implementations MUST NOT attempt to interpret `x-mx-p-` field values without access to the decode registry. Unknown values MUST be passed through unchanged.

**Example:**

```yaml
mx:
  x-mx-p-client-tier: "a3f2c8"
```

### 6.5 Prefix parsing rule

Implementations MUST parse field prefixes using the following precedence:

1. If the field name starts with `x-mx-p-`, it is a **private extension**.
2. If the field name starts with `x-mx-`, it is a **public extension**.
3. Otherwise, it is a **standard field**.

This ordering ensures that `x-mx-p-` is not misidentified as `x-mx-` followed by `p-`.

---

## 7. Context-specific naming

### 7.1 The naming problem

The same MX field must appear in syntactically different contexts — YAML frontmatter, HTML meta tags, JavaScript documentation comments, CSS comments, shell scripts, XMP metadata, sidecar files, and SQL comments. Each context has its own naming conventions.

### 7.2 Naming rules by context

| Context | Convention | Example field | Example syntax |
|---------|-----------|---------------|----------------|
| YAML | camelCase | `buildsOn` | `buildsOn: [cog-unified-spec]` |
| HTML | kebab-case with `mx:` prefix | `mx:content-type` | `<meta name="mx:content-type" content="guide">` |
| JSDoc | kebab-case with `@mx:` tag | `@mx:runtime` | `@mx:runtime node` |
| CSS | kebab-case with `@mx:` comment | `@mx:type` | `/* @mx:type utility */` |
| Shell | camelCase in `# key: value` lines | `contentType` | `# contentType: guide` |
| XMP | `mx:` namespace prefix | `mx:status` | `<mx:status>active</mx:status>` |
| Sidecar | camelCase in YAML | `contentType` | `contentType: guide` |
| SQL | camelCase in `-- @mx` comment blocks | `contentType` | `-- @mx contentType: guide` |

### 7.3 Conversion rules

When converting field names between contexts, implementations MUST apply the following rules:

1. **YAML to HTML:** Convert camelCase to kebab-case, add `mx:` prefix. Example: `contentType` becomes `mx:content-type`.
2. **YAML to JSDoc/CSS:** Convert camelCase to kebab-case, add `@mx:` prefix. Example: `buildsOn` becomes `@mx:builds-on`.
3. **YAML to Shell/Sidecar:** No conversion. Use camelCase as-is.
4. **YAML to SQL:** No conversion. Use camelCase in `-- @mx` comment blocks.
5. **YAML to XMP:** Add `mx:` namespace prefix. Case preserved. Example: `status` becomes `mx:status`.

### 7.4 Extension field naming in non-YAML contexts

Extension fields (`x-mx-` and `x-mx-p-`) already use kebab-case. In non-YAML contexts, the prefix is preserved as-is:

| Context | Standard field example | Extension field example |
|---------|----------------------|------------------------|
| YAML | `contentType: guide` | `x-mx-mount-type: personal` |
| HTML | `<meta name="mx:content-type">` | `<meta name="mx:x-mx-mount-type">` |
| JSDoc | `@mx:content-type guide` | `@mx:x-mx-mount-type personal` |

---

## 8. Carrier format mappings

### 8.1 `.cog.md` — YAML frontmatter

| Property | Value |
|----------|-------|
| **File types** | `.cog.md`, `.md`, `.mx.yaml.md` |
| **Carrier mechanism** | YAML frontmatter between `---` delimiters |
| **Field naming** | camelCase (Zone 2 fields under `mx:` object) |
| **Conformance** | MUST (Level 1) for all markdown-based MX documents |

**Definition:** The primary carrier format. All MX-aware markdown documents carry metadata in YAML frontmatter using the two-zone model defined in the [MX Core Metadata Standard](mxs-01-core-metadata.cog.md), Section 5.

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

**Normative notes:**

- Zone 1 fields MUST appear at the top level.
- Zone 2 fields MUST appear inside the `mx:` object.
- Extension fields appear inside the `mx:` object alongside standard Zone 2 fields.

---

### 8.2 `.cog.html` — HTML meta tags

| Property | Value |
|----------|-------|
| **File types** | `.cog.html`, `.html` |
| **Carrier mechanism** | `<meta name="mx:*">` tags in `<head>`, `data-mx-*` attributes on elements |
| **Field naming** | kebab-case with `mx:` prefix |
| **Conformance** | MUST (Level 2) for HTML documents claiming MX conformance |
| **Required fields** | `description` (standard HTML meta) + `author` (standard HTML meta) + at least one `<meta name="mx:*">` tag |

**Definition:** HTML documents carry MX metadata using `<meta>` tags in the document `<head>`. The `mx:` prefix in the `name` attribute distinguishes MX metadata from standard HTML meta tags.

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

**Normative notes:**

- Standard HTML meta tags (`description`, `author`) MUST use their native HTML names without the `mx:` prefix.
- MX-specific fields MUST use the `mx:` prefix in the `name` attribute.
- Array values MUST be serialised as comma-separated strings.
- `data-mx-*` attributes MAY be used on body elements for element-level MX metadata.

---

### 8.3 `.cog.js` — JSDoc comments

| Property | Value |
|----------|-------|
| **File types** | `.cog.js`, `.js` |
| **Carrier mechanism** | JSDoc `/** */` block with `@mx:` tags |
| **Field naming** | kebab-case with `@mx:` tag prefix |
| **Conformance** | MUST (Level 2) for JavaScript documents claiming MX conformance |
| **Required fields** | `@description` + `@version`/`@author` + at least one `@mx:*` tag |

**Definition:** JavaScript files carry MX metadata in a JSDoc comment block at the top of the file. MX-specific fields use the `@mx:` tag prefix.

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

**Normative notes:**

- Standard JSDoc tags (`@description`, `@version`, `@author`) MUST use their native JSDoc names.
- MX-specific fields MUST use the `@mx:` prefix.
- The JSDoc block SHOULD appear at the top of the file, before any code.

---

### 8.4 `.cog.css` — CSS comments

| Property | Value |
|----------|-------|
| **File types** | `.cog.css`, `.css` |
| **Carrier mechanism** | CSS comment `/* */` block with `@mx:` tags |
| **Field naming** | kebab-case with `@mx:` tag prefix |
| **Conformance** | MUST (Level 2) for CSS documents claiming MX conformance |
| **Required fields** | `@description` + `@version`/`@author` + at least one `@mx:*` tag |

**Definition:** CSS files carry MX metadata in a comment block at the top of the file. MX-specific fields use the `@mx:` tag prefix.

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

**Normative notes:**

- The metadata comment block SHOULD appear at the top of the file, before any rules.
- MX-specific fields MUST use the `@mx:` prefix.

---

### 8.5 `.cog.png` / `.cog.jpg` — EXIF/XMP metadata

| Property | Value |
|----------|-------|
| **File types** | `.cog.png`, `.cog.jpg`, `.png`, `.jpg`, `.webp`, `.svg` |
| **Carrier mechanism** | EXIF/XMP metadata with `mx:` namespace |
| **Field naming** | `mx:` namespace prefix, original case preserved |
| **Conformance** | MAY (Level 3) |

**Definition:** Image files carry MX metadata in EXIF/XMP metadata blocks using the `mx:` XML namespace. When XMP embedding is not practical, a sidecar file (`.mx.yaml`) SHOULD be used instead.

**Example (XMP):**

```xml
<rdf:Description xmlns:mx="https://allabout.network/ns/mx/1.0">
  <mx:status>active</mx:status>
  <mx:contentType>illustration</mx:contentType>
</rdf:Description>
```

**Normative notes:**

- XMP embedding is OPTIONAL. A sidecar file is an acceptable alternative.
- The MX XMP namespace URI is `https://allabout.network/ns/mx/1.0`.

---

### 8.6 Shell scripts — comment block

| Property | Value |
|----------|-------|
| **File types** | `.sh`, `.bash`, `.zsh` |
| **Carrier mechanism** | `# ---` YAML block with `#` prefix on each line |
| **Field naming** | camelCase in `# key: value` lines |
| **Conformance** | MUST (Level 2) for shell scripts claiming MX conformance |
| **Required fields** | `title`, `description`, `status`, `author` |

**Definition:** Shell scripts carry MX metadata in a comment block using `# ---` delimiters, with each YAML line prefixed by `#`.

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

**Normative notes:**

- The metadata block MUST start and end with `# ---`.
- Field names use camelCase, matching YAML conventions.
- The metadata block SHOULD appear immediately after the shebang line.

---

### 8.7 `.mx.yaml` — media sidecar

| Property | Value |
|----------|-------|
| **File types** | `.mx.yaml` (placed alongside a media asset) |
| **Carrier mechanism** | Standalone YAML file |
| **Field naming** | camelCase |
| **Conformance** | MAY (Level 3) |

**Definition:** A sidecar metadata file placed alongside a media asset (image, video, audio, binary) that cannot carry embedded metadata. The sidecar file name matches the asset file name with `.mx.yaml` appended.

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

**Normative notes:**

- The sidecar file MUST use the two-zone model (Zone 1 at top level, Zone 2 under `mx:`).
- The sidecar file name MUST match the asset file name with `.mx.yaml` appended.

---

### 8.8 `mx.yaml` — code repository root

| Property | Value |
|----------|-------|
| **File types** | `mx.yaml` (at repository root) |
| **Carrier mechanism** | Standalone YAML file |
| **Field naming** | camelCase |
| **Conformance** | MAY (Level 3) |

**Definition:** A repository-level metadata file placed at the root of a code repository. Provides MX metadata for the repository as a whole.

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

### 8.9 Database sidecars and inline SQL

| Property | Value |
|----------|-------|
| **File types** | `mx.database.yaml` (sidecar), `.sql` (inline) |
| **Carrier mechanism** | Standalone YAML sidecar or `-- @mx ... -- @mx:end` comment blocks in SQL |
| **Field naming** | camelCase |
| **Conformance** | MAY (Level 3) |

**Definition:** Database schemas carry MX metadata either in a standalone sidecar file (`mx.database.yaml`) or in inline SQL comments using the `-- @mx` / `-- @mx:end` delimiters.

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

**Normative notes:**

- Inline SQL metadata blocks MUST start with `-- @mx` and end with `-- @mx:end`.
- Each field line within the block MUST be prefixed with `--`.
- Field names use camelCase.

---

## 9. Field definitions — mx:* non-YAML identity fields

The following fields are the non-YAML equivalents of standard cog fields. They provide identity and classification metadata in carrier formats that do not use YAML frontmatter (HTML, JavaScript, CSS). All fields in this section use the `mx:` prefix in their respective carrier contexts.

### 9.1 `mx:name`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Cog name, equivalent to `title` in YAML. Provides a machine-readable identifier in carrier formats that lack YAML frontmatter.

**Example (HTML):**

```html
<meta name="mx:name" content="validation-utility">
```

**Example (JSDoc):**

```javascript
/** @mx:name validation-utility */
```

**Normative notes:**

- In YAML contexts, use `title` instead. `mx:name` is for non-YAML carriers only.

---

### 9.2 `mx:version`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Version string, equivalent to `version` in YAML.

**Example (HTML):**

```html
<meta name="mx:version" content="2.0">
```

**Example (CSS):**

```css
/* @mx:version 2.0 */
```

---

### 9.3 `mx:type`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

**Definition:** Domain type classification, equivalent to `contentType` in YAML.

**Example (HTML):**

```html
<meta name="mx:type" content="guide">
```

**Normative notes:**

- In HTML contexts, `mx:type` maps to the YAML `contentType` field.
- The shorter name `type` is used in HTML to follow web conventions.

---

### 9.4 `mx:purpose`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Why this file exists. Equivalent to `purpose` in YAML.

**Example (JSDoc):**

```javascript
/** @mx:purpose validation */
```

**Example (CSS):**

```css
/* @mx:purpose layout */
```

---

### 9.5 `mx:audience`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Intended readership. Equivalent to `audience` in YAML. In non-YAML contexts, array values are expressed as a comma-separated string.

**Example (JSDoc):**

```javascript
/** @mx:audience developers, agents */
```

---

### 9.6 `mx:stability`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |
| **Valid values** | stable, evolving, experimental, deprecated, archived |
| **Default** | *(none)* |

**Definition:** Stability level. Communicates how reliable this file's interface is for external consumers.

**Example (JSDoc):**

```javascript
/** @mx:stability stable */
```

**Normative notes:**

- `stable` — interface is fixed; breaking changes require a major version bump.
- `evolving` — interface may change; consumers should pin versions.
- `experimental` — interface is subject to significant change without notice.
- `deprecated` — interface is scheduled for removal; consumers should migrate.
- `archived` — interface is no longer maintained.

---

### 9.7 `mx:category`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Cog category. Equivalent to `category` in YAML. Determines registry grouping in non-YAML carrier formats.

**Example (HTML):**

```html
<meta name="mx:category" content="mx-tool">
```

---

### 9.8 `mx:tags`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | html, js, css |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Discovery keywords as a comma-separated string. Equivalent to `tags` (array) in YAML.

**Example (HTML):**

```html
<meta name="mx:tags" content="validation, metadata, utility">
```

**Normative notes:**

- Values MUST be comma-separated in non-YAML contexts.
- Values SHOULD be lowercase strings.

---

### 9.9 `mx:builds-on`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Context dependencies as a comma-separated list of cog names. Equivalent to `buildsOn` (array) in YAML.

**Example (JSDoc):**

```javascript
/** @mx:builds-on fields, cog-unified-spec */
```

**Normative notes:**

- Uses kebab-case (`builds-on`) in non-YAML contexts, following the context-specific naming rules in Section 7.
- Values MUST be comma-separated.

---

### 9.10 `mx:documented-in`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Path to companion documentation. Links a code file to its narrative documentation.

**Example (JSDoc):**

```javascript
/** @mx:documented-in ../docs/validation-guide.cog.md */
```

---

### 9.11 `mx:context-provides`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Profile** | js, css |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** What context this file provides to agents. Declares the knowledge or capability an agent gains by reading this file.

**Example (JSDoc):**

```javascript
/** @mx:context-provides Field validation rules and error message templates */
```

---

## 10. Field definitions — x-mx- public extension fields

The following fields use the `x-mx-` prefix and are owned by CogNovaMX. They are visible in published cogs and provide implementation-specific features that are not part of the open standard.

### 10.1 `x-mx-mount-type`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | x-mx-public |
| **Conformance** | MUST (Level 1) for x-mx-public profile |
| **Valid values** | personal, team, product, standard |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Categorises submodules in the hub mount table. Determines the governance and swapability characteristics of a mounted repository.

**Example:**

```yaml
mx:
  x-mx-mount-type: personal
```

**Normative notes:**

- `personal` — owned by an individual. Content is personal to the user.
- `team` — shared across a team. Content is collaborative.
- `product` — a product component. Content is part of a deliverable.
- `standard` — a forkable standard. Content follows community governance.
- This field is REQUIRED for all `.mx.yaml.md` folder metadata files that describe mount points.

---

### 10.2 `x-mx-mount-swappable`

| Property | Value |
|----------|-------|
| **Type** | string-or-boolean |
| **Zone** | 2 (mx:) |
| **Profile** | x-mx-public |
| **Conformance** | MUST (Level 1) for x-mx-public profile |
| **Valid values** | true, false, fork |
| **Default** | *(none — explicit declaration required)* |

**Definition:** Whether this mount can be swapped for a different implementation. Controls the replaceability of a mounted submodule.

**Example:**

```yaml
mx:
  x-mx-mount-swappable: true
```

**Normative notes:**

- `true` — the mount can be replaced with any compatible alternative.
- `false` — the mount is integral and MUST NOT be replaced.
- `fork` — the mount can be replaced only by forking and customising.

---

### 10.3 `x-mx-mount-upstream`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | x-mx-public |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

**Definition:** Upstream source for standard-type mounts. Identifies the canonical source repository for a forked or derived mount.

**Example:**

```yaml
mx:
  x-mx-mount-upstream: "https://github.com/example/upstream-repo"
```

**Normative notes:**

- This field is only meaningful when `x-mx-mount-type` is `standard`.
- The value SHOULD be a URL pointing to the upstream repository.

---

## 11. x-mx-p- private extension mechanism

### 11.1 Overview

The `x-mx-p-` prefix designates private extension fields. Unlike standard fields and `x-mx-` public extensions, private extension values are opaque — they are hashed, encoded, or tokenised so that only holders of the decode registry can interpret them.

### 11.2 Value encoding

Private extension values MUST be encoded before storage. The encoding method is at the vendor's discretion. Common approaches include:

- **Hash-based** — a one-way hash that maps to a lookup table in `$MX_HOME`
- **Token-based** — a short token (e.g., `a3f2c8`) that resolves to a full value via the registry
- **Encrypted** — symmetric or asymmetric encryption with keys stored in `$MX_HOME`

### 11.3 Decode registry

The decode registry lives in `$MX_HOME`. Implementations that encounter `x-mx-p-` fields without access to the decode registry MUST:

1. Preserve the field and its opaque value unchanged.
2. Not attempt to decode, interpret, or validate the value.
3. Pass the field through to any output format without modification.

### 11.4 Field listing

This standard does not list individual `x-mx-p-` fields. By definition, private extension fields are internal to CogNovaMX's implementation. Their names and semantics are documented in the `$MX_HOME` registry, not in public standards.

---

## 12. Extension registration process

### 12.1 How to add new extension fields

New extension fields are registered by CogNovaMX. The Gathering does not govern extension fields — vendor extensions are the vendor's decision.

### 12.2 Registration steps

1. **Choose the prefix.** Use `x-mx-` for public extensions, `x-mx-p-` for private extensions.
2. **Use kebab-case.** Extension field names MUST use kebab-case (unlike standard fields which use camelCase in YAML). Example: `x-mx-deploy-target`, not `x-mx-deployTarget`.
3. **Document the context.** Describe the field's purpose, type, valid values, and which carrier formats it applies to.
4. **No Gathering approval needed.** Extension fields follow vendor governance, not community governance.

### 12.3 Promotion to standard

An extension field MAY be promoted to a standard field through the following process:

1. The vendor proposes promotion to The Gathering.
2. The Gathering reviews the field's adoption, utility, and universality.
3. If accepted, the field's prefix is removed, and it becomes a standard field with camelCase naming.
4. The old `x-mx-` prefixed version SHOULD be maintained as an alias for one major version cycle.

---

## 13. Conformance levels summary

### 13.1 Namespace usage

| Requirement | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------------|:-:|:-:|:-:|
| Standard fields have no prefix | MUST | — | — |
| Extension fields use `x-mx-` or `x-mx-p-` | MUST | — | — |
| No prefix pollution | MUST | — | — |

### 13.2 Carrier format metadata

| Carrier | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|---------|:-:|:-:|:-:|
| `.cog.md` / `.md` (YAML frontmatter) | MUST | — | — |
| `.cog.html` / `.html` (meta tags) | — | MUST | — |
| `.cog.js` / `.js` (JSDoc) | — | MUST | — |
| `.cog.css` / `.css` (CSS comments) | — | MUST | — |
| Shell scripts (comment block) | — | MUST | — |
| `.mx.yaml` (media sidecar) | — | — | MAY |
| `mx.yaml` (repo root) | — | — | MAY |
| Image EXIF/XMP | — | — | MAY |
| Database sidecar / inline SQL | — | — | MAY |

### 13.3 mx:* non-YAML identity fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `mx:name` | — | SHOULD | — |
| `mx:version` | — | SHOULD | — |
| `mx:type` | — | SHOULD | — |
| `mx:purpose` | — | — | MAY |
| `mx:audience` | — | — | MAY |
| `mx:stability` | — | — | MAY |
| `mx:category` | — | — | MAY |
| `mx:tags` | — | — | MAY |
| `mx:builds-on` | — | — | MAY |
| `mx:documented-in` | — | — | MAY |
| `mx:context-provides` | — | — | MAY |

### 13.4 x-mx- public extension fields

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `x-mx-mount-type` | MUST | — | — |
| `x-mx-mount-swappable` | MUST | — | — |
| `x-mx-mount-upstream` | — | — | MAY |

---

## 14. Security and privacy considerations

- The `x-mx-p-` private extension mechanism is designed to protect sensitive metadata values. Implementations MUST NOT attempt to decode private values without the `$MX_HOME` registry.
- Extension fields MAY carry information about infrastructure, deployment targets, or organisational structure. Authors SHOULD consider whether public extension values expose sensitive operational details.
- The decode registry at `$MX_HOME` contains sensitive lookup tables. Access to `$MX_HOME` SHOULD be restricted to authorised users and agents.

---

## 15. References

### 15.1 Normative references

- [MX Core Metadata Standard](mxs-01-core-metadata.cog.md) — conformance level framework, Zone 1/Zone 2 field definitions
- [Cog Unified Specification](../specifications/cog-unified-spec.cog.md) — the cog document format and carrier format details
- [ADR-02: Namespace Policy](../architecture-decisions/adr-02-namespace-policy.cog.md) — architecture decision establishing the three-level namespace
- [NDR-02: camelCase Naming](../naming-decisions/ndr-02-camelcase-naming.cog.md) — field naming convention
- [NDR-03: Spelling Neutrality](../naming-decisions/ndr-03-spelling-neutrality.cog.md) — spelling-neutral vocabulary
- [MX Provenance Standard](mxs-03-provenance.cog.md) — trust, attribution, verification
- [MX AI/Agent Policy Standard](deferred/mxs-04-ai-agent-policy.cog.md) — AI governance and agent policies
- [MX Profile-Specific Metadata Standard](deferred/mxs-05-profile-metadata.cog.md) — content-type-specific fields

### 15.2 Informative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels
- [RFC 6648](https://www.rfc-editor.org/rfc/rfc6648) — Deprecating the "X-" prefix (historical context for the `x-mx-` convention)
- [HTTP Extension Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) — the `X-` prefix convention for non-standard headers
- [XMP Specification](https://www.adobe.com/devnet/xmp.html) — Extensible Metadata Platform for media files
- [Dublin Core DCMI Namespace](https://www.dublincore.org/specifications/dublin-core/dcmi-namespace/) — namespace governance model
- [MX Standards Alignment](../specifications/mx-standards-alignment.cog.md) — how MX conventions align with existing web standards

---

## 16. Change log

| Version | Date | Changes |
|---------|------|---------|
| 1.0-draft | 2026-04-02 | Initial draft. Namespace policy, carrier format mappings, and extension mechanisms based on ADR-02 namespace policy. |
