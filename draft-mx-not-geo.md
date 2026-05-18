---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX scope note: machine-readable documents beyond web optimisation"
docname: draft-cranstoun-mx-not-geo
date: 2026-05-18
consensus: false
keyword:
  - mx
  - geo
  - aeo
  - metadata
  - machine-readability
  - provenance
  - registry
  - standalone
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-mx-not-geo.md
---

# MX scope note: machine-readable documents beyond web optimisation

**Version:** 1.1
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 18 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note defines the scope of the Machine Experience (MX) framework in relation to the emerging vendor category called Generative Engine Optimisation (GEO). MX and GEO address different problems. Conflating them produces incorrect claims about what MX does and misleads practitioners about where to invest their effort.

The central MX scope claim is this: MX asks whether any machine can find any document in a corpus, verify it is genuine, and know whether it is current — regardless of which machine, which format, or which access pathway. GEO asks how to increase the probability that a specific class of LLM-powered system cites a specific web page. These are different questions with different answers.

---

## 2. Status of this note

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the scope definitions, claims, and recommendations in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. The GEO claim and why it fails

GEO vendors claim that adding structured markup — primarily Schema.org JSON-LD — increases the probability that LLM-powered systems cite a publisher's web page. This claim fails on its own terms.

Large language models process text as sequences of tokens. The transformer architecture does not inspect structured data tags in the way a classical search crawler does. Schema.org serves legitimate purposes in classical search (rich results, knowledge panels, voice assistant responses). It does not influence how a foundation model represents or retrieves content. The same query submitted to an LLM-powered system produces different answers across sessions, temperature settings, and model versions. No clean causal chain connects "added FAQ schema" to "the model cited my page."

GEO vendors fill this gap with dashboards and reportable metrics. The metrics measure activity, not outcomes. The outputs are defensible expenditure, not proven results.

This note takes no position on whether GEO as a category will develop a defensible evidence base. At the time of writing, it has not.

---

## 4. The MX scope

MX governance addresses a structural problem that GEO does not engage with: a machine reading a document may have no access to the originating website, no knowledge of where the document came from, and no way to verify whether it is current.

This is not a hypothetical. PDFs are downloaded and distributed. Cogs are extracted and used as context in AI pipelines. Contracts are signed and archived. White papers circulate for years after publication. In every case, the document may be encountered by a machine in isolation.

MX governance ensures that an isolated document contains everything a machine needs to:

1. Understand what it is (content type, purpose, audience)
2. Attribute it correctly (author, publisher, origin)
3. Verify it is unaltered (cryptographic signing)
4. Determine whether it is current (version, modified date, canonical location)
5. Find the authoritative source (canonicalUri, master location)

These requirements apply to every document format: web pages, PDFs (via XMP), Word documents, cogs, scripts, and any other format a machine might encounter.

---

## 5. Why the pathway is unknowable

Practitioners cannot know which machine will read a given document, or how. The machines reading content do not identify themselves reliably. User-agent strings are spoofable. Some agents screenshot rendered pages. Some read the DOM via browser automation. Some fetch raw HTML. Some request markdown. Some query structured data endpoints. Some read PDFs. Some are foundation models; some are on-device models with fewer than 100 million parameters that rely far more on explicit structure because they cannot fill gaps from inference.

The web is also architecturally hostile to machine access. Cookie banners, JavaScript-rendered content, accordion toggles, anti-scraping measures, and bot-detection systems block machines before they reach content.

The correct response to an unknowable pathway is not to optimise for one pathway and accept failure on the others. It is to build redundancy: the same facts expressed in multiple formats, across multiple access methods, so that any machine using any access method can reach the content.

---

## 6. The registry layer

MX defines a registry-contract layer that makes file-borne governance scalable at internet scope. The standard defines the contract; multiple registry operators are expected to run conforming implementations.

MX specifies that documents should carry provenance, freshness, and canonical location. A conforming registry is the system that verifies documents meet that specification, makes them discoverable, and answers a machine's questions about them via a queryable API.

A conforming registry typically operates as one or more of:

- A public, DNS-like service any agent can query without permission or payment
- A SaaS service for publishers registering their document corpus
- A private deployment for enterprise document estates

A conforming registry understands both MX YAML frontmatter and JSON-LD (Schema.org). It indexes full document metadata. An agent querying the registry receives: what the document is, who published it, when it was last signed, what version it is, and what it supersedes.

The distinction from GEO markup is architectural. GEO is passive - it adds tags to a page and depends on a crawler's behaviour. A registry is active - it provides a queryable interface an agent explicitly consults. The agent does not hope that metadata was noticed; it asks a registry and receives a verified answer.

A conforming registry attests one thing: this is the content the named publisher signed, unaltered. It makes no claim about whether the content is true, current in its subject matter, or sound in its reasoning. Editorial responsibility remains with the publisher.

---

## 7. Scope of this note

This note covers the following MX scope questions:

- What MX does (machine-readable documents across all formats and access pathways)
- What MX does not do (guarantee LLM citation, influence transformer token processing, promise specific machine behaviours)
- How a conforming registry complements MX governance (discovery, trust, currency)
- The standalone machine readability principle (a document must be interpretable in isolation)

This note does not cover:

- MX field definitions (see the MX Core Metadata note and related draft set)
- The registry-contract specification (a separate note covers the registry protocol)
- Implementation guidance for specific document formats (see Carrier Formats note)

---

## 8. Correct and incorrect claims about MX

### Correct

- "MX keeps content readable across every system that accesses it."
- "A document with full MX governance can be read, trusted, and located by any machine — in isolation."
- "A conforming registry provides explicit metadata that any agent can query directly."
- "Explicit structure reduces the need for inference, improving extraction reliability."
- "MX builds coverage across an unknowable landscape of machines and formats."

### Incorrect

- "MX makes your content AI-ready."
- "MX ensures LLMs cite your page."
- "MX eliminates hallucination."
- "Schema.org markup ensures AI discovery."
- "Future-proof your content for AI."

The distinction: correct claims describe coverage across an unknowable landscape without promising specific machine behaviour. Incorrect claims make promises about specific machine outputs that cannot be verified or guaranteed.

---

## 9. Relationship to other MX notes

This note is a scope clarification. It does not introduce new fields or change existing field definitions. It should be read alongside:

- **MX Core Metadata note** — defines the fields that implement these principles
- **MX Provenance note** — defines the attribution and trust fields that establish standalone readability
- **MX Carrier Formats note** — defines how MX metadata appears across document types
- **MX Contract Fingerprinting and Signing** — defines the cryptographic signing mechanism

---

*This note is offered to The Gathering for review and is not a ratified standard.*
