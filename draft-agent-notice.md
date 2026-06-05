---
title: "MX Agent Notice note"
docname: draft-cranstoun-mx-agent-notice
date: 2026-06-05
consensus: false
keyword:
  - mx
  - agent-notice
  - discovery
  - html
  - content-negotiation
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-agent-notice.md
---

# MX Agent Notice note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 5 June 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

MX metadata embedded in an HTML resource commonly lives in non-rendered markup: HTML comments, head `<meta>` elements, and JSON-LD `<script>` blocks. A large and growing class of automated consumers does not receive that markup. These consumers retrieve a page through an HTML-to-text or HTML-to-markdown conversion stage, then read the converted text. That stage discards head markup and non-rendered content. The consumer is left with the title and the visible body, and the machine-readable metadata is silently lost. The consumer is given no signal that the metadata ever existed, and no instruction for retrieving it.

The failure mode is acute because the lost reader is the reader MX exists to serve. A page authored to be machine-readable becomes, to a converting consumer, indistinguishable from a page that carries no metadata at all. The author believes the resource is self-describing; the consumer sees prose.

This note specifies an **agent notice**: a short signal carried in a channel that survives HTML-to-text conversion (visible rendered body text), stating that machine-readable metadata for the resource exists and naming how to retrieve the full-fidelity source. The note does not change what metadata a page carries or where the canonical copy lives; it ensures the page degrades to an actionable instruction when every non-visible channel is stripped. It builds on published infrastructure only: visible HTML text, the `alternate` link relation, HTTP content negotiation, and WAI-ARIA roles.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A host serving an HTML resource and claiming conformance to this note SHALL satisfy one of the following levels (modelled on the [WCAG 2.1](https://www.w3.org/TR/WCAG21/) Level A/AA/AAA pattern):

- **Level 1 (Visible Notice):** The resource carries a visible agent notice, in rendered body text, that states machine-readable metadata for the resource exists and gives a retrieval instruction a reader can follow.
- **Level 2 (Resolvable Source):** Level 1, plus the canonical full-fidelity source is named by an absolute URL that appears as visible text inside the notice, and is also declared in markup with the published `alternate` link relation so consumers that read raw markup can resolve it without parsing prose.
- **Level 3 (Negotiable Source):** Level 2, plus the full-fidelity source is retrievable from the resource's own URL by HTTP content negotiation, so a conforming agent obtains the metadata without following a second, separately guessed URL.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Implementations of this draft SHOULD note the draft version number they implemented against.

---

## 3. Scope

### 3.1 In scope

- The requirement that a machine-readable HTML resource degrade to an actionable, visible signal when non-rendered channels are stripped.
- The channel in which that signal MUST be carried.
- The minimum content of the signal.
- The markup affordances (`alternate` link relation, supplementary head metadata) and the content-negotiation behaviour that raise a notice to Levels 2 and 3.

### 3.2 Out of scope

- The vocabulary and field semantics of the metadata itself. Those are defined by the MX Core Metadata note and the carrier-specific notes.
- The syntactic envelope each file format uses to host MX metadata. That is owned by the MX Carrier Formats note.
- How an agent discovers a site's entry points or directory of resources. That is owned by the MX Agent Directory Discovery note.
- Any guarantee about the internal behaviour of a specific third-party fetcher or conversion pipeline. This note specifies what the publisher emits, not what any consumer does with it.

### 3.3 Relationship to existing standards

This note relies on:

- **[RFC 8288](https://www.rfc-editor.org/rfc/rfc8288)** (Web Linking) and the [IANA Link Relations registry](https://www.iana.org/assignments/link-relations/) for the `alternate` link relation used to point at the full-fidelity source.
- **[RFC 9110](https://www.rfc-editor.org/rfc/rfc9110)** (HTTP Semantics) for proactive content negotiation via the `Accept` request header.
- **[WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria-1.2/)** for the `note` role used to mark the notice for assistive technology and for deterministic location by automated consumers.

This note adds nothing to those standards; it specifies how they combine so that a machine-readable HTML resource remains addressable after a lossy fetch. For orientation only, and without normative weight: the MX Carrier Formats note describes how MX metadata is hosted inside HTML, and the MX Agent Directory Discovery note describes how an agent finds resources in the first place.

---

## 4. The survival requirement

### 4.1 Channels that MUST be assumed lossy

An implementation MUST assume that any of the following channels MAY be removed before the resource reaches an automated consumer:

- HTML comments, including any comment carrying MX source frontmatter.
- Head `<meta>` elements.
- `<script type="application/ld+json">` blocks and other `<script>` content.

A conformance claim under this note MUST NOT depend on any of these channels reaching the consumer. They remain valuable for consumers that read raw markup and SHOULD continue to be emitted, but they are supplementary under this note, never the sole signal.

### 4.2 The channel that MUST carry the notice

The agent notice MUST be carried as visible rendered body text: text that a conforming HTML user agent renders into the document's text flow. The notice MUST NOT be carried only in a channel enumerated in §4.1, and MUST NOT be hidden from rendering by `display:none`, `visibility:hidden`, the `hidden` attribute, `aria-hidden="true"`, off-screen positioning, or zero-size containment. A notice that is not part of the rendered text flow does not satisfy this note, because the conversion stage that motivates the note removes exactly such content.

### 4.3 Minimum content of the notice

The visible agent notice MUST:

- State that machine-readable metadata exists for this resource; and
- Name a retrieval method that yields the full-fidelity source.

The notice SHOULD name the canonical source as an absolute URL rendered as visible text, so the URL survives conversion to plain text. Where the retrieval method is content negotiation (§6.3), the notice SHOULD state the request header and value a consumer sends.

---

## 5. The visible notice element

### 5.1 Markup

The notice SHOULD be a single grouping element (for example an `aside`) carrying the WAI-ARIA `role="note"`, so that assistive technology presents it as an aside to the main content and an automated consumer parsing markup can identify its purpose. The element MUST contain human-readable text; an icon or a bare link alone does not satisfy §4.3.

### 5.2 Stable identification

The element SHOULD carry a stable hook attribute, `data-mx-agent-notice`, valued either empty or with the conformance level reached. Consumers that do read raw markup can then locate the notice deterministically rather than by text matching. The attribute is an identification aid only; presence of the attribute does not by itself satisfy any requirement of §4.

### 5.3 Human burden

The notice SHOULD be brief and MAY be visually de-emphasised by styling, provided it remains in the rendered text flow as required by §4.2. The notice serves both a human reader who arrives without context and a machine reader behind a lossy conversion stage; one short sentence with a named source URL serves both.

---

## 6. Machine-parseable affordances

The affordances in this section raise a Level 1 notice to Levels 2 and 3. They are complementary to the visible notice and, per §4.1, MUST NOT be the only signal a resource provides.

### 6.1 The alternate link (Level 2)

The resource SHOULD declare its full-fidelity source with the `alternate` link relation, naming the source media type and absolute URL, for example a `link` element in the head with `rel="alternate"`, a `type` attribute carrying the source media type, and an `href` attribute carrying the absolute source URL. The same absolute URL MUST also appear as visible text inside the notice (§4.3), so a consumer that loses the head still obtains it.

### 6.2 Supplementary head metadata

The resource MAY additionally restate the notice in a head `<meta>` element for consumers that read raw markup. This is supplementary. A resource MUST NOT rely on it to satisfy §4.

### 6.3 Content negotiation (Level 3)

For Level 3, the origin SHOULD serve the full-fidelity source from the resource's own URL in response to an appropriate `Accept` request header, per [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) proactive negotiation. A conforming agent that sends the negotiated header then obtains the metadata from the same URL it already holds, with no second URL to guess. The visible notice SHOULD state the header and value to send.

---

## 7. Rationale

### 7.1 Why not rely on head metadata, JSON-LD, or comments

These channels are the natural home for machine-readable metadata, and they reach consumers that fetch raw markup. They do not reach consumers that read a converted text rendering, and that population is large and growing. A framework whose purpose is to stop machines guessing cannot place its only signal in a channel that the most relevant class of machine never receives. This note does not deprecate those channels; it requires that they not be the sole signal.

### 7.2 Why not define a new crawler standard

A standard cannot retroactively change the behaviour of deployed fetchers and conversion pipelines. It can only ensure the resource degrades to an actionable instruction under the behaviour that already exists. This note therefore adds discipline to existing infrastructure rather than proposing new infrastructure: visible text already survives, the `alternate` relation is already registered, content negotiation is already specified.

### 7.3 Why visible text

Visible rendered body text is the one channel demonstrated to survive HTML-to-text and HTML-to-markdown conversion. A controlled test (see §8.3) confirms which channels survive a given converter; in the converters observed, only the title and visible body text passed through. Placing the signal there is the only placement that meets the consumer where it actually reads.

---

## 8. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 8.1 Edge injection

An edge layer can inject the notice at serve time from the resource's own metadata, keeping the source documents free of presentation concerns. The injected element is identical for every resource except the canonical URL and, optionally, the negotiated header to advertise.

### 8.2 Build-time injection

A static-site build step can emit the notice from frontmatter as the page is generated, so the notice is present in the stored artefact and survives a serve path that does no rewriting.

### 8.3 Verification

A host can confirm the surviving-channel claim with a controlled fixture rather than by inspection of a live page, where convention-guessable values can give a false positive. Place one unique, unguessable token in each candidate channel (one in an HTML comment, one in a head `<meta>`, one in a JSON-LD block, one in a visible aside, one in plain body text). Serve the fixture over public HTTPS with a `text/html` content type. Retrieve it through the target conversion tool and record, for each token, whether it appears in the returned text. The tokens reported present identify the channels that survived. Repeat against each consumer class of interest.

To confirm a deployed page rather than a fixture: fetch the page through the conversion tool and confirm that the notice text and the canonical source URL both appear in the returned text; then fetch the canonical source directly and confirm it carries the full metadata.

---

## 9. Security and privacy considerations

The agent notice exposes the URL of the full-fidelity source. A publisher MUST NOT place a private, access-controlled, or otherwise non-public URL in a publicly served notice; doing so would advertise a resource the publisher intends to restrict. The canonical source URL SHOULD be same-origin, or on an explicitly trusted host, so that the notice does not become an open redirection or a pointer to attacker-controlled content.

The notice text and any URL it carries MUST be escaped according to HTML rules, because the notice is, by construction, rendered content and a frequent target for injection if assembled from untrusted input.

The notice reveals that structured metadata exists for the resource. This disclosure is intentional and is the purpose of the note. It adds no exposure beyond what the metadata itself provides once fetched; a publisher unwilling to disclose that metadata exists should not be publishing it in a public resource.

---

## 10. IANA considerations

This note proposes no new IANA registrations. It reuses the `alternate` link relation already in the [IANA Link Relations registry](https://www.iana.org/assignments/link-relations/) and existing registered media types. A future revision MAY propose a dedicated link relation for a machine-readable source; this draft deliberately defers that to avoid introducing a registration before implementation experience justifies it.

---

## 11. References

### 11.1 Normative

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for use in RFCs to indicate requirement levels.
- [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) - Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288) - Web Linking; the `alternate` link relation.
- [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) - HTTP Semantics; proactive content negotiation.
- [WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria-1.2/) - Accessible Rich Internet Applications; the `note` role.
- [IANA Link Relations registry](https://www.iana.org/assignments/link-relations/) - registered link relation types.

### 11.2 Informative

- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) - Web Content Accessibility Guidelines; the A/AA/AAA level pattern this note's conformance levels follow.
- MX Carrier Formats note - how MX metadata is hosted inside HTML and other carriers.
- MX Agent Directory Discovery note - how an agent discovers a site's resources.

---

## 12. Acknowledgements

This note was motivated by a recorded session in which an automated agent, reading a machine-readable page through a converting fetch tool, could not see the metadata the page carried until directed to retrieve the raw source by hand. The gap, once observed, generalises to every machine-readable HTML resource fetched through a lossy conversion stage.
