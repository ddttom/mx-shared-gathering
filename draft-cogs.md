---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Cogs note"
docname: draft-cranstoun-mx-cogs
date: 2026-05-06
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
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-cogs.md
---

# MX Cogs note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

MX Core defines machine-readable document metadata. **Cogs (Community Owned Governance Standards) are an optional layer on top of MX.** A cog is a `.cog.md` file: a markdown document with structured YAML frontmatter that, in addition to the MX core fields it carries, declares additional fields describing its place in a registry, its dependencies, the typed content blocks in its body, an optional execution contract, and how it identifies itself to unfamiliar consumers.

A document does not need to be a cog to carry MX metadata. A blog post, a sidecar YAML, or an HTML page can declare title, author, dates, audience, license without ever being a cog. Cogs are the vocabulary you reach for when you want a document to be **navigable**, **composable**, and **runnable** by agents and runtimes — not the floor for being MX-aware.

This note specifies:

- The cog file format (`.cog.md`).
- The magic-header YAML comment line carried inside the frontmatter for raw-text self-identification.
- The HTML carrier form of the magic-header: an `mx:cog` `<meta>` tag in the document's `<head>` (§4.1).
- The `cogHeader` frontmatter field that mirrors the magic-header comment for consumers that work on parsed data structures rather than raw text.
- The cog structural fields (`partOf`, `buildsOn`, `dependencies`, `refersTo`, plus the cog-specific extensions for blocks, execute contracts, includes, deliverables, classification, and identifiers — some of which are carried in vendor-extension space and may be promoted into the standard tier by The Gathering).

**Out of scope:**

- **Signing.** A cog can be signed but does not have to be; signing is governed by the MX Contract Fingerprinting and Signing draft note.
- **Non-YAML carriers.** This note specifies the `.cog.md` markdown form only. How an MX-aware HTML page, JavaScript file, CSS file, or sidecar carries identity and operational metadata is governed by the MX Carrier Formats note.
- **Document identity field semantics.** What `title`, `description`, `author`, `created`, `modified`, and `version` mean is governed by the MX Core Metadata note. Cogs inherit these definitions; this note does not redefine them.
- **Namespace policy.** The `x-mx-` and `x-mx-p-` prefix conventions used by some cog fields are governed by the MX Extensions note.

The three-zone YAML frontmatter model is carried over from MX Core: Zone 1a (top-level) holds document identity fields; Zone 1b (top-level) holds OKF-reserved surface fields (`type`, `tags`, `resource`); Zone 2 (under the `mx:` object) holds MX-operational fields. Cog structural fields live in Zone 2 unless explicitly noted as top-level (`cogHeader`, `contractFields`, `metadataFields`). An OKF consumer reading a cog file treats the entire `mx:` block as one unknown preserved key — this is the dual-conformance property that makes a cog simultaneously valid OKF and full MX.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) when, and only when, they appear in all capitals.

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field — heading, property table, definition prose, example — and is binding on this note. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance tiers

This note defines three conformance tiers for cog documents (modelled on the [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level A/AA/AAA pattern). The tiers are deliberately labelled **A / B / C** rather than Level 1/2/3 to avoid collision with the MX Core ladder defined in the MX Core Metadata note §2.1, which also uses Level 1/2/3 to grade the document-identity floor. Each cog tier assumes the document already satisfies the matching MX Core level for document identity; the two ladders apply simultaneously.

| Tier | Cog requirement |
|------|-----------------|
| **Tier A** | The document is a `.cog.md` file. `partOf` declared. |
| **Tier B** | Adds `buildsOn` (or an empty array) and either a magic-header comment or a `cogHeader` field. |
| **Tier C** | Adds explicit `dependencies`, `refersTo`, plus the relevant extensions (block declarations, execute contract, deliverables, identifiers) for the cog's role. |

A cog claiming conformance at a given tier MUST satisfy all requirements at that tier and all lower tiers.

#### 2.1.1 Reconciliation with the MX Core ladder

A cog is graded by **two** ladders at once: the MX Core Metadata note's Level 1/2/3 ladder (the document-identity floor that applies to every MX document) and this note's Tier A/B/C ladder (the cog-layer fields). A complete claim names both — for example, *"this cog is MX Core Level 3 + Cog Tier C"*. The combinations have the following meanings:

| MX Core | Cog Tier | What this cog is |
|---------|----------|------------------|
| Level 1 | Tier A | Minimal cog: identifies as `.cog.md` and carries the MUST identity fields. |
| Level 2 | Tier B | Discoverable: the magic-header / `cogHeader` lets unfamiliar consumers self-orient, and the SHOULD identity fields are present. |
| Level 3 | Tier C | Fully wired: dependencies, refers-to, classification, and all applicable identity fields are declared. |

Lower-numbered MX Core levels can pair with higher cog tiers, and vice versa, but a Tier C cog at MX Core Level 1 is unusual — most fully-wired cogs also carry the Level 2 or Level 3 identity floor.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions and conformance rules are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Terminology

- **Cog** — A `.cog.md` file. The atomic unit of the cog layer: a markdown document with YAML frontmatter and optionally typed content blocks (prose, action, definition, code, html, sop, etc.) within the body.
- **Optional layer** — A capability some MX-aware documents adopt and others do not. A document that does not adopt the cog layer is still a valid MX document.
- **Magic-header comment** — A specially formatted YAML comment line placed inside a cog file's YAML frontmatter (RECOMMENDED: the line immediately after the opening `---`), used for unambiguous file-level recognition by consumers scanning the frontmatter. In HTML carriers, the same identification is carried by an `mx:cog` `<meta>` tag in the document's `<head>` (§4.1).
- **Profile** — A named set of fields applicable to a specific document type. The cog layer defines the `cog` profile.

---

## 4. The magic-header comment

A cog file SHOULD identify itself as a cog by carrying a magic-header comment inside its YAML frontmatter, written as a YAML line comment:

```yaml
---
# cog v1 spec=https://example.org/cog-spec.v1.md runtime=https://example.org/cog-runtime.md
title: "..."
```

A YAML comment inside the frontmatter is visible to any agent reading the raw text and is preserved by YAML parsers as a token, so consumers that scan raw bytes and consumers that tokenise frontmatter both see the same identification line. Every YAML frontmatter parser (pandoc, gray-matter, kramdown-rfc) requires the opening `---` to be the first content in the file; the magic-header line placed immediately after that delimiter keeps the parser happy and the marker visible at the top of the file.

The line is a single YAML comment. Tokens in order, after the leading `#`:

1. The literal `cog`.
2. A version token of the form `v\d+(\.\d+)*` (`v1`, `v1.2`, `v2`, ...) — the cog spec version the document claims.
3. Zero or more `key=value` pairs separated by spaces. Conventional keys: `spec` (URL where the spec is published), `runtime` (URL where a runtime implementation lives), `runtime-doc` (URL pointing to runtime documentation).

The line MAY appear at any position within the frontmatter; placement on the line immediately after the opening `---` is RECOMMENDED so the identification is the first content a reader sees.

### 4.1 HTML carrier form

In HTML carriers, the same identification is carried by a single `mx:cog` meta tag in the document's `<head>`:

```html
<meta name="mx:cog" content="v1 spec=https://example.org/cog-spec.v1.md runtime=https://example.org/cog-runtime.md">
```

The `name` attribute is the fixed string `mx:cog` (the `mx:` prefix is the MX namespace per the **MX Extensions note**; `cog` is the marker). The `content` attribute follows the same token grammar as the YAML comment after the leading `# cog` token: a version token (`v\d+(\.\d+)*`) followed by zero or more space-separated `key=value` pairs. The HTML meta tag MAY appear anywhere inside `<head>`; placement near the top of `<head>` is RECOMMENDED so the identification is the first content a reader sees.

The YAML comment form, the HTML meta-tag form, and the `cogHeader` frontmatter field (§5) carry identical information; each is the canonical form for its carrier. A document with multiple carriers (an HTML page generated from a Markdown source, a PDF with XMP metadata mirroring its source) SHOULD carry the appropriate form in every carrier where the content circulates.

### 4.2 Preserving the source YAML frontmatter in HTML

An HTML carrier MAY include the source markdown's YAML frontmatter verbatim as an HTML comment, typically placed inside `<head>` immediately after the `mx:cog` meta tag (§4.1). This preserves the full structured metadata for consumers that prefer to parse the original YAML rather than reconstruct it from individual `<meta>` tags:

```html
<head>
  <meta name="mx:cog" content="v1 spec=https://example.org/cog-spec.v1.md runtime=https://example.org/cog-runtime.md">
  <!--
  ---
  # cog v1 spec=https://example.org/cog-spec.v1.md runtime=https://example.org/cog-runtime.md
  title: "Example document"
  description: "..."
  author: "..."
  created: 2026-01-01
  modified: 2026-01-01
  type: info-doc
  mx:
    cogUrn: cog:web:example.com:01JC8X7K2N4P5Q6R7S8T9V0W1Y
    partOf: example-collection
  ---
  -->
  ...
</head>
```

The convention is useful when the HTML is generated from a markdown source and downstream tools (registries, validators, search indexers) prefer to consume the original YAML rather than reconstruct it field-by-field from `<meta>` tags. The HTML comment MUST contain the verbatim source frontmatter including the surrounding `---` delimiters; consumers parsing it use the same YAML grammar as for the source markdown file.

This carrier form is permissive (MAY), not required. The `mx:cog` meta tag (§4.1) is the canonical cog identification for HTML carriers; this convention is an additional metadata-preservation aid for implementations that generate HTML from a markdown source.

### 4.3 Consumers that operate on parsed YAML

For consumers that operate purely on parsed YAML data (registries, validators, graph builders that discard comments during parsing), the same information is also available through the `cogHeader` frontmatter field (§5).

### 4.4 Design rationale

Each carrier form earns its place from a specific reader's vantage point. The full reasoning behind the choice is informative, not normative; this sub-section records it so future implementers do not have to re-derive it.

**Why the YAML comment lives inside the frontmatter.** Pandoc, gray-matter, and kramdown-rfc all require the opening `---` of the YAML frontmatter to be the first thing in the file. Placing the magic-header as a YAML comment line inside the frontmatter preserves every parser the ecosystem depends on while still giving raw-byte scanners a marker in the first few hundred bytes of the file. The YAML lexer treats the comment as a token, so consumers that tokenise the frontmatter still see the line; consumers that scan raw bytes see it inline at the top of the file. One form serves both readerships without forcing either to compromise.

**Why the HTML form is an `mx:cog` `<meta>` tag.** HTML has a native metadata mechanism (`<meta>` in `<head>`) that every HTML consumer already understands. Cog identification in HTML therefore reuses the convention rather than inventing a new container: `<meta name="mx:cog" content="...">` mirrors the existing `<meta name="mx:fieldName" content="value">` pattern that every MX-aware HTML page uses for its other operational metadata. The `mx:` prefix carries the namespace and lets a consumer filter MX metadata from non-MX metadata in a single regex. The `content` value follows the same token grammar as the YAML comment (a version token, then `key=value` pairs) so an implementation that parses one form parses both with the same lexer.

**Why the preserved-YAML HTML comment is permissive and complementary.** HTML's `<meta>` tags are flat key-value pairs and do not represent nested YAML structures cleanly. A `mx:` block with sub-objects (`policyRef`, `model`, `humanInLoop`, `clauses`) requires either a flattened per-field naming convention (verbose, error-prone) or a structured value encoded into the `content` attribute (parsing burden on every consumer). For HTML generated from a Markdown source, the simpler answer is to keep the original YAML available verbatim in an HTML comment alongside the per-field `<meta>` tags. Tools that prefer the structured source read the comment; tools that need only specific fields read the meta tags; the two views are consistent because both are derived from the same source. The convention is MAY rather than MUST because not every HTML carrier has a Markdown source to preserve.

---

## 5. `cogHeader` — the frontmatter equivalent

A cog MAY declare `cogHeader` instead of, or in addition to, the magic-header comment. Both forms carry the same information; either suffices for cog identification. Implementations SHOULD prefer `cogHeader` for programmatic consumption (registries, validators) and the magic-header comment for raw-text file recognition by agents that have not yet parsed YAML.

### 5.1 `cogHeader`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 1 (top-level) |
| **Conformance** | MAY (Tier C); MUST be present if neither this field nor a magic-header comment is declared on a cog circulating outside a closed system (Tier B SHOULD-or-magic-header rule). |
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
- Prefer the magic-header comment for raw-text self-identification — it sits at the top of the frontmatter and is visible to agents that scan bytes before parsing YAML structurally.
- Emit both forms when generating cogs intended for circulation, so consumers of either kind can recognise the file without round-trip parsing.

---

## 6. Cog structural fields

The following fields define a cog's place in a registry, its dependencies, and its operational contract. All cog structural fields reside in Zone 2 (the `mx:` object) unless explicitly noted as top-level.

### Declaring dependencies — explicit when applicable, standalone otherwise

A cog that stands on another cog — by being part of it, building on it, requiring it, inheriting from it, including it, or referring to it — MUST declare the relationship in the frontmatter using the appropriate relationship field (`partOf`, `buildsOn`, `dependencies`, `refersTo`, `inherits`, plus any include-style field a profile declares). Inheritance and inclusion count as dependencies for this purpose: any field that names another cog the current cog stands on carries the rule.

A **standalone cog** is one that names no other cog. It declares `dependencies: []` to make the absence explicit (the visible standalone marker) and carries no other relationship fields. The standalone case is the explicit converse of the MUST rule, not a separate category: every cog either lists what it depends on, or declares that it depends on nothing.

The rule applies to dependencies the cog itself relies on. Cogs the current cog publishes about, comments on, or supersedes are not dependencies and belong in their own fields (`supersedes`, `replaces`, vendor-namespaced extensions). The relationship fields named above are the canonical declarations a runtime, a registry, or a reader reads to know what context the cog needs to make sense.

### 6.1 `partOf`

| Property | Value |
|----------|-------|
| **Type** | array of strings (a bare string is accepted as a single-element array) |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST (Tier A) for cogs |

**Definition:** Parent collection(s), suite(s), or initiative(s) the cog belongs to. Each value names a registry namespace, a series, or a parent artefact. Because it is an array, a cog can belong to more than one set at once - so a query for the members of any one set returns every cog that declares it, even when the cog also belongs to others.

**Example:**

```yaml
mx:
  partOf: [mx-the-gathering, mx-audit]
```

A single membership may be written as a bare string (`partOf: mx-the-gathering`); a conforming reader treats it as a one-element array.

### 6.2 `buildsOn`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Tier B) for cogs |

**Definition:** Context graph. Array of cog names this document builds upon. Soft dependency — provides context, not a hard requirement.

**Example:**

```yaml
mx:
  buildsOn: [what-is-a-cog, cog-unified-spec]
```

- `buildsOn` forms the cog knowledge graph. The referenced cogs provide context but are not required for this cog to function.
- Distinct from `dependencies` (hard dependency) and `refersTo` (informational link).

### 6.3 `dependencies`

| Property | Value |
|----------|-------|
| **Type** | array of objects |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Tier C) |

**Definition:** Hard dependencies. An array of objects, each describing one thing this cog depends upon. Each entry has at minimum a `name` sub-key; optional sub-keys are `version` (a version constraint such as a SemVer range), `reason` (a one-sentence explanation of why the dependency is needed), and `kind` (one of `cog` | `runtime` | `package` | `external`; defaults to `cog`).

**Example:**

```yaml
mx:
  dependencies:
    - name: schema
      kind: cog
      reason: Validator depends on the schema cog's contract.
    - name: node
      kind: runtime
      version: '>=18'
    - name: markdownlint-cli2
      kind: package
```

- A cog MUST NOT be considered functional if any of its `dependencies` entries are missing or unresolvable for their declared `kind`.
- The recommended `kind` is `cog`. Runtime, package, and external deps SHOULD usually live in their proper carrier (`package.json`, the cog's runbook, an external manifest); the non-cog kinds in this field are escape hatches for cases where inline declaration is genuinely the right home.
- A cog with no dependencies MUST declare `dependencies: []` to make the standalone state visible — see the *Declaring dependencies* rule at the head of §6. Omitting the field is the older form and is being retired.
- Distinct from `buildsOn` (soft context) and `refersTo` (informational link).

### 6.4 `refersTo`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY (Tier C) |

**Definition:** Related cogs or external resources. Informational links, not dependencies.

**Example:**

```yaml
mx:
  refersTo: [field-dictionary, cog-unified-spec]
```

### 6.5 Identifier and classification fields

The following fields classify a cog within its registry. They live in vendor-extension space (`x-mx-*`); the field names without prefix are reserved here and SHOULD be implemented under the `x-<vendor>-` convention. The Gathering may promote any of them into the standard cog tier.

| Field | Type | Zone | Purpose |
|-------|------|:----:|---------|
| `cogId` | string | 2 | Unique cog identifier within the registry. Typically derived from the filename without `.cog.md`. MUST be unique within its `partOf` namespace. |
| `cogType` | string | 2 | Cog classification. Common values: `info` (reference / documentation), `action.scripted` / `action.sop` / `action.hybrid` (carries an execute contract; the discriminator after the dot names the cognitive class — see §6.5.2; `action` paired with a separate `actionType` field is an equivalent alias), `routing` (agent navigation), `certificate-of-genuineness` (publisher-provenanced credential), `cogs` (community-owned governance standard; see §6.5.1). |
| `category` | string | 2 | Registry grouping (e.g. `mx-core`, `mx-tool`, `mx-content`). Coarser than `cogType`. |

#### 6.5.1 The `cogs` classification (community-owned governance standard)

A `cogs` cog (initialism: Community Owned Governance Standard) is a cog whose definition is owned and stewarded by an open community rather than a single vendor or author. The classification denotes the governance model, not the file format — every `cogs` cog is still a normal `.cog.md` file with the same frontmatter shape as any other cog. What differs is who can change the cog's definition and through what process.

A cog declaring `cogType: cogs` MUST also identify the community that governs its definition. The MX community uses <https://tg.community> as the convening venue for The Gathering's cog stewardship. Other communities MAY use this classification for their own governance arrangements, identifying the governing body in the cog's `partOf`, `publisher`, or `provenanceOrigin` field as appropriate.

The `cogs` classification is intended for cogs that act as community-ratified reference definitions — field dictionaries, terminology registers, governance policies, conformance rubrics — where vendor capture would defeat the purpose. A `cogs` cog has no implied license; the cog's own `license` field still applies.

#### 6.5.2 Action cog discriminator (`cogType: action.<kind>` and `actionType`)

The cog specification draws a sharp axis through every action cog: it MAY be **deterministic**, **inference-driven**, or **both**. The discriminator named in this subsection is how the cog declares which. A deterministic cog carries fixed instructions a runtime executes the same way every time; an inference-driven cog carries instructions a language-model runtime reads and performs using its own reasoning; a both-form cog carries each in the role each is fit for. A consumer that knows this axis up front can choose to accept, reject, or sandbox the cog without inspecting the body. A consumer that does not know it has no way to tell whether the cog will run a shell command or invite a language model to act on its behalf — two operationally very different requests.

An action cog MUST declare its cognitive class so a consumer (human or agent) knows what kind of execution is involved without inspecting the body. Two forms are permitted, both expressing the same meaning:

- **Dotted form (canonical):** `cogType: action.scripted`, `cogType: action.sop`, or `cogType: action.hybrid`. The discriminator lives entirely in the `cogType` value. A consumer parses one field.
- **Deconstructed form (alias):** `cogType: action` with a separate `actionType: scripted | sop | hybrid` field. Equivalent semantically; permitted for tools or vendors that prefer the parallel-field shape.

A cog MUST NOT declare both forms with disagreeing values. When both are present they MUST agree (e.g. `cogType: action.scripted` with `actionType: scripted` is valid; `cogType: action.scripted` with `actionType: sop` is a conformance failure).

The three discriminator values:

- **`scripted`** — the cog body carries an embedded executable artefact (a fenced code block annotated `@embedded:<id>` per the cog specification). A runtime extracts the artefact by id and runs it directly. Deterministic; the same inputs produce the same outputs.
- **`sop`** — the cog body carries no embedded executable artefact. The `execute.actions[].usage` value is descriptive prose intended for a language-model runtime to read and perform the steps. The runtime is the language model itself.
- **`hybrid`** — the cog carries both an embedded executable artefact AND descriptive `usage` prose. The script handles the deterministic portion; the prose carries the judgement-dependent portion a language model performs.

| Field | Type | Zone | Profile | Conformance |
|-------|------|:----:|---------|-------------|
| `cogType` | string | 2 | cog (action) | MUST be of the form `action.<kind>` for action cogs (or `action` when paired with `actionType`) |
| `actionType` | string | 2 | cog (action) | OPTIONAL alias of the dotted form; SHOULD NOT appear when the dotted form is used; MUST NOT appear on cogs without `execute` |

A cog declaring `execute` MUST declare the discriminator (in either form) so consumers can resolve the runtime requirement (interpreter, language model, or both) before extracting any artefact. A cog without `execute` MUST NOT declare the discriminator in either form.

**Example (dotted form, canonical):**

```yaml
mx:
  cogType: action.scripted
  execute:
    actions:
      - name: run
        description: Execute the embedded script
```

**Example (deconstructed form, alias):**

```yaml
mx:
  cogType: action
  actionType: scripted
  execute:
    actions:
      - name: run
        description: Execute the embedded script
```

#### 6.5.3 `cogUrn` — universal identifier

A cog SHOULD declare a `cogUrn` field carrying its universal identifier as a URN-shaped string of the form `cog:<publisher-namespace>:<local-id>`. Unlike `cogId` (§6.5), which is reserved in vendor-extension space for a registry-local short identifier, `cogUrn` is a standard-tier field intended for use by every party outside the publisher's own systems: registries, clients, verifiers, and downstream agents.

The `<publisher-namespace>` is a stable namespace the publisher controls, typically derived from the publisher's DID (for example `web:example.com` for `did:web:example.com`). The `<local-id>` is a publisher-chosen identifier unique within that namespace; [ULID](https://github.com/ulid/spec) values are RECOMMENDED for `<local-id>` because they are time-sortable and collision-resistant without coordination.

| Field | Type | Zone | Profile | Conformance |
|-------|------|:----:|---------|-------------|
| `cogUrn` | string | 2 | cog (any) | SHOULD; URN-shaped `cog:<publisher-namespace>:<local-id>`; permanent across versions of the same logical record |

The `cogUrn` is set when the cog is created. It is permanent: a cog never changes its `cogUrn`. New versions of the same logical record (a corrected record, an amended clause) share the `cogUrn` and differ in `schemaVersion` (§6.5.4). The namespace prefix ensures global uniqueness without a central authority; two publishers controlling distinct domains cannot collide.

A publisher that loses control of its namespace is a recoverable situation as long as the original DID document remains resolvable; permanent loss is a worst-case that the trust layer handles per the MX Attestation Records and Verification note, with liability resting on the publisher.

**Example:**

```yaml
mx:
  cogUrn: cog:web:example.com:01JC8X7K2N4P5Q6R7S8T9V0W1Y
```

`cogUrn` and `cogId` (§6.5) are independent identifiers and MAY both be present on the same cog. `cogUrn` is the universal long-form identifier; `cogId` is a vendor-extension short-form identifier scoped to a registry's namespace.

#### 6.5.4 `schemaVersion` — cog-schema version

A cog SHOULD declare a `schemaVersion` field carrying the semver of the cog schema this record conforms to. `schemaVersion` is a standard-tier field, distinct from `cogHeader.version` (§5.1) which declares the cog-spec version a file conforms to.

| Field | Type | Zone | Profile | Conformance |
|-------|------|:----:|---------|-------------|
| `schemaVersion` | string | 2 | cog (any) | SHOULD; semver (`MAJOR.MINOR.PATCH` or `MAJOR.MINOR`) |

The Gathering versions cog schemas using semver:

- **Patch** (1.0.0 → 1.0.1): editorial corrections to the schema, no field changes. Existing cogs remain valid.
- **Minor** (1.0.0 → 1.1.0): new optional fields added. Existing cogs remain valid. Clients that understand the previous minor version parse the newer cogs and ignore the unfamiliar fields.
- **Major** (1.0.0 → 2.0.0): breaking changes, including field removals, semantics changes, or required-field additions. Existing cogs remain readable but require version-aware clients.

Registries MUST serve cogs at whatever `schemaVersion` they were published under. A v1.0 cog is served as v1.0 indefinitely. Registries MUST NOT migrate cogs to newer schema versions; the publisher MAY publish a new version of the same `cogUrn` under a newer `schemaVersion` if they choose, but the old version remains queryable.

Clients SHOULD accept any minor version within the major version they support. Clients encountering a major version they do not support SHOULD report this to the caller rather than attempt partial parsing.

**Example:**

```yaml
mx:
  schemaVersion: 1.0
```

The Gathering publishes a schema-evolution policy as part of the cog-schema documentation. The policy commits to: no breaking changes within a major version; a minimum window between major versions so the ecosystem can adapt; both the current and previous major versions remaining supported for a published transition period.

### 6.6 Composition and content-shape fields

The following fields describe the cog's body content and how it composes with other cogs. They likewise live in vendor-extension space.

| Field | Type | Zone | Purpose |
|-------|------|:----:|---------|
| `blocks` | array | 2 | Names the typed content blocks present in the document body (e.g. `prose`, `action`, `definition`, `essence`, `provenance`, `version`, `code`, `html`, `sop`, `security`). The `prose` block is implicit and need not be declared. |
| `includes` | array | 2 | Cog composition declarations. Each entry names a source cog, an optional block filter, and a resolution mode (`build` resolves at build time and inlines content; `runtime` resolves on each request). |
| `execute` | object | 2 | Action-doc execution contract. Carries `runtime`, `command`, optional `actions[]`, optional `policy`. The presence of an `execute` field distinguishes action cogs from info cogs. |
| `policy` | string-or-object | 2 | Content-handling rules for agents. Inheritable from a parent uber-doc. |
| `deliverable` | string-or-array | 2 | What this cog produces or delivers — a report, a validated artefact, a published page. For action cogs, describes what running the actions yields; for info cogs, describes the knowledge or artefact the cog represents. |

#### 6.6.1 Named sub-sections within the prose block

A single prose block MAY carry more than one named zone. A **sub-section** is a contiguous span of prose inside the document body enclosed by two HTML comments of the form `<!-- begin: <id> -->` and `<!-- end: <id> -->`. The `<id>` is a stable, kebab-case identifier. The markers are HTML comments so they survive markdown round-trips and render invisibly in any HTML view of the document.

The prose block remains the single implicit block of the cog. Sub-sections are an addressing convention within that block, not separate blocks; the `blocks` array does not list them.

Sub-sections are addressable by id. Readers and renderers MAY include, exclude, or extract a named sub-section. The canonical motivating case is a renderer that produces two derived artefacts from one source — for example, a covering letter plus a formal briefing in one markdown file, where one derived artefact strips the covering-letter sub-section and the other keeps it. Prose outside any marker pair is the default zone and is always included.

Conformance requirements:

- Sub-sections MUST NOT nest. A marker pair MUST close before the next pair opens. Nested ids constitute a malformed document.
- Each `<id>` MUST appear in at most one marker pair per document. Repeated ids constitute a malformed document.
- A reader that does not implement sub-section addressing MUST treat the markers as ordinary HTML comments and render the contained prose as part of the body.

### 6.7 Action contract fields

The following fields apply to cogs that declare an `execute` block or a procedure-declaring field. They make two questions a consumer always asks of an action cog answerable from frontmatter alone: *what shape does a successful output take* (`produces`), and *what should the runtime do when a failure arises that I have not catalogued* (`defaultRemedy` paired with `troubleshooting`). Both fields reside in Zone 2 (the `mx:` object).

#### 6.7.1 `produces`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Tier C) for cogs that emit a structured output; MAY otherwise |

**Definition:** Output contract. Declares the typed shape a successful execution of this cog produces, so consumers can reason about (and validate) the output without first running the cog. Distinct from `deliverable` (§6.6), which names what the cog yields in human terms; `produces` declares a machine-checkable shape.

**Sub-keys:**

| Sub-key | Type | Required when present | Definition |
|---------|------|-----------------------|-----------|
| `shape` | string | yes | Pointer to a JSON Schema, YAML schema, or named profile describing the output. Resolved using the same rules the host runtime applies to a cog's input `schema` field. |
| `format` | string | optional | MIME type or named format identifier (`application/json`, `text/markdown`, `application/pdf`). |
| `example` | any | optional | An illustrative output value matching `shape`. |

**Example:**

```yaml
mx:
  produces:
    shape: ./schemas/audit-report.v1.yaml
    format: application/json
    example:
      summary: "12 issues, 3 critical"
      issues: []
```

- A cog with no `execute` block and no procedure-declaring field SHOULD NOT declare `produces`.
- Distinct from the document-level `schema` field (input contract for the document itself) and from `deliverable` (semantic description, not typed shape).
- Runtimes MAY validate the produced output against `produces.shape` after execution; runtimes that do so MUST report a shape mismatch as an unmodelled failure (see `defaultRemedy`).

#### 6.7.2 `defaultRemedy`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cog |
| **Conformance** | SHOULD (Tier C) when `troubleshooting` is declared; MAY otherwise |

**Definition:** Catch-all remedy. Names the runtime-resolvable remedy a runtime SHOULD invoke when a failure arises that does not match any entry in `troubleshooting`. Resolves through the same registry used for `troubleshooting[].remedy` and obeys the same dotted-name syntax (lowercase, two or more dot-separated segments).

**Example:**

```yaml
mx:
  defaultRemedy: cogs.remedies.halt-and-escalate
  troubleshooting:
    - condition: required-field-missing
      remedy: cogs.remedies.request-supplier-resubmit
    - condition: validator-timeout
      remedy: cogs.remedies.retry-with-backoff
```

- The value MUST match the validator name regex (`^[a-z][a-z0-9_-]*(\.[a-z][a-z0-9_-]*)+$`).
- A cog declaring `defaultRemedy` without `troubleshooting` is valid but unusual; it asserts a single fallback for every failure mode.
- Distinct from a per-condition `troubleshooting` entry: `defaultRemedy` catches unmodelled conditions, not named ones, and removes the runtime's need to guess a safe behaviour.
- Combines naturally with `produces`: if a runtime cannot produce an output matching `produces.shape`, the situation is an unmodelled failure and `defaultRemedy` applies.

---

## 7. Conformance summary

| Field | Tier A (MUST) | Tier B (SHOULD) | Tier C (MAY) |
|-------|:-:|:-:|:-:|
| `partOf` | MUST | — | — |
| `buildsOn` | — | SHOULD | — |
| `cogHeader` *or* magic-header comment | — | SHOULD (when circulating) | — |
| `cogUrn` | — | SHOULD | — |
| `schemaVersion` | — | SHOULD | — |
| `dependencies` | — | — | MAY |
| `refersTo` | — | — | MAY |
| `cogId`, `cogType`, `category` | — | — | MAY |
| `blocks`, `includes`, `execute`, `policy`, `deliverable` | — | — | MAY |
| `produces`, `defaultRemedy` | — | — | MAY (SHOULD when the cog declares an `execute` block or a procedure-declaring field) |

A cog at Tier A is registry-locatable. At Tier B it is identifiable to unfamiliar consumers and provides its context graph. At Tier C it is fully described — dependencies, classifications, body content shape, and any execution contract.

---

## 8. Security and privacy considerations

- URLs declared in `cogHeader.runtime` may leak deployment topology. A cog declaring an internal runtime URL is exposing that URL to any reader of the cog. Operators concerned with topology privacy SHOULD either publish a public runtime URL or omit the field.
- A `cogHeader.spec` or `cogHeader.runtime` URL pointing at an attacker-controlled host is a phishing vector for runtimes that auto-fetch from those URLs. The protections are normative:
  - Runtimes **MUST** validate the `cogHeader.spec` and `cogHeader.runtime` URLs against an operator-controlled allowlist or a trusted registry before any fetch.
  - Runtimes **SHOULD NOT** auto-fetch unknown spec or runtime URLs. A first sight of an unrecognised URL warrants operator confirmation, not automatic resolution.
  - Runtimes **MAY** refuse to process cogs whose declared spec URL is unrecognised or unreachable, returning an explicit identification failure rather than guessing a fallback.
- The magic-header comment and `cogHeader` field are NOT authenticated — anyone who can write the cog can lie about which spec it claims to follow. Authentication, if required, is the responsibility of an external signing or witness mechanism, not this note.
- Cogs declaring `execute` contracts give agents a runnable surface. Operators MUST treat `execute.command` and `execute.actions[]` values as untrusted input and apply the same sandboxing, allowlisting, and audit measures they apply to any user-supplied command.
- A signed cog only attests provenance and integrity, never the truth of the content. Even when a cog's signature verifies cleanly, downstream consumers MUST NOT treat the values as factually correct — the signature confirms only that the cog genuinely came from the named signer and has not been altered. Editorial truth-curation is a different problem and out of scope for the cog format. User-facing language SHOULD say *attested*, not *verified*, to avoid implying a truth claim no signer makes.

---

## 9. References

### 9.1 Normative references

- [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119), [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) — keywords for use in RFCs to indicate requirement levels

### 9.2 Informative references

- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — conformance level model that inspired the Tier A/B/C framework in §2.1

---

## 10. Author quickstart (Informative)

Two minimum-viable cogs an author can copy and adapt — one of each type.

### 10.1 Minimum-viable info-cog (Tier A)

The smallest valid info-cog. Identity floor is MX Core Level 1; cog layer is Tier A. Saved as `auth-design-notes.cog.md`:

```markdown
---
# cog v1 spec=https://example.org/cog-spec.v1.md runtime=https://example.org/cog-runtime.md
title: "Auth design notes"
description: "Working notes on the authentication redesign."
created: 2026-05-01
modified: 2026-05-07
author: "Tom Cranstoun"
mx:
  partOf: mx-internal
  cogType: info
---

# Auth design notes

(Body content goes here.)
```

This cog declares the four MUST-at-Level-1 identity fields (`title`, `created`, `modified`, `author`), the one MUST-at-Tier-A cog field (`partOf`), and an info-cog `cogType`. The magic-header YAML comment on the line immediately after the opening `---` identifies the document as a cog to any reader that scans the frontmatter.

### 10.2 Minimum-viable action-cog (Tier C)

The smallest valid scripted action-cog. Identity floor is MX Core Level 2; cog layer is Tier C. Saved as `run-audit.cog.md`:

````markdown
---
# cog v1 spec=https://example.org/cog-spec.v1.md runtime=https://example.org/cog-runtime.md
title: "Run audit"
description: "Run the audit pipeline against a target site."
created: 2026-05-01
modified: 2026-05-07
author: "Tom Cranstoun"
cogHeader:
  version: v1
  spec: https://example.org/cog-spec.v1.md
  runtime: https://example.org/cog-runtime.md
mx:
  partOf: mx-tools
  buildsOn: [audit-runtime]
  dependencies:
    - name: audit-runtime
      kind: cog
  cogType: action.scripted
  execute:
    runtime: bash
    actions:
      - name: run
        description: Run the audit pipeline
---

# Run audit

```bash
@embedded:run
echo "running audit"
```
````

This cog satisfies Tier C: `partOf`, `buildsOn`, a `cogHeader` block (and a magic-header comment line on the first line of the frontmatter), `dependencies` declared, the canonical dotted `cogType`, and an `execute` contract with a named action whose embedded artefact is identifiable to a runtime.

---

## 11. Consumer checklist (Informative)

The order a consuming tool SHOULD process a cog file:

1. **Scan the frontmatter for the magic-header line.** Inside the YAML frontmatter (the block between the opening `---` and the closing `---`), look for a YAML comment line of the form `# cog <version> <key=value> ...`. Parse the version token and any `spec=`, `runtime=`, `runtime-doc=` tokens.
2. **Parse YAML frontmatter.** Read identity (Zone 1) and operational (`mx:`) fields. If a `cogHeader` field is present, verify it agrees with the magic-header per §5.2.
3. **Validate the spec URL.** If `cogHeader.spec` is not on the operator's allowlist or trusted registry, refuse to process the cog or escalate to operator confirmation (§8).
4. **Resolve `partOf`.** Locate the cog within its parent registry. A cog whose `partOf` cannot be resolved is identifiable but not navigable.
5. **Walk `buildsOn`.** Load the named context cogs into the graph. Missing `buildsOn` entries are warnings, not failures.
6. **Walk `dependencies`.** Hard-resolve each entry per its declared `kind`. A missing `dependencies` entry is a hard failure (§6.3).
7. **Branch on `cogType`.**
    - `info` / `routing` / `cogs` / `certificate-of-genuineness` — process as a documentation or governance artefact; do not look for `execute`.
    - `action.<kind>` (or `action` with a separate `actionType`) — process as an executable cog; resolve the runtime requirement before extracting any artefact.
8. **For action cogs, validate `execute`.** Confirm the runtime is permitted in the consumer's environment. For `scripted` and `hybrid` cogs, locate the `@embedded:<id>` artefact named in `execute.actions[].name`.
9. **For action cogs that declare `produces`,** prepare to validate the output against the declared shape after execution. A shape mismatch is an unmodelled failure and triggers `defaultRemedy` (§6.7.2).

---

## 12. Common authoring mistakes (Informative)

Three shapes a fresh author often produces, each with the fix.

**Mistake 1: discriminator forms disagree.**

```yaml
# WRONG — dotted form and deconstructed alias contradict each other
mx:
  cogType: action.scripted
  actionType: sop
```

The two forms MUST agree (§6.5.2). Pick one form; if both are present, set `actionType` to the suffix of `cogType` or remove it entirely. The dotted form alone is sufficient.

**Mistake 2: `actionType` on an info-cog.**

```yaml
# WRONG — actionType MUST NOT appear without an execute block
mx:
  cogType: info
  actionType: sop
```

`actionType` is the discriminator for action cogs only (§6.5.2). On a cog without `execute`, remove the field.

**Mistake 3: claiming Tier B without an identifier.**

```yaml
# WRONG — Tier B requires either a cogHeader OR a magic-header
mx:
  partOf: mx-tools
  buildsOn: [other-cog]
```

The Tier B SHOULD-or-magic-header rule (§2.1, §5.1) requires at least one of: a magic-header YAML comment line inside the frontmatter, or a `cogHeader` field in the frontmatter. A cog circulating outside a closed system MUST carry one.

**Mistake 4: cog structural fields at the top level.**

```yaml
# WRONG — partOf and dependencies belong under mx:, not at the top level
title: "Run audit"
partOf: mx-tools
dependencies: [...]
```

All cog structural fields live in Zone 2 (§6) unless explicitly noted as top-level (`cogHeader`, `contractFields`, `metadataFields`). Move the misplaced fields under the `mx:` object.

---

*End of MX Cogs note draft.*

<!-- cog-spec-sync: 2026-07-01-a -->
