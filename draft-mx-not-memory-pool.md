---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX scope note: file-borne provenance beyond memory-pool architectures"
docname: draft-cranstoun-mx-not-memory-pool
date: 2026-05-07
consensus: false
keyword:
  - mx
  - memory-pool
  - llm-wiki
  - knowledge-base
  - obsidian
  - provenance
  - reginald
  - extraction
  - standalone
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-mx-not-memory-pool.md
---

# MX scope note: file-borne provenance beyond memory-pool architectures

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 7 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note defines the scope of the Machine Experience (MX) framework in relation to memory-pool architectures — the design pattern in which an LLM curates a structured knowledge base from raw documents and queries the curated artefact rather than the raw sources. The most cited current example is Andrej Karpathy's LLM-wiki proposal, in which Claude reads source documents and maintains an interconnected wiki of markdown files, viewed in Obsidian. The article describes raw sources as "source code" and the wiki as "the compiled executable" — you do not recompile every time you run the program.

MX and memory-pool architectures address different problems. Conflating them produces incorrect claims about what MX does and misses the complementary fit between the two.

The central MX scope claim relevant here is this: a memory-pool architecture governs how knowledge is organised inside one system. MX governs what a file carries when it leaves any system. The two are orthogonal. An MX-conformant document inside a memory pool is both better-organised by the pool and remains interpretable when an agent extracts it — at which point the pool's structure is no longer accessible and only the file's own metadata can answer the agent's questions.

---

## 2. Status of this note

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the scope definitions, claims, and recommendations in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. The memory-pool architecture and what it does

A memory-pool architecture (LLM-wiki, Karpathy's proposal, similar designs built on top of Obsidian, Notion, or other markdown knowledge bases) treats raw documents as sources to be ingested into a curated knowledge structure. An LLM reads incoming documents, integrates them with existing pages, creates entity pages, builds backlinks, and maintains the structure as new sources arrive.

A user (or an agent) then queries the curated wiki rather than the raw sources. The argument for this pattern is that real-time retrieval against raw sources (the RAG approach) repeats expensive interpretation on every query, whereas a curated wiki accumulates structured knowledge that compounds across queries and supports multi-hop reasoning the raw-sources approach cannot reach.

The pattern is sound. It addresses a real problem. It is not the problem MX addresses.

A memory-pool architecture answers the question: *how should one system organise knowledge for its own retrieval?* That question is internal to the pool. It does not address what happens when a file leaves the pool — when it is exported, attached to an email, archived, sent to a contractor, copied into another agent's context window, or encountered by a third-party machine that has no access to the originating pool's index.

---

## 4. The MX scope

MX governance addresses a problem the memory-pool architecture does not engage with: a machine reading a file may have no access to the originating pool, no knowledge of where the file came from, and no way to verify whether it is current.

This is not a hypothetical. Files are extracted from corpora and travel as standalone objects — PDFs in email, markdown copied into chat, signed contracts archived offline, training datasets bundled and shipped, cogs embedded in agent prompts. In every case, the file is encountered by a downstream agent in isolation. The pool's index, backlinks, and entity-page structure are no longer available to that agent.

MX governance ensures the file itself carries enough metadata to answer the questions the pool would otherwise answer for it:

1. What is this file? (content type, purpose, audience)
2. Who published it? (author, organisation, originator)
3. Has it been altered? (cryptographic signature, integrity attestation)
4. Is it current? (version, modified date, supersedes / supersededBy)
5. Where is the authoritative copy? (canonicalUri, Reginald registry entry)

When these are present, an agent meeting the file in the wild reads them directly rather than inferring them from prose context. Inference is the failure mode the memory-pool architecture is itself trying to avoid; MX prevents that failure mode from re-appearing the moment a file crosses the pool's boundary.

---

## 5. The DNA framing

A useful shorthand: when you extract a file from any pool, MX is the DNA the file carries with it.

The pool — whether it is an LLM-wiki, an Obsidian vault, a CMS, a document management system, a training corpus, or a personal note collection — provides organisational context: backlinks, tags, taxonomies, the position of the file in a graph of related files. That context lives in the pool. The moment the file is extracted, the context is gone unless the file itself carries it.

DNA is the right metaphor because the metadata is *embedded in the file*, not declared by the surrounding system. A YAML frontmatter block, an XMP packet in a PDF, a Schema.org JSON-LD script in HTML, a `.mx.yaml` sidecar — these all encode the file's identity in a form that survives extraction. The downstream agent does not have to reconstruct provenance; it reads it.

This framing also clarifies what MX is *not*:

- MX is not a memory-pool design. It does not specify how a pool should organise its files.
- MX is not a retrieval algorithm. It does not specify how an agent should query a corpus.
- MX is not a substitute for RAG, vector search, or compiled-wiki approaches.
- MX does not compete with Karpathy's LLM-wiki proposal. The two are complementary.

A memory pool built from MX-conformant files is a stronger memory pool than one built from raw markdown. The pool's curated structure benefits from explicit metadata (an LLM building backlinks reads `provenanceAuthor` directly rather than inferring authorship from prose); files extracted from such a pool carry their DNA into whatever context they land in next.

---

## 6. Why this matters at LLM scale

At small scale, a memory-pool architecture is closed: one user, one Obsidian vault, one Claude session. Files rarely leave. Provenance can be implicit because the user remembers where things came from.

At LLM scale — the operating context for which Karpathy's proposal is interesting — files leave constantly. An agent extracts a wiki entry to answer a query. Another agent reads that entry and extracts a fragment to feed into a third agent's context. A research pipeline harvests entries to build a training set. A user copies an entry into an email. A regulator subpoenas the corpus. A future maintainer encounters the file three years later, divorced from the wiki it lived in.

In every one of those cases, the agent or human encountering the file has no access to the wiki's structure. The only thing the receiving party can read is the file itself. If the file carries MX metadata, the receiving party knows what it is, who wrote it, when it was published, and whether it is the current version. If the file does not, the receiving party guesses — and guesses become hallucinations at the next inference step.

This is the structural argument for MX in the memory-pool context: the pool's value compounds across queries inside the pool, but a file's value compounds across systems only when it carries its own DNA. A memory pool of MX-conformant files captures both.

---

## 7. The Reginald registry

Reginald (Registry for Genuine Information, Notarised Authentication, and Legitimate Documentation) is the infrastructure layer that makes MX governance verifiable at internet scope.

Reginald operates as a queryable registry that any agent can consult, without permission or payment, to verify a document's provenance and currency. An agent encountering a file in the wild — extracted from a memory pool, attached to an email, embedded in a training dataset — can pass the file's canonical identifier to Reginald and receive a verified answer: this is the content the named publisher signed, unaltered, this is the current version, this is the master location.

The complementarity to memory-pool architectures is direct. Inside the pool, an LLM curates structured knowledge. Outside the pool, Reginald answers the questions the pool's structure would have answered if it were still accessible. The two together cover both the inside-system and outside-system cases that real-world AI deployments routinely span.

Reginald attests one thing: this is the content the named publisher signed, unaltered. It makes no claim about whether the content is true, current in its subject matter, or sound in its reasoning. Editorial responsibility remains with the publisher.

---

## 8. Scope of this note

This note covers the following MX scope questions:

- What MX does (file-borne metadata that survives extraction from any pool)
- What MX does not do (specify how a pool organises its files, replace RAG or compiled-wiki retrieval, design a memory architecture)
- How memory-pool architectures and MX combine (orthogonal layers, both useful)
- The DNA principle (metadata embedded in the file, not declared by the pool)

This note does not cover:

- Specific memory-pool implementations (Karpathy's LLM-wiki, Obsidian, Notion, training-corpus designs) — these are reference points, not specifications MX defines
- MX field definitions (see the MX Core Metadata note and related draft set)
- Reginald's technical specification (a separate note covers the registry protocol)

---

## 9. Correct and incorrect claims about MX

### Correct

- "MX is the metadata DNA a file carries when it leaves any pool."
- "An MX-conformant file inside a memory pool remains interpretable after extraction; a raw file does not."
- "Memory-pool architectures and MX are complementary, not alternatives."
- "Reginald answers the questions a pool's index would have answered, but works on files in the wild."
- "Explicit file-borne metadata reduces the inference burden on every downstream agent."

### Incorrect

- "MX is a competitor to Karpathy's LLM-wiki."
- "MX is a memory-pool design."
- "MX replaces RAG or compiled-wiki retrieval."
- "An Obsidian vault is the same kind of system as MX governance."
- "MX is what Tom calls his version of an LLM-wiki."

The distinction: correct claims describe MX as the layer that makes a file interpretable when it leaves a system. Incorrect claims position MX as a system itself, competing with retrieval architectures it is in fact orthogonal to.

---

## 10. Relationship to other MX notes

This note is a scope clarification. It does not introduce new fields or change existing field definitions. It should be read alongside:

- **MX scope note: machine-readable documents beyond web optimisation** (`draft-mx-not-geo.md`) — the parallel boundary note for GEO
- **MX Core Metadata note** — defines the fields that implement file-borne DNA
- **MX Provenance note** — defines the attribution and trust fields a file needs to remain interpretable in isolation
- **MX Carrier Formats note** — defines how MX metadata is embedded across document formats so the DNA travels with the file
- **MX Contract Fingerprinting and Signing note** — defines the cryptographic signature that lets a downstream agent verify the file is unaltered

---

*This note is offered to The Gathering for review and is not a ratified standard.*
