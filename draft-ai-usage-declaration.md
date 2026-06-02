---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX AI Usage Declaration note"
docname: draft-cranstoun-mx-ai-usage-declaration
date: 2026-05-17
consensus: false
keyword:
  - mx
  - ai-usage
  - disclosure
  - provenance
  - well-known
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-ai-usage-declaration.md
---

# MX AI Usage Declaration note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 17 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies how a publisher declares the role AI played in producing a body of work. The declaration is a signed, scope-bounded, machine-readable statement carrying four facts: what the work is, who is responsible for it, what machines did during its production, and what machines did not do. It is published at a discoverable location on the publisher's host, available in both human-readable and machine-readable carriers, and may be self-attested or third-party-attested.

The note is motivated by a gap in the existing AI-adjacent web conventions. The Spawning Labs `ai.txt` and `robots.txt` carry publisher consent for *training* use of content. The `llms.txt` convention carries navigation aids for *inference-time* agents reading a site. C2PA manifests carry asset-level provenance for individual files. None of these answer the question a regulator, auditor, or attentive reader actually puts to a publisher: how, and to what extent, did AI participate in producing the content you are publishing? An AI Usage Declaration answers that question in a single document, in a form a person can read and a machine can verify.

The note is deliberately framework-agnostic about whether AI involvement is positive, negative, or neutral. It specifies disclosure, not judgement. It does not grant compliance with any AI-governance regime; it provides the kind of structured, machine-readable, tamper-evident record those regimes increasingly expect organisations to produce.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A declaration claiming conformance to this note SHALL satisfy:

- **Level 1 (Declared):** the declaration is published at a discoverable location (§5), carries every required field defined in §4, names the work it scopes, and names a responsible author. Self-attestation is the default authority.
- **Level 2 (Signed):** Level 1 plus a cryptographic signature over a canonical serialisation of the declaration payload, using the signature mechanism in §6.
- **Level 3 (Attested):** Level 2 plus a third-party attestation binding the declaration to a verified publisher identity, expressed as a compliance claim per the MX Compliance Claims note's third-party authority pattern.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Implementations of this draft SHOULD note the draft version number they implemented against.

---

## 3. Scope

### 3.1 In scope

- A machine-readable schema for a publisher AI Usage Declaration record.
- A discovery convention by which agents and humans locate the declaration for a given host or work.
- A signature mechanism that lets a verifier confirm the declaration has not been altered since signing.
- The relationship between this declaration and the existing AI-adjacent web conventions covering training opt-out, navigation aid, and asset-level provenance.

### 3.2 Out of scope

- Whether AI involvement in a given work is desirable, acceptable, or otherwise. The note specifies disclosure, not editorial judgement.
- Training data opt-out signalling, which is the territory of `ai.txt` proposals and `robots.txt` directives.
- Inference-time navigation aids for LLM agents reading a site, which is the territory of `llms.txt`.
- Asset-level provenance manifests for individual binary files (images, audio, video), which are the territory of C2PA.
- Model cards and other model-provider disclosures about the AI systems themselves, which are the territory of model documentation standards.
- Legal compliance with any specific AI-governance regime. A declaration is evidence; compliance is a legal duty of the organisation.

### 3.3 Relationship to existing standards

This note relies on:

- **[RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119)** and **[RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)** for normative key word interpretation.
- **[RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615)** for the well-known URI mechanism.
- **[RFC 7515 JSON Web Signature](https://datatracker.ietf.org/doc/html/rfc7515)** for the baseline signature serialisation when a declaration is signed.
- **[RFC 8032 EdDSA](https://datatracker.ietf.org/doc/html/rfc8032)** as the RECOMMENDED signature algorithm.
- **[ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)** for date and timestamp values.
- **[HTML Living Standard](https://html.spec.whatwg.org/)** for the `<link rel>` discovery mechanism.
- **[Schema.org](https://schema.org/)** for the publisher and author entity vocabulary referenced in §4, and for the `digitalSourceType` property used in the page-level interoperability mapping in §5.6.
- **[WICG AI Content Disclosure proposal](https://github.com/WICG/proposals/issues/261)** for the page-level `<meta name="ai-disclosure">` and element-level `ai-disclosure` attribute that a declaration's `aiUsage` array reduces to. The MX declaration is the publisher-level superset that produces the page-level value (§4.7).
- **[IPTC Digital Source Type vocabulary](https://cv.iptc.org/newscodes/digitalsourcetype/)** for the controlled-vocabulary terms referenced in the same mapping (`digitalCapture`, `compositeWithTrainedAlgorithmicMedia`, `trainedAlgorithmicMedia`).

This note adds nothing to those standards; it specifies how they combine for the publisher AI-usage disclosure case.

---

## 4. The declaration record

A declaration is authored as a markdown file with structured YAML frontmatter, following the long-established convention of YAML as the metadata block and CommonMark as the human-readable body. The markdown source is the source of truth at every conformance level; it is the artefact the author edits and the artefact a publisher commits to version control.

From this source file, three deterministic serialisations are derived:

- A canonical JSON form for machine transmission, interop, and signing (§6). The JSON is the artefact that gets signed at Level 2 because RFC 8785 JCS gives a deterministic canonical byte sequence over JSON, which YAML alone does not.
- A rendered HTML form for in-browser reading by humans.
- A tagged PDF form for accessible archival (optional but RECOMMENDED).

Each derived form MUST reflect the same fields as the source markdown; no field present in a serialisation MAY be absent from, or contradict, the source. A publisher MAY publish the source markdown itself at `/AI-USAGE.md` (§5.1) so any reader can fetch the editable form. The JSON, HTML, and PDF serialisations are siblings derived from the same source, not parallel artefacts edited in parallel.

The fields below are the ones the source markdown's YAML frontmatter MUST carry. They appear verbatim in the derived JSON (with the prose narrative from the markdown body summarised into the `aiBoundary` array). A conformant declaration MUST declare:

| Field | Cardinality | Type | Description |
| --- | --- | --- | --- |
| `id` | required | string (URI) | A stable URI that identifies this declaration. The URI SHOULD be the canonical URL at which the declaration is served. |
| `subject` | required | object | The work the declaration scopes. See §4.1. |
| `publisher` | required | object | The organisation or individual responsible for the work. See §4.2. |
| `author` | required | object | The person signing the declaration. May equal `publisher` when the publisher is an individual. See §4.3. |
| `created` | required | string (ISO 8601) | The date the declaration was first issued. |
| `modified` | RECOMMENDED | string (ISO 8601) | The date the declaration was last materially changed. Omit when equal to `created`. |
| `reviewSchedule` | OPTIONAL | string (ISO 8601 duration) | The interval at which the publisher commits to reviewing and re-confirming or updating the declaration. Example: `P1Y` for annual review. See §9.2. |
| `aiUsage` | required | array of objects | Enumerated tasks AI performed during production. See §4.4. |
| `aiBoundary` | required | array of strings | Tasks AI did not perform, stated as boundary claims. See §4.5. |
| `narrativeUri` | RECOMMENDED | string (URI) | Pointer to the human-readable rendering of the declaration, served on the same host. |
| `authority` | required at Level 2+ | object | Authority type and identity. See §4.6. |
| `signature` | required at Level 2+ | object | Cryptographic signature over the canonical payload. See §6. |

### 4.1 Subject object

The work the declaration scopes. A declaration MAY scope a single work (a book, a paper, a podcast episode, a film, a software release), a serial collection (a book series, a podcast feed, a periodical), or the entire output of a host. The `subject` object carries:

- `type`: one of `work`, `collection`, `host`. Required.
- `name`: human-readable name of the work, collection, or host. Required.
- `identifier`: a stable URI or ISBN, DOI, ISSN, or similar canonical identifier where one exists. RECOMMENDED.

A declaration that scopes a host (`type: host`) covers every publication on that host until superseded by a more specific declaration whose `subject` names a particular work or collection. Specific declarations override host-level declarations for the works they name.

### 4.2 Publisher object

The organisation or individual responsible for the work. The object SHOULD align with the Schema.org `Organization` or `Person` vocabulary, carrying at minimum:

- `type`: `Organization` or `Person`. Required.
- `name`: the publisher's name as it appears on the work. Required.
- `identity`: a URI (web page, DID, or other dereferenceable identifier). RECOMMENDED.

### 4.3 Author object

The person signing the declaration. The same shape as `publisher` but constrained to `Person`. The author signs in a personal capacity; the publisher accepts the signing person's authority to bind the publisher to the declaration.

### 4.4 AI-usage entries

An array of zero or more objects, each describing a category of AI involvement during production. Each object carries:

- `task`: a short noun phrase naming the task. Required. Values are open-ended in this version of the note. Implementations SHOULD use lowercase hyphen-separated phrases (e.g. `spelling-and-grammar`, `code-generation`). The common values listed below are the seed vocabulary; a community registry of task names is expected to emerge from implementation experience. Seed values: `research`, `spelling-and-grammar`, `consistency-checking`, `drafting`, `translation`, `summarisation`, `image-generation`, `code-generation`, `transcription`. Implementations that coin a novel value SHOULD document it in the declaration's `scope` field so that downstream readers can map it back to the closest seed term.
- `scope`: a one-sentence narrative description of how the AI was used for this task in this work. Required.
- `extent`: one of `assisted`, `substantial`, `primary`. Required. `assisted` means the AI's output was treated as a suggestion and reworked by a person; `substantial` means the AI produced a draft a person then directed and edited; `primary` means the AI produced the content with light human direction.

An empty `aiUsage` array is a valid declaration. It states that no AI was used in producing the work.

### 4.5 AI-boundary statements

An array of strings, each stating a boundary the publisher commits to: a task AI did not perform. Each string is a complete sentence in the publisher's voice. Common patterns include:

- "No machine decided what this work is about."
- "No machine wrote the text that carries the ideas."
- "No machine signed off on the published version."

The boundary array is normative for the publisher: a publisher that later relaxes a stated boundary MUST issue a new declaration with a later `modified` date and an updated boundary array.

### 4.6 Authority object

Authority type and identity. The object SHARES vocabulary with the MX Compliance Claims note for interoperability:

- `type`: `self` or `third-party`. Required.
- `identity`: URI or DID that names the signing party. Required.
- `accreditation`: present only when `type` is `third-party`. Object naming the accreditation body, validity dates, and a lookup URI for verification status.

Self-attested declarations carry the assurance level of the publisher's identity. They are appropriate for first-person disclosure by the work's author. They are not appropriate for evidence-trail use cases where the consumer's interest is in independent confirmation of the publisher's claim.

### 4.7 Mapping to WICG and IPTC vocabularies

Page-level and asset-level disclosure standards use finite enumerations. The MX declaration carries a richer per-task record. An MX-conformant implementation MUST derive the single-value WICG `ai-disclosure` and IPTC `digitalSourceType` for the declaration's `subject` from its `aiUsage` and `aiBoundary` by the following rules. The derivation is a function of the declaration record; it adds no new normative content.

A derived `disclosure` object on the declaration record MAY carry:

- `wicg`: one of `none`, `ai-assisted`, `ai-generated`, `autonomous`.
- `iptc`: one of `digitalCapture`, `compositeWithTrainedAlgorithmicMedia`, `trainedAlgorithmicMedia`.
- `derivedFrom`: the literal string `aiUsage+aiBoundary` to mark the value as computed rather than independently asserted.

The derivation rules:

| Condition | wicg | iptc |
| --- | --- | --- |
| `aiUsage` is empty (no AI involvement asserted) | `none` | `digitalCapture` |
| Every `aiUsage` entry has `extent: assisted` | `ai-assisted` | `compositeWithTrainedAlgorithmicMedia` |
| At least one `aiUsage` entry has `extent: substantial`, OR `extent: primary` with `aiBoundary` asserting human decision authority over the published version | `ai-generated` | `trainedAlgorithmicMedia` |
| Every `aiUsage` entry has `extent: primary` AND `aiBoundary` does not assert human decision authority over the published version | `autonomous` | `trainedAlgorithmicMedia` |

A declaration MAY publish the derived `disclosure` object alongside the underlying record. Consumers that only understand the WICG or IPTC vocabularies MAY rely on the derived values. Consumers that understand the MX schema SHOULD prefer the underlying `aiUsage` and `aiBoundary` for any reasoning that requires per-task granularity.

A publisher MUST NOT publish a derived `disclosure` value that contradicts the rules above. Manually overriding the derivation defeats the interoperability the mapping exists to provide.

The WICG value `mixed` is a page-level signal indicating that different elements on the same page carry different `ai-disclosure` attribute values. It does not arise from the publisher-level derivation in this section. A publisher that serves pages with the element-level WICG `ai-disclosure` attribute on individual elements MAY use `mixed` as the page-level meta value on those pages without this altering the publisher-level MX declaration. The MX `disclosure.wicg` field is restricted to the four values in the derivation table above.

### 4.8 Worked example

```json
{
  "id": "https://mx.allabout.network/AI-USAGE.json",
  "subject": {
    "type": "collection",
    "name": "MX book series",
    "identifier": "https://mx.allabout.network/books/"
  },
  "publisher": {
    "type": "Organization",
    "name": "CogNovaMX",
    "identity": "https://mx.allabout.network/about/about.html#organization"
  },
  "author": {
    "type": "Person",
    "name": "Tom Cranstoun",
    "identity": "https://mx.allabout.network/blog/profiles/about.tom.cranstoun.html#person"
  },
  "created": "2026-05-17",
  "aiUsage": [
    {"task": "research", "scope": "Gathering sources and prior work before forming a view.", "extent": "assisted"},
    {"task": "spelling-and-grammar", "scope": "Mechanical checks across long manuscripts.", "extent": "assisted"},
    {"task": "consistency-checking", "scope": "Term usage and structural coherence across chapters.", "extent": "assisted"}
  ],
  "aiBoundary": [
    "No machine decided what the books are about.",
    "No machine set the argument, chose the angle, or judged whether a passage was sound.",
    "No machine wrote the text that carries the ideas."
  ],
  "narrativeUri": "https://mx.allabout.network/AI-USAGE.html",
  "authority": {
    "type": "self",
    "identity": "https://mx.allabout.network/blog/profiles/about.tom.cranstoun.html#person"
  },
  "disclosure": {
    "wicg": "ai-assisted",
    "iptc": "compositeWithTrainedAlgorithmicMedia",
    "derivedFrom": "aiUsage+aiBoundary"
  }
}
```

This example illustrates the schema. It is not a registered declaration.

---

## 5. Discovery and serving

### 5.1 Canonical locations

A host that publishes an AI Usage Declaration SHOULD serve the derived forms at one of these canonical locations, and MAY also serve the source markdown alongside them:

- **Root-level (recommended):**
  - `/AI-USAGE.md` for the source markdown (OPTIONAL but RECOMMENDED for transparency).
  - `/AI-USAGE.json` for the machine-readable derived record (REQUIRED at Level 1).
  - `/AI-USAGE.html` for the human-readable rendering (REQUIRED at Level 1).
  - `/AI-USAGE.pdf` for the paginated archival form (OPTIONAL).
  - This pattern matches the conventional placement of `/llms.txt`, `/robots.txt`, `/humans.txt`, and `/ai.txt`.
- **Well-known prefix:** `/.well-known/ai-usage` (returning the JSON record) with content negotiation or sibling resources at `/.well-known/ai-usage.md`, `/.well-known/ai-usage.html`, and `/.well-known/ai-usage.pdf`. This pattern follows [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615) for hosts that prefer to keep root URIs clear.

A host MUST NOT serve different declarations at the root-level and well-known locations for the same `subject`. If both locations are served, they MUST point to identical declaration records (one MAY 301-redirect to the other). When the source markdown is served alongside its derived forms, the JSON, HTML, and PDF MUST be derivable from it with no contradictions; a derived form that contradicts the source is a conformance failure.

### 5.2 Content types

- The source markdown SHOULD be served with `Content-Type: text/markdown; charset=utf-8` per [RFC 7763](https://datatracker.ietf.org/doc/html/rfc7763).
- The JSON record SHOULD be served with `Content-Type: application/json; charset=utf-8`. The media type `application/ai-usage-declaration+json` is reserved for a future revision once implementation experience justifies registration.
- The human-readable HTML rendering SHOULD be served with `Content-Type: text/html; charset=utf-8`.
- A PDF rendering SHOULD be served with `Content-Type: application/pdf` and SHOULD be tagged per [ISO 14289-1](https://www.iso.org/standard/64599.html) (PDF/UA-1, `pdfuaid:part=1`) for accessibility.

### 5.3 In-page discovery

Every page on a host that publishes an AI Usage Declaration SHOULD carry a `<link>` element in `<head>` that points to the declaration:

```html
<link rel="ai-usage" href="/AI-USAGE.json" type="application/json">
```

The same page MAY carry additional `<link rel="alternate" type="...">` entries pointing to the HTML and PDF renderings. The kebab-case `rel` value `ai-usage` is proposed for a future link-relation registration; until registered, implementations MAY use the extension form `https://mx.allabout.network/ns/ai-usage` as an alternative `rel` value.

### 5.4 Sitemap inclusion

Every canonical location at which the declaration is served SHOULD appear in the host's sitemap. The JSON record and the HTML rendering SHOULD both be listed.

### 5.5 Per-work overrides

When the host-level declaration's `subject` is `host` and a specific work has its own narrower declaration, the work's pages SHOULD carry their own `<link rel="ai-usage">` pointing to the specific declaration. Agents MUST resolve to the more specific declaration when one is named for the work currently being read.

### 5.6 Page-level interoperability with WICG ai-disclosure

A page carrying a `<link rel="ai-usage">` to the publisher's MX declaration SHOULD also carry the page-level WICG `<meta name="ai-disclosure">` value derived from that declaration by the rules in §4.7. A page MAY additionally carry the Schema.org `digitalSourceType` property in its JSON-LD using the IPTC value derived by the same rules.

```html
<!-- discovery for the MX publisher-level declaration -->
<link rel="ai-usage" type="application/json" href="/AI-USAGE.json">
<!-- page-level WICG value derived from the MX declaration -->
<meta name="ai-disclosure" content="ai-assisted">
```

```json
{
  "@context": "https://schema.org",
  "@type": "CreativeWork",
  "digitalSourceType": "https://cv.iptc.org/newscodes/digitalsourcetype/compositeWithTrainedAlgorithmicMedia"
}
```

A page MUST NOT carry a WICG or IPTC value that contradicts the derivation from the publisher's MX declaration. Where the two would disagree, the publisher MUST either correct the MX declaration or correct the page-level value; both MUST NOT be served simultaneously with conflicting content.

A consumer that supports only the WICG meta tag or only the Schema.org property gets a useful page-level signal without having to fetch the MX declaration. A consumer that follows `<link rel="ai-usage">` gets the structured, signable, per-task record the page-level signal summarises.

---

## 6. Signature

### 6.1 Mechanism

A signed declaration (Level 2 or above) MUST carry a cryptographic signature over a canonical serialisation of the declaration payload, computed without the `signature` field present.

- The signature serialisation MUST follow [RFC 7515 JSON Web Signature](https://datatracker.ietf.org/doc/html/rfc7515).
- The signature algorithm SHOULD be Ed25519 ([RFC 8032 EdDSA](https://datatracker.ietf.org/doc/html/rfc8032)). Other algorithms MAY be used where the verifier's policy permits.
- The signature value MUST be base64url-encoded as defined in RFC 7515.
- The canonical form of the declaration payload over which the signature is computed MUST follow [RFC 8785 JSON Canonicalization Scheme](https://datatracker.ietf.org/doc/html/rfc8785). Implementations MAY additionally apply the layered fingerprinting approach described in the MX Contract Fingerprinting and Signing note (a sister draft in the MX family) for cross-cog content fingerprinting; the present note does not normatively depend on that approach.

### 6.2 `signature` object

The `signature` field of a signed declaration carries:

- `algorithm`: the JWS algorithm identifier (e.g. `EdDSA`). Required.
- `value`: the base64url-encoded signature. Required.
- `keyId`: a URI dereferenceable to the public key used to verify the signature. Required.
- `signedAt`: ISO 8601 timestamp the signature was produced. Required.

### 6.3 Re-signing

A signature is bound to a specific canonical serialisation. Any change to the declaration payload that affects the canonical form invalidates the signature. The publisher MUST issue a new signature when re-publishing.

---

## 7. Why a separate note

The publisher's AI-usage disclosure sits in a gap between adjacent standards that almost cover the case but do not.

**Why not WICG ai-content-disclosure alone.** The WICG AI Content Disclosure proposal carries a single page-level or element-level enumeration: `none`, `ai-assisted`, `ai-generated`, or `autonomous`. It is the right surface for a browser, a search engine, or a regulator's compliance check that needs a one-token answer for a single page. It does not carry which tasks AI performed, the human-authorship boundary the publisher commits to, or a signature that lets a verifier confirm the disclosure has not been altered. The MX declaration is the publisher-level superset whose summary reduces to the WICG value (§4.7 carries the mapping). A conformant MX implementation publishes both: the structured record at `/AI-USAGE.json` for verifiers, and the derived `<meta name="ai-disclosure">` on every page for consumers that read only the WICG vocabulary.

**Why not Schema.org `digitalSourceType` alone.** The IPTC Digital Source Type vocabulary, exposed in Schema.org as the `digitalSourceType` property, is a three-state controlled vocabulary for indicating how an asset came to exist. It is search-engine readable today and lives in the same JSON-LD block a page already serves. It addresses a strictly narrower question than this note. The MX declaration produces the IPTC value as a derived field (§4.7) so a publisher does not have to choose between the two.

**Why not C2PA.** The Coalition for Content Provenance and Authenticity specifies asset-level provenance manifests for individual binary files. C2PA answers the question "who made and altered this image, audio, or video, and how?" A book series, a website, a podcast feed, or a research programme has no single asset. A publisher-level disclosure of AI involvement across many works is a different layer of the same problem.

**Why not ai.txt.** The Spawning Labs `ai.txt` proposal carries publisher consent for AI *training* use of content. It is a directive to crawlers and dataset builders. It does not describe how the publisher themselves used AI in producing the content. The two declarations are orthogonal and may both be published by the same host.

**Why not robots.txt.** [RFC 9309](https://datatracker.ietf.org/doc/html/rfc9309) governs robots.txt. Its extension forms for AI-related directives are scoped to crawler permission. They do not carry publisher disclosure.

**Why not llms.txt.** The `llms.txt` convention carries navigation aids for inference-time LLM agents reading a site. It tells the agent where to look. It does not tell the agent how the content it is reading came to exist.

**Why not model cards.** Model documentation describes the AI system. An AI Usage Declaration describes a work produced with the help of one or more such systems. Both can be true at once; neither replaces the other.

**Why not a platform-specific AI-content checkbox.** Self-publishing platforms increasingly require authors to declare AI involvement at upload time. These declarations are not interoperable across platforms, not signed, not discoverable from the published work, and not durable beyond the platform's lifetime. A web-native declaration travels with the work.

**Why not EU AI Act Annex IV documentation alone.** Annex IV documentation is a regulatory record an organisation maintains. It is not a web-discoverable, machine-readable declaration attached to the published work. A jurisdiction-neutral declaration format complements the regulatory record without replacing it.

The combination of these gaps is the reason for a separate note.

---

## 8. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 8.1 Minimum viable implementation

A publisher beginning from scratch can reach Level 1 conformance in a single afternoon:

1. Write the human-readable narrative declaration. Name the subject, the publisher, the author, what AI did, what AI did not do.
2. Serialise the same facts into the JSON form defined in §4.
3. Compute the derived WICG and IPTC values per §4.7 and embed them in the JSON `disclosure` object.
4. Publish both at `/AI-USAGE.html` and `/AI-USAGE.json` on the publisher's host.
5. Add `<link rel="ai-usage" href="/AI-USAGE.json" type="application/json">` to the site template's `<head>`.
6. Add `<meta name="ai-disclosure" content="<derived-wicg-value>">` to the same template, and the Schema.org `digitalSourceType` property to the page's JSON-LD `CreativeWork` block.
7. Add both URLs to the host's sitemap.

Level 2 adds a signing step using any RFC 7515 JWS library that supports Ed25519. Level 3 requires arrangement with a third-party attestation authority operating under the MX Compliance Claims framework; this is a multi-week process and is not necessary for most publishers.

### 8.2 Worked implementation

An existing Level 1 implementation serves the publisher-level declaration for CogNovaMX (subject: the MX book series) at the following URLs:

- Machine-readable JSON record: <https://mx.allabout.network/AI-USAGE.json>
- Human-readable HTML rendering: <https://mx.allabout.network/AI-USAGE.html>
- Tagged PDF archival form (ISO 14289-1 Level 2): <https://mx.allabout.network/AI-USAGE.pdf>

Discovery is wired across the site: every page under `https://mx.allabout.network/` carries `<link rel="ai-usage" type="application/json" href="/AI-USAGE.json">` in `<head>`, along with the page-level `<meta name="ai-disclosure" content="ai-assisted">` derived per §4.7. The Schema.org `digitalSourceType` IRI `https://cv.iptc.org/newscodes/digitalsourcetype/compositeWithTrainedAlgorithmicMedia` is carried in the `WebPage` JSON-LD block on the declaration page itself. All three URLs appear in the host's sitemap.

The signature workflow (Level 2) is the next implementation step on this host. Reviewers can `curl` the JSON record to verify Level 1 conformance against the rules in §8.3.

### 8.3 Deriving the JSON from the source markdown

The JSON form is produced from the source markdown by a mechanical extraction. The recommended algorithm:

1. Parse the file's YAML frontmatter.
2. Drop document-meta fields not relevant to the declaration (`title`, `description`, `version`, and any document-mechanics fields not enumerated in §4). Keep the fields enumerated in §4.
3. If the frontmatter does not carry an explicit `aiBoundary` array, derive one from the boundary statements in the markdown body. The author SHOULD carry `aiBoundary` explicitly in frontmatter to avoid round-trip fragility.
4. Compute the derived `disclosure` object per §4.7 from the resulting `aiUsage` and `aiBoundary` values, and attach it to the JSON.
5. Serialise the resulting object to canonical JSON per [RFC 8785 JCS](https://datatracker.ietf.org/doc/html/rfc8785).

A reference implementation in any scripting language is approximately 30 lines. Implementations SHOULD publish the derivation script alongside the source markdown for reviewer transparency; the markdown is the editable artefact, the script is the contract that makes the JSON reproducible. At Level 2, the canonical JSON output of step 5 is the byte sequence over which the signature is computed (§6).

### 8.4 Verification

A consumer can verify Level 1 conformance with shell-level commands:

```bash
HOST=https://example.com
# 1. The declaration is discoverable
curl -sf "$HOST/AI-USAGE.json" > /tmp/decl.json
# 2. The required fields are present
jq -e '.id, .subject, .publisher, .author, .created, .aiUsage, .aiBoundary' /tmp/decl.json
# 3. The link relation is wired up on the homepage
curl -sf "$HOST/" | grep -E '<link[^>]+rel="ai-usage"'
```

Level 2 verification additionally validates the signature using a JWS library against the public key dereferenced from `signature.keyId`.

---

## 9. Security and privacy considerations

### 9.1 Substitution

An attacker who can replace the JSON declaration on a host can issue a false disclosure. Mitigation: serve Level 2 (signed) declarations whose signature a consumer can verify independently. A consumer that requires high assurance MUST verify the signature against an independently-resolved key, not against a key served from the same host.

### 9.2 Replay and staleness

A signed declaration is bound to its canonical payload, including the `created` and `modified` timestamps. A consumer that requires freshness SHOULD reject declarations whose `modified` is older than the consumer's policy permits. The note does not mandate a freshness window in general.

Publishers that carry a `reviewSchedule` field commit to issuing a new `modified` timestamp at least as frequently as that interval, whether the underlying disclosure has changed or merely been reviewed and re-confirmed. Consumers MAY treat a declaration whose `modified` is older than the stated schedule as stale and request a fresh one before relying on its content. A declaration without `reviewSchedule` makes no such commitment; consumers MUST NOT infer staleness from age alone where the schedule is absent.

### 9.3 Identity exposure

The `author` field names a person. Publishers SHOULD consider whether the named author wishes to be associated with the work in a machine-readable, durable, web-discoverable form. For works published anonymously or pseudonymously, the `author` field carries the pseudonym, and the publisher accepts the disclosure responsibility.

### 9.4 Boundary drift

The `aiBoundary` array states commitments. A publisher who later relaxes a boundary (for example, allowing AI to draft text that earlier declarations said it would not) MUST issue a new declaration with the boundary updated. Silently relaxing a boundary while leaving the old declaration in place is a misrepresentation. The note does not specify enforcement; it specifies the publisher's normative obligation.

### 9.5 Standard scope

These are operational concerns for hosts and consumers implementing the standard, not concerns for the standard itself.

---

## 10. IANA considerations

This note introduces three artefacts that fall within IANA registries:

- The link relation `ai-usage` (§5.3), candidate for the IANA Link Relation Types registry.
- The media type `application/ai-usage-declaration+json` (§5.2), candidate for the IANA Media Types registry.
- The well-known URI suffix `ai-usage` (§5.1), candidate for the IANA Well-Known URIs registry per [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615).

Formal IANA registration for each of these is deferred to a later revision of this note, pending sufficient implementation experience to justify the request. Implementations using the unregistered link relation MAY use the extension URI form `https://mx.allabout.network/ns/ai-usage` as the `rel` value until formal registration is complete. Implementations using the unregistered media type MAY serve `application/json` with a `profile` parameter, or use the vendor-prefixed form `application/vnd.mx.ai-usage-declaration+json`, until the unprefixed registration is granted.

---

## 11. References

### 11.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119): Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174): Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515): JSON Web Signature.
- [RFC 8032](https://datatracker.ietf.org/doc/html/rfc8032): Edwards-Curve Digital Signature Algorithm (EdDSA).
- [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615): Well-Known Uniform Resource Identifiers.
- [RFC 8785](https://datatracker.ietf.org/doc/html/rfc8785): JSON Canonicalization Scheme (JCS), the canonical form used at Level 2 for the declaration payload over which the signature is computed.
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html): Date and time format.
- [HTML Living Standard](https://html.spec.whatwg.org/): The HTML specification, for `<link rel>` semantics.

### 11.2 Informative

- [RFC 9309](https://datatracker.ietf.org/doc/html/rfc9309): Robots Exclusion Protocol.
- [Schema.org](https://schema.org/): Person, Organization, and CreativeWork.digitalSourceType vocabularies.
- [C2PA Specifications](https://c2pa.org/specifications/specifications/2.1/index.html): Coalition for Content Provenance and Authenticity, asset-level provenance manifests.
- [ISO 14289-1](https://www.iso.org/standard/64599.html): PDF/UA Universal Accessibility.
- [WICG AI Content Disclosure proposal](https://github.com/WICG/proposals/issues/261): page-level `<meta name="ai-disclosure">` and element-level `ai-disclosure` attribute, four-value enumeration `none` / `ai-assisted` / `ai-generated` / `autonomous`. W3C Community Group formed February 2026.
- [IPTC Digital Source Type vocabulary](https://cv.iptc.org/newscodes/digitalsourcetype/): controlled vocabulary for `CreativeWork.digitalSourceType` covering `digitalCapture`, `compositeWithTrainedAlgorithmicMedia`, `trainedAlgorithmicMedia`.
- [EU AI Act Article 50](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689): machine-readable marking of AI-generated content, effective August 2026.
- The MX Contract Fingerprinting and Signing note: a sister draft in the MX family that elaborates a layered fingerprinting approach for cross-cog content. Referenced informatively in §6.1; the present note does not normatively depend on it.
- The MX Compliance Claims note: a sister draft in the MX family that defines the third-party authority pattern referenced at Level 3 conformance (§2.1).
- [YAML 1.2.2](https://yaml.org/spec/1.2.2/): authoring format for the frontmatter block.
- [CommonMark 0.31.2](https://spec.commonmark.org/0.31.2/): authoring format for the human-readable body.
- [RFC 7763](https://datatracker.ietf.org/doc/html/rfc7763): media type `text/markdown` for serving the source markdown.
- llms.txt: Howard, J., September 2024, navigation aid convention for LLM agents.
- ai.txt: Spawning Labs proposal, training opt-out signalling.

---

## 12. Acknowledgements

The author thanks the publishers and authors whose first-person AI-usage disclosures, written without a standard to refer to, demonstrated the schema before it was specified. The note formalises the practice; it did not invent it.
