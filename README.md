# MX Drafts for The Gathering

Public home for the source `.cog.md` drafts of the MX metadata notes. These are draft notes authored by Tom Cranstoun and offered to [The Gathering](https://tg.community) community for review via [Stream](https://stream.tg.community). They are not ratified standards.

## The drafts

| Draft | File | Status |
|-------|------|--------|
| **MX Core Metadata note** | [draft-core-metadata.cog.md](draft-core-metadata.cog.md) | Draft (v1.3-draft) — pure document metadata; no cog content. |
| **MX Cogs note** | [draft-cogs.cog.md](draft-cogs.cog.md) | Draft (v1.0-draft) — the `.cog.md` file format as an optional layer on top of MX. |
| **MX Extensions note** | [draft-extensions.cog.md](draft-extensions.cog.md) | Draft (v1.1-draft) — namespace policy and carrier-format mappings. |
| **MX Provenance note** | [draft-provenance.cog.md](draft-provenance.cog.md) | Draft (v1.0-draft) — attribution, trust, maintenance, decision records. |
| **MX Carrier Formats note** | [draft-carrier-formats.cog.md](draft-carrier-formats.cog.md) | Draft (v1.1-draft) — code-only carrier provenance. |
| **MX Contract Fingerprinting and Signing note** | [draft-contract-fingerprinting.cog.md](draft-contract-fingerprinting.cog.md) | Draft (v1.1-draft) — *signing is optional*; this note specifies the contract a cog must satisfy when it elects to be signed. |

Every draft here is **a draft, not a ratified standard**. None is final. Each will evolve through public review and, if the community consents, be ratified by The Gathering.

Each draft is **standalone**: it defines its own conformance levels and field semantics inline, and refers only to actual external published standards (RFC, ISO, W3C, NIST, Schema.org, Dublin Core, SPDX, and similar). Drafts do not cross-reference each other.

## How the drafts fit together

- **MX Core Metadata** is the floor. Any text-bearing artefact can adopt it: a markdown file, an HTML page, a YAML sidecar.
- **MX Cogs** adds an optional layer for documents that want to be navigable, composable, and runnable by agents. Most MX-aware documents will not be cogs.
- **MX Extensions** governs the namespace policy (standard / vendor public / vendor private prefixes) so vendors can extend without polluting the core vocabulary.
- **MX Provenance** covers attribution, trust, maintenance, and decision-record references — the metadata that makes a document's origin and stewardship verifiable.
- **MX Carrier Formats** specifies the minimal code-specific provenance vocabulary; what code does (signatures, APIs, tests) defers to JSDoc, docstrings, OpenAPI, and similar.
- **MX Contract Fingerprinting and Signing** specifies the format a cog uses *when it chooses to be signed*. Signing is optional. The fingerprint format is open.

## Reginald

**Reginald is Digital Domain Technologies Ltd's proprietary implementation for signing and serving cogs.** The fingerprint format defined in the Contract Fingerprinting and Signing note is open and free for anyone to implement; Reginald is one such implementation, and the only one DDT supports. Operators wanting to sign or serve cogs at scale can either run Reginald or build their own implementation against the open format.

The drafts in this repository describe the **format**. They are vendor-neutral. Reginald the product is not part of The Gathering's standard suite; it is a separate commercial offering from DDT.

## What this repo is — and what it isn't

This repo holds the **source drafts** as they're authored: `.cog.md` files with MX frontmatter and prose. It is the canonical reading copy.

The same content is also published under The Gathering's Stream platform in **RFC-format** form, one repo per draft, for the formal review and sync process:

- [TG-Community/draft-cranstoun-mx-core-metadata](https://github.com/TG-Community/draft-cranstoun-mx-core-metadata)
- [TG-Community/draft-cranstoun-mx-extensions](https://github.com/TG-Community/draft-cranstoun-mx-extensions)
- [TG-Community/draft-cranstoun-mx-provenance](https://github.com/TG-Community/draft-cranstoun-mx-provenance)
- [TG-Community/draft-cranstoun-mx-carrier-formats](https://github.com/TG-Community/draft-cranstoun-mx-carrier-formats)

Two formats, one set of drafts. Stream review happens in the RFC-format repos (that's where the platform expects to find them); reading, citing, and linking happens here.

## Related resources

- **The Gathering** — [tg.community](https://tg.community) · [stream.tg.community](https://stream.tg.community).

## Governance

These drafts are authored by Tom Cranstoun and offered to The Gathering for ratification. Until ratified they are proposals, not standards. The Gathering's role is to review, refine, and — by community consent — ratify.

Contributions welcome via Stream review. For editorial issues with the source drafts themselves, open an issue or PR here.

## Licence

MIT. See [LICENCE](LICENCE). Free, open, vendor-neutral — the MX vocabulary is for everyone.
