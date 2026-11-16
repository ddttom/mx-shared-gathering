---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Agent Directory Discovery note"
docname: draft-cranstoun-mx-agent-directory-discovery
date: 2026-11-16
consensus: false
keyword:
  - mx
  - agent-directory
  - llms.txt
  - discovery
  - describedby
  - link-relations
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-agent-directory-discovery.md
---

# MX Agent Directory Discovery note

**Version:** 1.1
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 16 November 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies how a host SHOULD expose an *agent-directory file* so that AI agents, and the tools that check and index on their behalf, can find it and read it as written. An agent-directory file is any well-known, host-rooted resource that describes the host to non-human readers. Examples include `llms.txt` (Jeremy Howard, September 2024), `robots.txt` (RFC 9309), `humans.txt` (humanstxt.org), and `ai.txt` (various proposals).

This note does not define or redefine any specific agent-directory file format. Each format remains owned by its own community. This note specifies a transport and discovery layer that any agent-directory file SHOULD adopt to be reliably reached by the systems it is intended to inform.

The note addresses two gaps. An agent-directory file wrapped in HTML, or served with a misleading content type, is no longer the text its readers expect to parse. And a file linked from no page on the host gives an agent that arrives on a deep page no signal that the file exists. The normative requirements in this note (plain-text or Markdown transport with no wrapper, no sitemap entry, and discovery through the registered `describedby` link relation, in line with the [llms.txt proposal](https://llmstxt.org/)) close both gaps without modifying the directory file's content.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A host claiming conformance to this note SHALL satisfy:

- **Level 1 (Transport):** the plain-text serving requirement in §4 and the sitemap requirement in §5.
- **Level 2 (Discovery):** Level 1 plus the `describedby` link requirement in §6.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Hosts implementing this draft SHOULD note the draft version number they implemented against.

---

## 3. Scope

### 3.1 In scope

- The transport mechanism by which an agent-directory file is delivered to its readers.
- The discovery mechanism by which agents learn the file exists.
- The in-page mechanism by which agents loading any HTML page on the host learn the file's location.
- The relationship between the agent-directory file and the host's `sitemap.xml`.

### 3.2 Out of scope

- The content, schema, or vocabulary of any specific agent-directory file. This note does not specify what `llms.txt` should contain; that is owned by the llms.txt community. The same applies to `robots.txt`, `ai.txt`, `humans.txt`, and any future agent-directory format.
- How training pipelines and bulk crawlers choose what to ingest. The llms.txt proposal expects `llms.txt` to be used mainly at inference time, on demand, when an agent needs information while assisting a user, rather than for training ([llms.txt proposal](https://llmstxt.org/)). This note serves that use.
- HTTP caching, CDN configuration, geographic routing, and other transport details unrelated to content type, link relations, and sitemap exclusion.

### 3.3 Relationship to existing standards

This note relies on:

- **[RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110)** for HTTP semantics, including `Content-Type`.
- **[RFC 7763](https://www.rfc-editor.org/rfc/rfc7763.html)** for the `text/markdown` media type.
- **[RFC 8288](https://datatracker.ietf.org/doc/html/rfc8288)** for Web Linking, including the HTTP `Link` header.
- **[IANA Link Relations registry](https://www.iana.org/assignments/link-relations/link-relations.xhtml)** for the `describedby` relation used in §6.
- **[Sitemaps 0.9 protocol](https://www.sitemaps.org/protocol.html)** for `sitemap.xml`.
- **[HTML Living Standard](https://html.spec.whatwg.org/)** for the `<link>` and `<meta>` element semantics used in §6.

This note adds nothing to those standards; it specifies how they combine for the agent-directory case.

---

## 4. Transport: serve as text/plain or text/markdown

A host conforming to this note SHALL serve every agent-directory file as the file's own text, with no HTML wrapper, and with one of:

- `Content-Type: text/markdown; charset=utf-8` ([RFC 7763](https://www.rfc-editor.org/rfc/rfc7763.html)) for a file written in Markdown, such as `llms.txt`;
- `Content-Type: text/plain; charset=utf-8` for any other agent-directory file, and for a Markdown file whose host prefers the plain-text type.

The canonical URL referenced by §6 SHALL NOT return `text/html`. The host SHALL NOT wrap, escape, or otherwise alter the file's text at serve time.

### 4.1 Why no wrapper

Agent-directory files are read by software that parses their text directly. An `llms.txt` file is Markdown with an H1 heading followed by lists of Markdown links ([llms.txt proposal](https://llmstxt.org/)); `robots.txt` is a line-based record format ([RFC 9309](https://datatracker.ietf.org/doc/html/rfc9309)). A wrapper adds markup the reader has to strip and escapes characters the format depends on, so the reader no longer receives the file as written.

Tools that check these files read the response body as the format itself. The Lighthouse `llms-txt` audit, for example, checks that the body has an H1 heading, at least one Markdown link, and at least 50 characters. It treats a 4xx response as not applicable and fails a 5xx or 429 response ([Lighthouse audit source](https://github.com/GoogleChrome/lighthouse/blob/main/core/audits/agentic/llms-txt.js)).

### 4.2 Status codes

The canonical URL of a published agent-directory file SHOULD return `200`. A host that does not publish a given file SHOULD return `404` for its conventional path, not a 5xx or `429` response.

### 4.3 Proxies and edge functions

Where a reverse proxy, edge worker, or CDN function sits in front of the origin, it SHALL pass the agent-directory file through unchanged. It MAY set the `Content-Type` in §4 where the origin does not, and MAY add the HTTP `Link` header described in §6.

---

## 5. Sitemap: do not list

A host SHALL NOT list an agent-directory file as a `<url>` entry in `sitemap.xml`.

### 5.1 Why not the sitemap

`sitemap.xml` lists a host's indexable, human-readable pages ([Sitemaps 0.9 protocol](https://www.sitemaps.org/protocol.html)). An agent-directory file is not one of those pages. The llms.txt proposal sets the two side by side: a sitemap lists all pages for search engines, while `llms.txt` offers a curated overview for language models ([llms.txt proposal](https://llmstxt.org/)). Agents find the file at its conventional path and through the `describedby` link in §6.

### 5.2 Hosts that already list the file

A host whose `sitemap.xml` lists an agent-directory file SHOULD remove the entry. The file's URL stays where it is.

---

## 6. Discovery: rel="describedby"

Every HTML page on the host SHOULD include in its `<head>` a `<link>` element that points to the agent-directory file covering it:

```html
<link rel="describedby" href="/llms.txt">
```

A host MAY instead, or as well, send the same relation as an HTTP `Link` header ([RFC 8288](https://datatracker.ietf.org/doc/html/rfc8288)):

```http
Link: </llms.txt>; rel="describedby"
```

The header form also works for non-HTML responses, such as the Markdown versions of pages, and can be added in web server or CDN configuration without modifying any page ([llms.txt proposal](https://llmstxt.org/)).

### 6.1 Why per-page links

Two discovery paths reinforce each other. The conventional path (`/llms.txt`, `/robots.txt`) covers readers that already know the convention. The `describedby` link covers everything else: agents that arrive via a deep link, JS-only single-page applications whose `<body>` is empty until script execution completes, and readers that do not probe conventional paths.

An `llms.txt` file covers the URLs under its path, and where more than one file applies, agents should use the most specific one ([llms.txt proposal](https://llmstxt.org/)). A host that publishes `llms.txt` at more than one path SHOULD point each page to the most specific file that covers it.

For headless and JavaScript-rendered sites the per-page link is particularly load-bearing. The `<head>` is delivered before any script runs and is therefore the only part of the response many crawlers and agents see.

### 6.2 The rel attribute

For `llms.txt`, the `rel` value SHALL be `describedby`, in line with the llms.txt proposal ([llms.txt proposal](https://llmstxt.org/)). `describedby` is registered in the IANA Link Relations registry, where it refers to a resource providing information about the link's context ([IANA Link Relations registry](https://www.iana.org/assignments/link-relations/link-relations.xhtml)). Hosts SHALL NOT use unregistered per-file values such as `llms-txt`.

For other agent-directory formats, a host SHOULD follow the format's own discovery convention where it has one, and MAY use `describedby` where it does not.

### 6.3 Optional inline description

A host MAY include alongside the link a `<meta>` element carrying a short prose description of the agent-directory file's content:

```html
<meta name="<directory-name>-description" content="<short prose description>">
```

This is a courtesy to agents that can act on a single tag without having to fetch the directory file itself.

### 6.4 Optional inline content

For hosts whose agent-directory file is short enough to embed in every page, a host MAY include the full content as a `<meta>` element:

```html
<meta name="<directory-name>-content" content="<full file content>">
```

This is RECOMMENDED only when the file is short (under approximately 1KB after escaping) and stable. For longer or frequently-updated directory files, prefer the link form (§6).

---

## 7. Why not depend on a new crawler standard

A host could reasonably ask why this note specifies plain-text transport and `describedby` discovery instead of asking crawlers to learn one new convention specific to agent-directory files. The answer is the principle on which the rest of the MX framework rests: **do not require new infrastructure when existing infrastructure already handles the problem**.

Readers already parse plain text and Markdown. The `describedby` relation is already registered, and HTML `<link>` elements and HTTP `Link` headers already carry it. Specifying that agent-directory files SHOULD use those mechanisms costs the host a one-time configuration change and costs the reader nothing at all. Specifying a new mechanism would require the host to wait for agents and tools to update before the directory file becomes useful.

This note is therefore not novel discovery infrastructure. It is a configuration discipline that makes existing discovery infrastructure work for a class of files that has, until now, fallen between the cracks.

---

## 8. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 8.1 Worker-based headers

A reverse-proxy worker (Cloudflare Worker, Fastly Compute, Vercel Edge Function, Akamai EdgeWorker, etc.) is well suited to the serve-time parts of this note. The worker passes the agent-directory file's body through from the origin unchanged, sets the `Content-Type` described in §4, and can add the `Link: </llms.txt>; rel="describedby"` header described in §6 to every response.

The header logic SHOULD be expressed as a pure function (request path in, headers out) so that it can be unit-tested independently of the worker runtime.

### 8.2 Sitemap automation

When the host generates `sitemap.xml` from a content tree, a single rule can exclude files at the host root that match the directory naming convention (`llms.txt`, `robots.txt`, `ai.txt`, `humans.txt`).

### 8.3 In-page link injection

For static-site generators, the `<link rel="describedby">` element SHOULD be added to the canonical page template. For server-rendered applications, it SHOULD be added to the layout component that wraps every page. For client-rendered applications (SPAs), it SHOULD be present in the static `index.html` shell, not injected by JavaScript at runtime, since runtime injection is invisible to readers that do not execute scripts. The HTTP `Link` header form (§6) needs no template change.

### 8.4 Verification

A host can verify Level 1 conformance in three checks:

1. Issue `curl -I` against the directory URL and check that the `Content-Type` header begins with `text/plain` or `text/markdown`.
2. Issue `curl` against the directory URL and check that the body is the file's own text, not a document beginning `<!DOCTYPE` or `<html`.
3. Check with `grep` that `sitemap.xml` has no entry for the directory URL.

Level 2 verification is a `grep` for `rel="describedby"` against the rendered `<head>` of any sample page, or a `curl -I` against a sample page that finds a `Link` header carrying `rel="describedby"`. For `llms.txt`, the Lighthouse `llms-txt` audit (§4.1) is an independent check of the body.

---

## 9. Security and privacy considerations

This note requires the agent-directory file to be served unaltered as `text/plain` or `text/markdown`, never as `text/html` (§4). A browser therefore does not interpret any `<` or `&` character in the directory text as markup, and no escaping step is needed. Hosts SHOULD also send `X-Content-Type-Options: nosniff` so that the declared type is respected.

The agent-directory file is, by design, public. This note adds no fields that carry confidential data. Hosts that wish to restrict access to the directory file should do so via HTTP authentication or IP allowlisting; that is a host policy, not a content-type concern.

The optional inline content form (§6.4) places the full directory content into every page response. Hosts SHOULD consider whether the directory's content is appropriate to repeat across the entire response surface, and SHOULD NOT use the inline form for content that is not safe to expose alongside any page on the host.

---

## 10. IANA considerations

This note requests no new IANA registrations. It uses the `describedby` relation already in the [IANA Link Relations registry](https://www.iana.org/assignments/link-relations/link-relations.xhtml) and the registered media types `text/plain` and `text/markdown`.

---

## 11. References

### 11.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119): Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174): Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110): HTTP Semantics.
- [RFC 7763](https://www.rfc-editor.org/rfc/rfc7763.html): The text/markdown Media Type.
- [RFC 8288](https://datatracker.ietf.org/doc/html/rfc8288): Web Linking.
- [IANA Link Relations registry](https://www.iana.org/assignments/link-relations/link-relations.xhtml).
- [Sitemaps XML format 0.9](https://www.sitemaps.org/protocol.html).
- [HTML Living Standard](https://html.spec.whatwg.org/).

### 11.2 Informative

- [llms.txt proposal (Jeremy Howard, 2024)](https://llmstxt.org/): the agent-directory format that motivated this note, and which recommends the `describedby` link relation.
- [RFC 9309](https://datatracker.ietf.org/doc/html/rfc9309): Robots Exclusion Protocol; the canonical example of an agent-directory file with established discovery and indexing conventions.
- [Lighthouse `llms-txt` audit source](https://github.com/GoogleChrome/lighthouse/blob/main/core/audits/agentic/llms-txt.js): an independent check of an `llms.txt` response body, cited in §4.1.

---

## 12. Acknowledgements

The author thanks the llms.txt community, whose work and conventions this note builds upon. Earlier prose treatment of the underlying problem appears at <https://mx.allabout.network/blog/llms-txt-guide.html>.
