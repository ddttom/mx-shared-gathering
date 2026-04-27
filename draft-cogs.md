---
title: "MX Cogs note"
docname: draft-cranstoun-mx-cogs
date: 2026-04-27
consensus: false
keyword:
  - mx
  - cogs
  - optional-layer
  - specification
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Cogs note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

MX Core defines machine-readable document metadata. **Cogs are an optional layer on top of MX.** A cog is a `.cog.md` file: a markdown document with structured YAML frontmatter that, in addition to the MX core fields it carries, declares additional fields describing its place in a registry, its dependencies, the typed content blocks in its body, an optional execution contract, and how it identifies itself to unfamiliar consumers.

A document does not need to be a cog to carry MX metadata. A blog post, a sidecar YAML, or an HTML page can declare title, author, dates, audience, license without ever being a cog. Cogs are the vocabulary you reach for when you want a document to be **navigable**, **composable**, and **runnable** by agents and runtimes — not the floor for being MX-aware.

This note specifies:

- The cog file format (`.cog.md`).
- The magic-header HTML comment used for byte-zero self-identification.
- The `cogHeader` frontmatter field that mirrors the magic-header comment for YAML-only consumers.
- The cog structural fields (`partOf`, `buildsOn`, `requires`, `refersTo`, plus the cog-specific extensions for blocks, execute contracts, includes, deliverables, classification, and identifiers — many of which currently live in vendor extensions until ratified into the standard tier).

**Out of scope:**

- **Signing.** A cog can be signed but does not have to be; signing is governed by the MX Contract Fingerprinting and Signing draft note.
- **Non-YAML carriers.** This note specifies the `.cog.md` markdown form only. How an MX-aware HTML page, JavaScript file, CSS file, or sidecar carries identity and operational metadata is governed by the MX Carrier Formats note.
- **Document identity field semantics.** What `title`, `description`, `author`, `created`, `modified`, and `version` mean is governed by the MX Core Metadata note. Cogs inherit these definitions; this note does not redefine them.
- **Namespace policy.** The `x-mx-` and `x-mx-p-` prefix conventions used by some cog fields are governed by the MX Extensions note.

The two-zone YAML frontmatter model is carried over from MX Core: Zone 1 (top-level) holds document identity fields; Zone 2 (under the `mx:` object) holds operational fields. Cog structural fields live in Zone 2 unless explicitly noted as top-level (`cogHeader`, `contractFields`, `metadataFields`).

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) when, and only when, they appear in all capitals.

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field — heading, property table, definition prose, example — and is binding on this note. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

This note defines three conformance levels for cog documents (modelled on the [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level A/AA/AAA pattern). Each level assumes the document already satisfies the matching MX Core level for document identity.

| Level | Name | Cog requirement |
|-------|------|-----------------|
| Level 1 | **MX Cog Core** | The document is a `.cog.md` file. `partOf` declared. |
| Level 2 | **MX Cog Standard** | Adds `buildsOn` (or an empty array) and either a magic-header comment or a `cogHeader` field. |
| Level 3 | **MX Cog Complete** | Adds explicit `requires`, `refersTo`, plus the relevant extensions (block declarations, execute contract, deliverables, identifiers) for the cog's role. |

A cog claiming conformance at a given level MUST satisfy all requirements at that level and all lower levels.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions and conformance rules are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Terminology

- **Cog** — A `.cog.md` file. The atomic unit of the cog layer: a markdown document with YAML frontmatter and optionally typed content blocks (prose, action, definition, code, html, sop, etc.) within the body.
- **Optional layer** — A capability some MX-aware documents adopt and others do not. A document that does not adopt the cog layer is still a valid MX document.
- **Magic-header comment** — A specially formatted HTML comment placed at byte zero of a cog file, used for unambiguous file-level recognition by consumers that have not parsed the YAML.
- **Profile** — A named set of fields applicable to a specific document type. The cog layer defines the `cog` profile.

---

## 4. The magic-header comment

A cog file SHOULD identify itself as a cog at byte zero. The conventional mechanism is a magic-header HTML comment:

```html
<!-- cog v1 spec=https://example.org/cog-spec.v1.md runtime=https://example.org/cog-runtime.md -->
```

The comment is invisible in rendered Markdown but immediately visible to any agent reading the raw text. It serves byte-zero self-identification — a consumer that has not yet parsed YAML can still recognise the file as a cog.

The comment is a single line. Tokens in order:

1. The literal `cog`.
2. A version token of the form `v\d+(\.\d+)*` (`v1`, `v1.2`, `v2`, ...) — the cog spec version the document claims.
3. Zero or more `key=value` pairs separated by spaces. Conventional keys: `spec` (URL where the spec is published), `runtime` (URL where a runtime implementation lives), `runtime-doc` (URL pointing to runtime documentation).

The comment is invisible to YAML-only consumers (registries, validators, graph builders that parse frontmatter and discard comments). For those consumers, the same information must be carried in the `cogHeader` frontmatter field (§5).

---

## 5. `cogHeader` — the frontmatter equivalent

A cog MAY declare `cogHeader` instead of, or in addition to, the magic-header comment. Both forms carry the same information; either suffices for cog identification. Implementations SHOULD prefer `cogHeader` for programmatic consumption and the magic-header comment for byte-zero file recognition.

### 5.1 `cogHeader`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 1 (top-level) |
| **Conformance** | MAY (Level 3); MUST be present if neither this field nor a magic-header comment is declared on a cog circulating outside a closed system (Level 2 SHOULD-or-magic-header rule). |
| **Default** | *(none)* |

**Definition:** Object carrying the same information as the cog magic-header comment. Sub-keys:

| Sub-key | Type | Required when present | Definition |
|---------|------|-----------------------|-----------|
| `version` | string | yes | Cog spec version the document claims. Format: `v` followed by an integer or a dotted version string (`v1`, `v1.1`, `v2`). Same token as the magic header's version slot. |
| `spec` | string (URL) | yes | URL where the cog specification is published. SHOULD be HTTPS. |
| `runtime` | string (URL) | optional | URL where a runtime implementation can be found, downloaded, or invoked. |
| `runtimeDoc` | string (URL) | optional | URL pointing to runtime documentation explaining how to consume cogs. |

**Example:**

```yaml
cogHeader:
  version: v1
  spec: https://example.org/cog-spec.v1.md
  runtime: https://example.org/cog-runtime.md
  runtimeDoc: https://example.org/cog-runtime.md
```

- A cog declaring `cogHeader` MUST populate `version` and `spec`.
- `runtime` and `runtimeDoc` are optional; consumers MUST NOT treat their absence as a conformance failure.
- All URL-typed sub-keys SHOULD be HTTPS.
- Implementations SHOULD treat `cogHeader` values as informational metadata (excluded from any contract fingerprint applied to the cog) — these values describe identity, not contract.

### 5.2 The equivalence rule

When a cog includes both a magic-header comment AND a `cogHeader` field, the two forms MUST agree on every key they both declare:

- The magic header's version token MUST equal `cogHeader.version`.
- The magic header's `spec=` value MUST equal `cogHeader.spec`.
- The magic header's `runtime=` value MUST equal `cogHeader.runtime` when both are present.
- The magic header's `runtime-doc=` value MUST equal `cogHeader.runtimeDoc` when both are present.

A key present in only one form is permitted; a key present in both with mismatched values is a conformance failure.

### 5.3 Implementer guidance

Implementations SHOULD:

- Prefer `cogHeader` for programmatic consumption — it is queryable, robust to comment-stripping parsers, and addressable by registry tooling.
- Prefer the magic-header comment for byte-zero self-identification — it is visible to agents that have not yet parsed YAML.
- Emit both forms when generating cogs intended for circulation, so consumers of either kind can recognise the file without round-trip parsing.

---

## 6. Cog structural fields

The following fields define a cog's place in a registry, its dependencies, and its operational contract. All cog structural fields reside in Zone 2 (the `mx:` object) unless explicitly noted as top-level.

### 6.1 `partOf`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST (Level 1) for cogs |

**Definition:** Parent collection, suite, or initiative the cog belongs to. Names a registry namespace, a series, or a parent artefact.

**Example:**

```yaml
mx:
  partOf: mx-the-gathering
```

### 6.2 `buildsOn`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) for cogs |

**Definition:** Context graph. Array of cog names this document builds upon. Soft dependency — provides context, not a hard requirement.

**Example:**

```yaml
mx:
  buildsOn: [what-is-a-cog, cog-unified-spec]
```

- `buildsOn` forms the cog knowledge graph. The referenced cogs provide context but are not required for this cog to function.
- Distinct from `requires` (hard dependency) and `refersTo` (informational link).

### 6.3 `requires`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

**Definition:** Hard dependencies. Array of cog names that MUST exist for this cog to function.

**Example:**

```yaml
mx:
  requires: [node-runtime, markdownlint-cli2]
```

- A cog MUST NOT be considered functional if any of its `requires` entries are missing.
- Distinct from `buildsOn` (soft context) and `refersTo` (informational link).

### 6.4 `refersTo`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Level 3) |

**Definition:** Related cogs or external resources. Informational links, not dependencies.

**Example:**

```yaml
mx:
  refersTo: [field-dictionary, cog-unified-spec]
```

### 6.5 Identifier and classification fields

The following fields classify a cog within its registry. They currently live in vendor-extension space (`x-mx-*`) until ratified into the standard cog tier; the field names without prefix are reserved here and SHOULD be implemented under the `x-<vendor>-` convention until then.

| Field | Type | Zone | Purpose |
|-------|------|:----:|---------|
| `cogId` | string | 2 | Unique cog identifier within the registry. Typically derived from the filename without `.cog.md`. MUST be unique within its `partOf` namespace. |
| `cogType` | string | 2 | Cog classification. Common values: `info` (reference / documentation), `action` (carries an execute contract), `routing` (agent navigation), `certificate-of-genuineness` (publisher-provenanced credential). |
| `category` | string | 2 | Registry grouping (e.g. `mx-core`, `mx-tool`, `mx-content`). Coarser than `cogType`. |

### 6.6 Composition and content-shape fields

The following fields describe the cog's body content and how it composes with other cogs. They likewise live in vendor-extension space until ratified.

| Field | Type | Zone | Purpose |
|-------|------|:----:|---------|
| `blocks` | array | 2 | Names the typed content blocks present in the document body (e.g. `prose`, `action`, `definition`, `essence`, `provenance`, `version`, `code`, `html`, `sop`, `security`). The `prose` block is implicit and need not be declared. |
| `includes` | array | 2 | Cog composition declarations. Each entry names a source cog, an optional block filter, and a resolution mode (`build` resolves at build time and inlines content; `runtime` resolves on each request). |
| `execute` | object | 2 | Action-doc execution contract. Carries `runtime`, `command`, optional `actions[]`, optional `policy`. The presence of an `execute` field distinguishes action cogs from info cogs. |
| `policy` | string-or-object | 2 | Content-handling rules for agents. Inheritable from a parent uber-doc. |
| `deliverable` | string-or-array | 2 | What this cog produces or delivers — a report, a validated artefact, a published page. For action cogs, describes what running the actions yields; for info cogs, describes the knowledge or artefact the cog represents. |

---

## 7. Conformance summary

| Field | Level 1 (MUST) | Level 2 (SHOULD) | Level 3 (MAY) |
|-------|:-:|:-:|:-:|
| `partOf` | MUST | — | — |
| `buildsOn` | — | SHOULD | — |
| `cogHeader` *or* magic-header comment | — | SHOULD (when circulating) | — |
| `requires` | — | — | MAY |
| `refersTo` | — | — | MAY |
| `cogId`, `cogType`, `category` | — | — | MAY |
| `blocks`, `includes`, `execute`, `policy`, `deliverable` | — | — | MAY |

A cog at Level 1 is registry-locatable. At Level 2 it is identifiable to unfamiliar consumers and provides its context graph. At Level 3 it is fully described — dependencies, classifications, body content shape, and any execution contract.

---

## 8. Security and privacy considerations

- URLs declared in `cogHeader.runtime` may leak deployment topology. A cog declaring an internal runtime URL is exposing that URL to any reader of the cog. Operators concerned with topology privacy SHOULD either publish a public runtime URL or omit the field.
- A `cogHeader.spec` URL pointing at an attacker-controlled host is a phishing vector for runtimes that auto-fetch the spec. Runtimes SHOULD validate the spec URL against a trusted registry or operator-allowlist before fetching.
- The magic-header comment and `cogHeader` field are NOT authenticated — anyone who can write the cog can lie about which spec it claims to follow. Authentication, if required, is the responsibility of an external signing or witness mechanism, not this note.
- Cogs declaring `execute` contracts give agents a runnable surface. Operators MUST treat `execute.command` and `execute.actions[]` values as untrusted input and apply the same sandboxing, allowlisting, and audit measures they apply to any user-supplied command.
- A signed cog only attests provenance and integrity, never the truth of the content. Even when a cog's signature verifies cleanly, downstream consumers MUST NOT treat the values as factually correct — the signature confirms only that the cog genuinely came from the named signer and has not been altered. Editorial truth-curation is a different problem and out of scope for the cog format. User-facing language SHOULD say *attested*, not *verified*, to avoid implying a truth claim no signer makes.

---

## 9. References

### 9.1 Normative references

- [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119), [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) — keywords for use in RFCs to indicate requirement levels

### 9.2 Informative references

- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — conformance level model that inspired the Level 1/2/3 framework in §2.1

---

*End of MX Cogs note draft.*
