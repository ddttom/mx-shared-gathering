# MX Drafts for The Gathering

Public home for the MX metadata draft notes. These are draft notes authored by Tom Cranstoun and offered to [The Gathering](https://tg.community) community for review via [Stream](https://stream.tg.community). They are not ratified standards.

All drafts are written in plain markdown with kramdown-rfc YAML frontmatter. They cannot use MX metadata to describe themselves, since they are the documents specifying it.

## The drafts

The **MX Field Definition Pattern note** is the primary note of the set: it specifies how every other note defines its frontmatter fields. Read it first.

| Draft | File | Status |
|-------|------|--------|
| **MX Field Definition Pattern note** *(primary)* | [draft-field-pattern.md](draft-field-pattern.md) | Draft (v1.0) — the authoring pattern every sister note follows when defining a frontmatter field; recommends a reading order across the draft set. |
| **MX Core Metadata note** | [draft-core-metadata.md](draft-core-metadata.md) | Draft (v1.0) — Zone 1 / Zone 2 document metadata and pass-through fields. |
| **MX Cogs note** | [draft-cogs.md](draft-cogs.md) | Draft (v1.0) — the `.cog.md` file format as an optional layer on top of MX. |
| **MX Extensions note** | [draft-extensions.md](draft-extensions.md) | Draft (v1.0) — namespace policy and context-specific naming. |
| **MX Provenance note** | [draft-provenance.md](draft-provenance.md) | Draft (v1.0) — attribution, trust, maintenance, decision records. |
| **MX Carrier Formats note** | [draft-carrier-formats.md](draft-carrier-formats.md) | Draft (v1.0) — carrier mechanisms (markdown, HTML, JSDoc, CSS, shell, XMP, sidecar, SQL) and code-specific provenance. |
| **MX Workflow Contracts note** | [draft-workflow-contracts.md](draft-workflow-contracts.md) | Draft (v1.0) — top-level fields for workflow contract cogs (thresholds, approvers, procedures, target environment). |
| **MX Agent Directory Discovery note** | [draft-agent-directory-discovery.md](draft-agent-directory-discovery.md) | Draft (v1.0) — three-layer discoverability standard for `llms.txt` and any agent-directory file: HTML transport, sitemap inclusion, in-page `<link>`. |
| **MX Document Accessibility note** | [draft-document-accessibility.md](draft-document-accessibility.md) | Draft (v1.0) — three-layer accessibility standard for non-HTML document carriers: tagged structure, declared conformance, independent verification. PDF normative; DOCX and EPUB informative. Defers to ISO 14289 PDF/UA, ISO 32000, WCAG 2.1, BCP 47, Schema.org accessibility properties, and Directive (EU) 2019/882 (EAA). |
| **MX Contract Fingerprinting and Signing note** | [draft-contract-fingerprinting.md](draft-contract-fingerprinting.md) | Draft (v1.0) — *signing is optional*; this note specifies the contract a cog must satisfy when it elects to be signed. |

Every draft here is **a draft, not a ratified standard**. None is final. Each will evolve through public review and, if the community consents, be ratified by The Gathering.

Each draft is **standalone**: it defines its own conformance levels and field semantics inline, and refers only to actual external published standards (RFC, ISO, W3C, NIST, Schema.org, Dublin Core, SPDX, and similar) for normative content. Where one draft excludes a topic owned by a sister draft, it names the sister draft for orientation; no draft depends on another for normative material — except that all sister drafts adhere to the field-definition pattern in the primary note.

## How the drafts fit together

- **MX Field Definition Pattern** is the primary note: every sister note follows its template when defining a frontmatter field, and adopts its discipline (machines need more fields, tightly constrained, to understand intent).
- **MX Core Metadata** is the vocabulary floor. Any text-bearing artefact can adopt it: a markdown file, an HTML page, a YAML sidecar.
- **MX Cogs** adds an optional layer for documents that want to be navigable, composable, and runnable by agents. Most MX-aware documents will not be cogs.
- **MX Extensions** governs the namespace policy (standard / vendor public / vendor private prefixes) so vendors can extend without polluting the core vocabulary.
- **MX Provenance** covers attribution, trust, maintenance, and decision-record references — the metadata that makes a document's origin and stewardship verifiable.
- **MX Carrier Formats** specifies how MX metadata is carried in each supported file format and adds a small code-specific provenance vocabulary; what code does (signatures, APIs, tests) defers to JSDoc, docstrings, OpenAPI, and similar.
- **MX Workflow Contracts** defines a small set of optional top-level fields for cogs that declare an executable approval, review, or procedural workflow.
- **MX Contract Fingerprinting and Signing** specifies the format a cog uses *when it chooses to be signed*. Signing is optional. The fingerprint format is open.

## What this repo is

This repo is the canonical home for the drafts. There is one copy in one place, in one format. Public review and citation happen here.

## Related resources

- **The Gathering** — [tg.community](https://tg.community) · [stream.tg.community](https://stream.tg.community).

## Governance

These drafts are authored by Tom Cranstoun and offered to The Gathering for ratification. Until ratified they are proposals, not standards. The Gathering's role is to review, refine, and — by community consent — ratify.

Contributions welcome via Stream review. For editorial issues with the source drafts themselves, open an issue or PR here.

## Licence

MIT. See [LICENCE](LICENCE). Free, open, vendor-neutral — the MX vocabulary is for everyone.
