---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Extensions note"
docname: draft-cranstoun-mx-extensions
date: 2026-05-23
consensus: false
keyword:
  - mx
  - extensions
  - namespace
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-extensions.md
---

# MX Extensions note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note defines the namespace policy for the Machine Experience (MX) framework. It specifies how vendors extend the MX vocabulary without polluting the standard namespace, how field names adapt across syntactic contexts, and how new extension fields are registered.

The namespace policy establishes three levels: standard fields (no prefix, owned by The Gathering), public extensions (`x-mx-`, the public extension namespace operated by CogNovaMX during the seed phase; vendor sub-namespaces of the form `x-mx-{vendor}-*` are reserved for individual vendors), and private extensions (`x-mx-p-`, opaque-value extensions). The prefix itself is the policy: an implementation can determine governance from the field name alone, without external lookup.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field — heading, property table, definition prose, example — and is binding on this note. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

This note defines two conformance levels (modelled on the [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level A/AA pattern):

| Level | Name | Requirement |
|-------|------|-------------|
| Level 1 | **MX Core** | Valid namespace usage — standard fields have no prefix, extensions use `x-mx-` or `x-mx-p-`. No prefix pollution. |
| Level 2 | **MX Standard** | Adds Level 1 plus correct context-specific naming when fields appear outside YAML (HTML, JSDoc, CSS, XMP, shell, sidecar, SQL). |

A document claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions, conformance requirements, and normative rules in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Scope

### 3.1 In scope

This note specifies:

- **Namespace policy** — the three-level prefix hierarchy and governance rules
- **Context-specific naming** — how the same field appears in YAML, HTML, JSDoc, CSS, shell, XMP, sidecar, and SQL contexts
- **Public extension fields** — the `x-mx-` vendor extension vocabulary
- **Private extension mechanism** — the `x-mx-p-` obfuscated extension system
- **Extension registration process** — how to add new extension fields

### 3.2 Out of scope

- **Carrier format mechanics** — how each file format (HTML meta tags, JSDoc, CSS comments, EXIF/XMP, shell, sidecar, SQL) carries MX metadata is governed by a separate carrier-formats note.
- **Non-YAML identity fields** — the `mx:*` field vocabulary used by HTML, JavaScript, and CSS carriers (`mx:name`, `mx:version`, `mx:type`, etc.) is governed by the carrier-formats note.
- **Domain-specific extension fields** — fields belonging to a particular application domain (workflow contracts, content audits, observability) belong in their own notes, not this one.
- **Standard field definitions** — the meaning, type, and validation of standard MX fields (`title`, `author`, `created`, `description`, `status`, `contentType`) is governed by the MX Core Metadata note.

### 3.3 Relationship to existing standards

This note draws on the following published standards:

- **HTTP "X-" extension header convention** ([RFC 6648](https://www.rfc-editor.org/rfc/rfc6648) provides historical context) — the `x-` prefix convention for non-standard headers, adapted for metadata namespaces as `x-mx-`
- **Schema.org Style Guide** — vocabulary naming conventions; fields in YAML use camelCase per the Schema.org pattern
- **Dublin Core DCMI Namespace** — namespace governance model

---

## 4. Terminology

- **Namespace** — A prefix that identifies the owner and governance policy of a metadata field.
- **Standard field** — A field with no prefix, owned by The Gathering, universal across all MX implementations.
- **Extension field** — A field with an `x-mx-` or `x-mx-p-` prefix, owned by a vendor (currently CogNovaMX).
- **$MX_HOME** — The local MX OS registry directory. Contains decode tables for private extension values.

---

## 5. Design principles

The namespace system follows these principles:

1. **The prefix IS the policy.** No documentation lookup is needed to determine governance — `x-` means extension, `mx-` means CogNovaMX, `p-` means private.
2. **Standard fields are universal.** Every MX implementation uses the same core vocabulary with no prefix.
3. **Extensions are opt-in.** No implementation is required to support `x-mx-` or `x-mx-p-` fields.
4. **Context determines syntax, not semantics.** A field means the same thing regardless of whether it appears in YAML, HTML, or JSDoc.

---

## 6. Namespace policy

### 6.1 The three-level namespace

MX metadata fields are organised into three namespace levels. Each level has a distinct prefix, owner, and governance policy.

| Level | Prefix | Owner | Governance | Visibility |
|-------|--------|-------|------------|------------|
| Standard | *(none)* | The Gathering | Community-governed. Changes require Gathering approval. | Universal — all implementations use these fields. |
| Public extension | `x-mx-` | Public namespace, operated by CogNovaMX during the seed phase. Vendor sub-namespaces `x-mx-{vendor}-*` are reserved for individual vendors and allocated on request. | Vendor-governed within each sub-namespace. | Visible in published cogs. Implementation extension, not the open standard. |
| Private extension | `x-mx-p-` | Vendor-internal namespace. CogNovaMX during the seed phase; vendor sub-namespaces `x-mx-p-{vendor}-*` allocated as the public namespace is. | Vendor-governed. | Obfuscated. Only registry holders can decode values. |

### 6.2 Standard fields (no prefix)

Standard fields are the core MX vocabulary owned by The Gathering. They have no prefix.

Implementations MUST NOT add a prefix to standard fields. A field that is part of the core MX vocabulary MUST always appear without a prefix in all contexts.

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

Public extension fields use the `x-mx-` prefix. The namespace is the public extension space for the MX vocabulary; CogNovaMX operates it during the seed phase. Vendor sub-namespaces of the form `x-mx-{vendor}-*` are reserved for individual vendors and allocated on request — for example, a vendor named "Acme" would publish fields as `x-mx-acme-deploy-target`. Within a vendor sub-namespace the vendor governs the field set.

Public extension fields MUST use the `x-mx-` prefix. They MUST use kebab-case for the field name portion (unlike standard fields which use camelCase in YAML). They are visible in published cogs and public outputs; they are implementation-specific features, not part of the open standard.

**Example (CogNovaMX, the seed-phase operator):**

```yaml
mx:
  x-mx-mount-type: personal
  x-mx-mount-swappable: true
```

**Example (vendor sub-namespace):**

```yaml
mx:
  x-mx-acme-deploy-target: production
  x-mx-acme-cluster-id: cluster-east-1
```

### 6.4 Private extension fields (`x-mx-p-`)

Private extension fields use the `x-mx-p-` prefix. They are owned by CogNovaMX and their values are hashed, encoded, or tokenised.

- Private extension field values MUST be opaque to any system that does not hold the decode registry at `$MX_HOME`.
- Implementations MUST NOT attempt to interpret `x-mx-p-` field values without access to the decode registry. Unknown values MUST be passed through unchanged.

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

The same MX field appears in syntactically different contexts — YAML frontmatter, HTML meta tags, JavaScript documentation comments, CSS comments, shell scripts, XMP metadata, sidecar files, and SQL comments. Each context has its own naming convention. Implementations MUST apply the conversion consistently so that a field's identity is preserved across carriers.

### 7.1 Naming and conversion rules (Informative summary)

The table below summarises the per-carrier conventions. The **normative form** of each conversion — the binding rule a verifier checks — lives in the **MX Carrier Formats note**, which is the single source of truth for carrier-specific syntax. This summary is informative; if a row here disagrees with the carrier-formats note, the carrier-formats note wins.

| Context | Convention | Conversion from YAML camelCase | Example field | Example syntax |
|---------|-----------|--------------------------------|---------------|----------------|
| YAML | camelCase | *(canonical)* | `buildsOn` | `buildsOn: [cog-unified-spec]` |
| HTML | kebab-case with `mx:` prefix | camelCase → kebab-case, prepend `mx:` | `mx:content-type` | `<meta name="mx:content-type" content="guide">` |
| JSDoc | kebab-case with `@mx:` tag | camelCase → kebab-case, prepend `@mx:` | `@mx:runtime` | `@mx:runtime node` |
| CSS | kebab-case with `@mx:` comment | camelCase → kebab-case, prepend `@mx:` | `@mx:type` | `/* @mx:type utility */` |
| Shell | camelCase in `# key: value` | none — preserve camelCase | `contentType` | `# contentType: guide` |
| XMP | `mx:` namespace prefix, case preserved | prepend `mx:` | `mx:status` | `<mx:status>active</mx:status>` |
| Sidecar | camelCase in YAML | none — preserve camelCase | `contentType` | `contentType: guide` |
| SQL | camelCase in `-- @mx` block | none — preserve camelCase | `contentType` | `-- @mx contentType: guide` |

For HTML in particular: the four MUST-at-Level-2 fields from MX Core Metadata §7a (`canonicalUri`, `summary`, `conformsTo`, `trainingDataPolicy`) MUST be present in HTML carriers as the kebab-case `mx:` `<meta>` tags `mx:canonical-uri`, `mx:summary`, `mx:conforms-to`, `mx:training-data-policy`. The carrier-formats note defines the binding form and worked examples.

### 7.2 Extension field naming in non-YAML contexts

Extension fields (`x-mx-` and `x-mx-p-`) already use kebab-case. In non-YAML contexts, the prefix is preserved as-is:

| Context | Standard field example | Extension field example |
|---------|----------------------|------------------------|
| YAML | `contentType: guide` | `x-mx-mount-type: personal` |
| HTML | `<meta name="mx:content-type">` | `<meta name="mx:x-mx-mount-type">` |
| JSDoc | `@mx:content-type guide` | `@mx:x-mx-mount-type personal` |

---

## 8. Public extension fields (`x-mx-`)

The following fields use the `x-mx-` prefix and are owned by CogNovaMX. They are visible in published cogs and provide implementation-specific features that are not part of the open standard.

### 8.1 `x-mx-mount-type`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | x-mx-public |
| **Conformance** | MUST (Level 1) for x-mx-public profile |
| **Valid values** | personal, team, product, standard |

Categorises submodules in the hub mount table. Determines the governance and swapability characteristics of a mounted repository.

- `personal` — owned by an individual.
- `team` — shared across a team.
- `product` — a product component.
- `standard` — a forkable standard.

This field is REQUIRED for all `.mx.yaml.md` folder metadata files that describe mount points.

```yaml
mx:
  x-mx-mount-type: personal
```

### 8.2 `x-mx-mount-swappable`

| Property | Value |
|----------|-------|
| **Type** | string-or-boolean |
| **Zone** | 2 (mx:) |
| **Profile** | x-mx-public |
| **Conformance** | MUST (Level 1) for x-mx-public profile |
| **Valid values** | true, false, fork |

Whether this mount can be swapped for a different implementation.

- `true` — the mount can be replaced with any compatible alternative.
- `false` — the mount is integral and MUST NOT be replaced.
- `fork` — the mount can be replaced only by forking and customising.

```yaml
mx:
  x-mx-mount-swappable: true
```

### 8.3 `x-mx-mount-upstream`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | x-mx-public |
| **Conformance** | MAY (Level 2) |

Upstream source for standard-type mounts. Identifies the canonical source repository for a forked or derived mount. Only meaningful when `x-mx-mount-type` is `standard`. The value SHOULD be a URL pointing to the upstream repository.

```yaml
mx:
  x-mx-mount-upstream: "https://github.com/example/upstream-repo"
```

---

## 9. Private extension mechanism (`x-mx-p-`)

### 9.1 Overview

The `x-mx-p-` prefix designates private extension fields. Unlike standard fields and `x-mx-` public extensions, private extension values are opaque — they are hashed, encoded, or tokenised so that only holders of the decode registry can interpret them.

### 9.2 Value encoding

Private extension values MUST be encoded before storage. The encoding method is at the vendor's discretion. Common approaches include:

- **Hash-based** — a one-way hash that maps to a lookup table in `$MX_HOME`
- **Token-based** — a short token (e.g., `a3f2c8`) that resolves to a full value via the registry
- **Encrypted** — symmetric or asymmetric encryption with keys stored in `$MX_HOME`

### 9.3 Decode registry

The decode registry lives in `$MX_HOME`. Implementations that encounter `x-mx-p-` fields without access to the decode registry MUST:

1. Preserve the field and its opaque value unchanged.
2. Not attempt to decode, interpret, or validate the value.
3. Pass the field through to any output format without modification.

### 9.4 Field listing

This note does not list individual `x-mx-p-` fields. By definition, private extension fields are internal to CogNovaMX's implementation. Their names and semantics are documented in the `$MX_HOME` registry, not in public standards.

---

## 10. Extension registration process

### 10.1 How to add new extension fields

New extension fields are registered by CogNovaMX. The Gathering does not govern extension fields — vendor extensions are the vendor's call.

### 10.2 Registration steps

1. **Choose the prefix.** Use `x-mx-` for public extensions, `x-mx-p-` for private extensions.
2. **Use kebab-case.** Extension field names MUST use kebab-case (unlike standard fields which use camelCase in YAML). Example: `x-mx-deploy-target`, not `x-mx-deployTarget`.
3. **Document the context.** Describe the field's purpose, type, valid values, and which carrier formats it applies to.
4. **No Gathering approval needed.** Extension fields follow vendor governance, not community governance.

### 10.3 Promotion to the core vocabulary

An extension field MAY be promoted into the core MX vocabulary through the following process:

1. The vendor proposes promotion to The Gathering.
2. The Gathering reviews the field's adoption, utility, and universality.
3. If accepted, the field's prefix is removed, and it becomes a core field with camelCase naming.
4. The old `x-mx-` prefixed version SHOULD be maintained as an alias for one major version cycle.

### 10.4 Field deprecation and retirement (Normative)

A field — standard or extension — that turns out to be a bad idea, that fails to find adoption, or that is replaced by a better-named or better-shaped equivalent follows the announce → deprecate → retire lifecycle. The lifecycle is normative for fields owned by The Gathering; vendor sub-namespaces SHOULD adopt it but MAY follow their own retirement policy provided the policy is documented and the same one-major-version alias guarantee is offered.

**Stage 1 — Announce.** The owner publishes notice that the field will be deprecated. The notice MUST name:

- The field being deprecated.
- The replacement field, if any (or the rationale for retirement without replacement).
- The major version at which the field will be marked deprecated.
- The major version at which the field will be retired (typically one major version after deprecation).

The notice is a draft note, an entry in the field dictionary's deprecation log, or both. From the announcement onwards the canonical documentation for the field MUST link to the announcement.

**Stage 2 — Deprecate.** At the announced major version, the field is marked deprecated:

- The field's canonical documentation gains a `Deprecated:` banner naming the replacement.
- New documents SHOULD declare the replacement field; existing documents continue to validate.
- Tools SHOULD emit a one-line warning when they encounter the deprecated field, naming the replacement.
- The replacement field MUST treat the deprecated field as an alias and copy the value across when migrating documents.

A field MUST remain in the deprecated state for at least one major version cycle. This window is the migration runway: tooling, registries, downstream consumers, and authored documents have time to switch.

**Stage 3 — Retire.** At the announced retirement version, the field is removed:

- The field is removed from the canonical field dictionary and from the conformance summary tables.
- Validators MAY treat the retired field as a conformance failure (warning at minimum, error at the validator's discretion).
- The deprecation log keeps the field's entry permanently as historical record, with the retirement version annotated.

**Reverse of promotion.** A field that was promoted from extension space into the core vocabulary (§10.3) follows the same announce → deprecate → retire path if it is later retired. The `x-mx-`-prefixed alias maintained during promotion is itself retired in the same lifecycle event.

**Aliases as a migration tool.** When a field is renamed (the active case throughout the MX 1.0 ratification cycle — `author` → `originator`, `maintainer` / `ownership` → `stewardship`), the old name is treated as a deprecated alias of the new name from the deprecation announcement onward. The one-major-version alias guarantee gives every author and tool a mechanical migration path: update the field name at any point in the migration window, and validators on either side of the change will accept the document.

---

## 11. Conformance summary

| Requirement | Level 1 (MUST) | Level 2 (SHOULD/MAY) |
|-------------|:-:|:-:|
| Standard fields have no prefix | MUST | — |
| Extension fields use `x-mx-` or `x-mx-p-` | MUST | — |
| No prefix pollution | MUST | — |
| Correct context-specific naming for non-YAML carriers | — | SHOULD |
| `x-mx-mount-type` (when x-mx-public profile applies) | MUST | — |
| `x-mx-mount-swappable` (when x-mx-public profile applies) | MUST | — |
| `x-mx-mount-upstream` | — | MAY |

---

## 12. Security and privacy considerations

- The `x-mx-p-` private extension mechanism is designed to protect sensitive metadata values. Implementations MUST NOT attempt to decode private values without the `$MX_HOME` registry.
- Extension fields MAY carry information about infrastructure, deployment targets, or organisational structure. Authors SHOULD consider whether public extension values expose sensitive operational details.
- The decode registry at `$MX_HOME` contains sensitive lookup tables. Access to `$MX_HOME` SHOULD be restricted to authorised users and agents.

---

## 13. References

### 13.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels

### 13.2 Informative references

- [RFC 6648](https://www.rfc-editor.org/rfc/rfc6648) — Deprecating the "X-" prefix (historical context for the `x-mx-` convention)
- [HTTP Extension Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) — the `X-` prefix convention for non-standard headers
- [Schema.org Style Guide](https://schema.org/docs/styleguide.html) — Vocabulary naming conventions
- [Dublin Core DCMI Namespace](https://www.dublincore.org/specifications/dublin-core/dcmi-namespace/) — namespace governance model
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (conformance level model)

---

## 14. Common authoring mistakes (Informative)

Five shapes a fresh author often produces, each with the fix.

**Mistake 1: prefixing a standard field.**

```yaml
# WRONG — status is a standard field; no prefix
mx:
  x-mx-status: active
```

Standard fields (§6.2) MUST never carry the `x-mx-` prefix. Drop the prefix:

```yaml
# RIGHT
mx:
  status: active
```

**Mistake 2: camelCase on an extension field.**

```yaml
# WRONG — extension fields use kebab-case in YAML, not camelCase
mx:
  x-mx-deployTarget: production
```

Extension fields (§6.3, §10.2) use kebab-case for the suffix:

```yaml
# RIGHT
mx:
  x-mx-deploy-target: production
```

**Mistake 3: claiming a vendor sub-namespace without permission.**

```yaml
# WRONG — `acme` sub-namespace not allocated to this author
mx:
  x-mx-acme-cluster-id: "prod-1"
```

The vendor sub-namespace pattern `x-mx-{vendor}-*` (§6.3) is reserved. Only use a vendor's sub-namespace if the namespace has been allocated to that vendor. For one-off fields outside an allocated sub-namespace, the seed-phase `x-mx-` namespace operated by CogNovaMX is the default until a sub-namespace is allocated.

**Mistake 4: trying to decode an `x-mx-p-` value.**

```yaml
# WRONG — implementations without the registry MUST NOT attempt to interpret
mx:
  x-mx-p-client-tier: "a3f2c8"
  # ... and downstream code does:
  # if (clientTier === "a3f2c8") { ... }
```

Private extension values (§9) are opaque. Implementations without access to the decode registry MUST pass the value through unchanged and MUST NOT branch on its content. The opaque token is meaningful only to a registry-equipped consumer.

**Mistake 5: introducing `x-mx-` for something the standard already covers.**

```yaml
# WRONG — modified is already a standard field
mx:
  x-mx-last-changed: 2026-05-07
```

Before declaring an `x-mx-` extension, check the standard vocabulary in MX Core Metadata §5–6. The intent here is captured by `modified` (Zone 1, §5.5). Use the standard field; the extension namespace is for genuinely new vocabulary, not synonyms of existing fields.

---

*End of MX Extensions note draft.*
