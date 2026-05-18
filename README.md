# MX Drafts for The Gathering

**Make anything you publish — a video, a podcast, a PDF, an image, a web page — readable by machines.** That is what MX is for. The drafts in this repo are the open-standard specifications that define how.

Public home for the MX metadata draft notes. These are draft notes authored by Tom Cranstoun and offered to [The Gathering](https://tg.community) community for review via [Stream](https://stream.tg.community). They are not ratified standards.

All drafts are written in plain markdown with kramdown-rfc YAML frontmatter. They cannot use MX metadata to describe themselves, since they are the standards specifying it.

## The drafts

The **MX Field Definition Pattern note** is the primary note of the set: it specifies how every other note defines its frontmatter fields. Read it first.

| Draft | File | Status |
|-------|------|--------|
| **MX Field Definition Pattern note** *(primary)* | [draft-field-pattern.md](draft-field-pattern.md) | Draft (v1.0) — the authoring pattern every sister note follows when defining a frontmatter field; recommends a reading order across the draft set. |
| **MX Core Metadata note** | [draft-core-metadata.md](draft-core-metadata.md) | Draft (v1.0) — Zone 1 / Zone 2 document metadata and externally-aligned fields. |
| **MX Cogs note** | [draft-cogs.md](draft-cogs.md) | Draft (v1.0) — the `.cog.md` file format as an optional layer on top of MX. |
| **MX Extensions note** | [draft-extensions.md](draft-extensions.md) | Draft (v1.0) — namespace policy and context-specific naming. |
| **MX Provenance note** | [draft-provenance.md](draft-provenance.md) | Draft (v1.0) — attribution, trust, maintenance, decision records. **Part of the Level-2 floor for documents that need verifiable origin and stewardship.** |
| **MX Temporal Stance note** | [draft-temporal-stance.md](draft-temporal-stance.md) | Draft (v1.0) — temporal vocabulary for documents whose prose depends on dated anchors (regulatory analyses, contracts, SLAs, compliance reports, pricing pages). |
| **MX Carrier Formats note** | [draft-carrier-formats.md](draft-carrier-formats.md) | Draft (v1.0) — carrier mechanisms (markdown, HTML, JSDoc, CSS, shell, XMP, sidecar, SQL) and code-specific provenance. **Carries the binding HTML mapping for the four MUST-at-Level-2 web fields.** |
| **MX Workflow Contracts note** | [draft-workflow-contracts.md](draft-workflow-contracts.md) | Draft (v1.0) — top-level fields for workflow contract cogs (thresholds, approvers, procedures, target environment). |
| **MX Agent Directory Discovery note** | [draft-agent-directory-discovery.md](draft-agent-directory-discovery.md) | Draft (v1.0) — three-layer discoverability standard for `llms.txt` and any agent-directory file: HTML transport, sitemap inclusion, in-page `<link>`. |
| **MX Document Accessibility note** | [draft-document-accessibility.md](draft-document-accessibility.md) | Draft (v1.0) — three-layer accessibility standard for non-HTML document carriers: tagged structure, declared conformance, independent verification. PDF normative; DOCX and EPUB informative. Defers to ISO 14289 PDF/UA, ISO 32000, WCAG 2.1, BCP 47, Schema.org accessibility properties, and Directive (EU) 2019/882 (EAA). |
| **MX Contract Fingerprinting and Signing note** | [draft-contract-fingerprinting.md](draft-contract-fingerprinting.md) | Draft (v1.0) — *signing is optional*; this note specifies the contract a cog must satisfy when it elects to be signed. The contract scope is derivable from a `schema:` annotation. |
| **MX Compliance Claims note** | [draft-compliance-claims.md](draft-compliance-claims.md) | Draft (v1.0) — signed, dated, revocable claims that an artefact conforms to a named external standard. Defines claim structure, self-attestation and third-party authority types, evidence and revocation contracts, and predicate-vocabulary requirements. WCAG 2.2 Level AA carried as the worked example. |
| **MX Attestation Records and Verification note** | [draft-attestation-records.md](draft-attestation-records.md) | Draft (v1.0) — partial: §4.1 / §4.2 only — the record format a trust layer produces when wrapping a cog with cryptographic apparatus, and the producer/serve/verify split (registries serve; verifiers verify). Builds on the **MX Contract Fingerprinting and Signing note** §5 for canonicalisation; signing mechanics are not restated. |
| **MX Decision Records note** | [draft-decision-records.md](draft-decision-records.md) | Draft (v1.0) — the record class an operator of an automated decision system produces for each decision (or batched group of decisions). Carries the cited policy reference, model identity, input commitment, output summary, and human-in-loop status. Companion to the **MX Policy Records note** (cited by the decision) and the **MX Attestation Records and Verification note** (which wraps the record cryptographically). |
| **MX Policy Records note** | [draft-policy-records.md](draft-policy-records.md) | Draft (v1.0) — the record class that publishes a regulatory code, code of practice, or governance policy as a citable artefact. Carries stable identifier, semver versioning, jurisdiction, effective-from date, and addressable clauses. Cited by decision records via `policyRef`; permits third-party rendering by intermediaries until issuing bodies publish their own. |
| **MX AI Usage Declaration note** | [draft-ai-usage-declaration.md](draft-ai-usage-declaration.md) | Draft (v1.0): a publisher's signed disclosure of how AI participated in producing a body of work. Defines a schema (subject, publisher, author, `aiUsage`, `aiBoundary`), a discovery convention (root-level `/AI-USAGE.*` or `/.well-known/ai-usage`, plus `<link rel="ai-usage">`), and a signature mechanism (RFC 7515 JWS over a canonical payload, Ed25519 RECOMMENDED). Sits in the gap between `ai.txt` (training opt-out), `llms.txt` (inference navigation), and C2PA (asset-level provenance). |
| **MX scope note: machine-readable documents beyond web optimisation** *(boundary note)* | [draft-mx-not-geo.md](draft-mx-not-geo.md) | Draft (v1.0) — the boundary between MX governance and the Generative Engine Optimisation (GEO/AEO) vendor category. MX builds coverage across an unknowable landscape of machines and formats; GEO targets one pathway and accepts failure on the others. |
| **MX scope note: file-borne provenance beyond memory-pool architectures** *(boundary note)* | [draft-mx-not-memory-pool.md](draft-mx-not-memory-pool.md) | Draft (v1.0) — the boundary between MX governance and memory-pool architectures (LLM-wikis, Karpathy-style compiled knowledge bases, Obsidian vaults). A memory pool governs how knowledge is organised inside one system; MX governs the DNA a file carries when it leaves any system. The two are complementary, not competitors. |

Every draft here is **a draft, not a ratified standard**. None is final. Each will evolve through public review and, if the community consents, be ratified by The Gathering.

Each draft is **standalone**: it defines its own conformance levels and field semantics inline, and refers only to actual external published standards (RFC, ISO, W3C, NIST, Schema.org, Dublin Core, SPDX, and similar) for normative content. Where one draft excludes a topic owned by a sister draft, it names the sister draft for orientation; no draft depends on another for normative material — except that all sister drafts adhere to the field-definition pattern in the primary note.

## How the drafts fit together

- **MX Field Definition Pattern** is the primary note: every sister note follows its template when defining a frontmatter field, and adopts its discipline (machines need more fields, tightly constrained, to understand intent).
- **MX Core Metadata** is the vocabulary floor. Any text-bearing artefact can adopt it: a markdown file, an HTML page, a YAML sidecar. Reading this and **MX Provenance** together gives an author the full Level-2 floor.
- **MX Cogs** adds an optional layer for documents that want to be navigable, composable, and runnable by agents. The cog ladder grades a document with **Tier A / B / C** to avoid colliding with the MX Core Level 1/2/3 ladder; a cog is graded by both at once.
- **MX Extensions** governs the namespace policy (standard / vendor public / vendor private prefixes) so vendors can extend without polluting the core vocabulary. The public namespace is operated by CogNovaMX during the seed phase; vendor sub-namespaces of the form `x-mx-{vendor}-*` are reserved for individual vendors.
- **MX Provenance** covers attribution, trust, maintenance, and decision-record references — the metadata that makes a document's origin and stewardship verifiable. Skipping this note leaves authors inventing private `x-mx-prov-*` fields they have to migrate later. Read it alongside **MX Core Metadata**.
- **MX Temporal Stance** specifies how a document declares its relationship to time. Regulatory analyses, contracts, SLAs, compliance reports, and pricing pages — anything whose prose depends on dated anchors — read this. Documents that are time-neutral can skip it.
- **MX Carrier Formats** specifies how MX metadata is carried in each supported file format and adds a small code-specific provenance vocabulary; what code does (signatures, APIs, tests) defers to JSDoc, docstrings, OpenAPI, and similar. The binding form of the HTML kebab-case `mx:` mapping (and the four MUST-at-Level-2 web fields) lives here.
- **MX Workflow Contracts** defines a small set of optional top-level fields for cogs that declare an executable approval, review, or procedural workflow.
- **MX Contract Fingerprinting and Signing** specifies the format a cog uses *when it chooses to be signed*. Signing is optional. The contract surface is derivable from a `schema:` annotation; the explicit `contractFields` array is the override.
- **MX Compliance Claims** specifies how an artefact carries a signed, dated, revocable claim of conformance to a named external standard. Two authority types are supported from v1.0: self-attestation, where the publisher claims about their own work, and third-party attestation, where an independent authority claims about an artefact they did not produce. The note defers to **MX Contract Fingerprinting and Signing** for canonicalisation and signature mechanics, and is independent of any specific accreditation programme; accreditation is an operational layer on top.
- **MX scope note: not GEO** and **MX scope note: not memory-pool** are boundary notes. They do not introduce fields. They draw the line between MX governance and two adjacent vendor categories — Generative Engine Optimisation and memory-pool / LLM-wiki architectures — that practitioners and stakeholders frequently conflate with MX. Read them when you need to explain what MX is *not* doing in a given conversation.

## What this repo is

This repo is the canonical home for the drafts. There is one copy in one place, in one format. Public review and citation happen here.

## Related resources

- **The Gathering** — [tg.community](https://tg.community) · [stream.tg.community](https://stream.tg.community).

## Governance

These drafts are authored by Tom Cranstoun and offered to The Gathering for ratification. Until ratified they are proposals, not standards. The Gathering's role is to review, refine, and — by community consent — ratify.

Contributions welcome via Stream review. For editorial issues with the source drafts themselves, open an issue or PR here.

## Licence

MIT. See [LICENCE](LICENCE). Free, open, vendor-neutral — the MX vocabulary is for everyone.
