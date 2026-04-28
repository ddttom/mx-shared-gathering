---
title: "MX Agent Directory Discovery note"
docname: draft-cranstoun-mx-agent-directory-discovery
date: 2026-04-28
consensus: false
keyword:
  - mx
  - agent-directory
  - llms.txt
  - discovery
  - common-crawl
  - sitemap
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Agent Directory Discovery note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 28 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies how a host SHOULD expose an *agent-directory file* so that AI training pipelines, search indexers, and agent crawlers can discover and ingest it. An agent-directory file is any well-known, host-rooted resource that describes the host to non-human readers. Examples include `llms.txt` (Jeremy Howard, September 2024), `robots.txt` (RFC 9309), `humans.txt` (humanstxt.org), and `ai.txt` (various proposals).

This note does not define or redefine any specific agent-directory file format. Each format remains owned by its own community. This note specifies a transport, sitemap, and in-page discovery layer that any agent-directory file SHOULD adopt to be reliably reached by the systems it is intended to inform.

The note is motivated by an observable gap: well-formed agent-directory files are routinely invisible to large-scale crawlers because they are served as `text/plain` (which Common Crawl and similar HTML-only pipelines do not ingest), are absent from `sitemap.xml`, and are not linked from any page in the host. The three normative requirements in this note (HTML transport, sitemap inclusion, in-page discovery link) close that gap without modifying the directory file's content.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A host claiming conformance to this note SHALL satisfy:

- **Level 1 (Transport):** the HTML-serving requirement in §4.
- **Level 2 (Discovery):** Level 1 plus the sitemap requirement in §5.
- **Level 3 (Resilience):** Level 2 plus the in-page link requirement in §6.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Hosts implementing this draft SHOULD note the draft version number they implemented against.

---

## 3. Scope

### 3.1 In scope

- The transport mechanism by which an agent-directory file is delivered to crawlers.
- The discovery mechanism by which crawlers learn the file exists.
- The in-page mechanism by which agents loading any HTML page on the host learn the file's location.
- The relationship between the agent-directory file and the host's `sitemap.xml`.

### 3.2 Out of scope

- The content, schema, or vocabulary of any specific agent-directory file. This note does not specify what `llms.txt` should contain; that is owned by the llms.txt community. The same applies to `robots.txt`, `ai.txt`, `humans.txt`, and any future agent-directory format.
- The behaviour of inference-time agents that retrieve the directory file on demand. This note addresses training-pipeline and bulk-crawler discovery; inference-time retrieval has its own discovery patterns.
- HTTP caching, CDN configuration, geographic routing, and other transport details unrelated to content negotiation and sitemap inclusion.

### 3.3 Relationship to existing standards

This note relies on:

- **[RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110)** for HTTP semantics, including `Content-Type` negotiation.
- **[Sitemaps 0.9 protocol](https://www.sitemaps.org/protocol.html)** for `sitemap.xml`.
- **[HTML Living Standard](https://html.spec.whatwg.org/)** for the `<link>` and `<meta>` element semantics used in §6.
- **[Schema.org WebPage](https://schema.org/WebPage)** as the structured-data vocabulary for the optional JSON-LD block in §4.3.

This note adds nothing to those standards; it specifies how they combine for the agent-directory case.

---

## 4. Transport: serve as text/html

A host conforming to this note SHALL serve every agent-directory file with `Content-Type: text/html; charset=utf-8`. The host MAY additionally serve the file at a separate URL with `Content-Type: text/plain` for legacy clients, but the canonical URL referenced by §5 and §6 SHALL return `text/html`.

### 4.1 Why HTML

The largest training-data crawlers (Common Crawl is the canonical example, ingested by most major LLM training pipelines) only index HTML responses. A file served as `text/plain` is fetched but excluded from the resulting HTML corpus. The agent-directory file therefore does not reach training data, even though the file exists, is well-formed, and is reachable by direct URL.

### 4.2 Wrapper requirements

The HTML wrapper SHALL:

1. Preserve the agent-directory file's text content verbatim. Implementations SHOULD place the text inside a `<pre>` element to preserve whitespace.
2. HTML-escape any `<`, `>`, and `&` characters in the directory text before insertion into the wrapper.
3. Include a `<title>` element. If the directory text begins with a heading recognisable in its own format (e.g. a Markdown `# Heading` line in `llms.txt`), the wrapper SHOULD promote that heading to `<title>`. Otherwise the wrapper SHOULD use a fallback of the form `<directory-filename>, <hostname>`.
4. Include `<meta charset="utf-8">` and `<meta name="viewport" content="width=device-width, initial-scale=1">`.
5. Include `<meta name="robots" content="index, follow">`. The point of the wrapper is to be indexed.
6. Include `<link rel="canonical" href="<bare-resource-url>">` where `<bare-resource-url>` is the agent-directory file's URL with query string and fragment stripped.

### 4.3 Optional structured data

The wrapper MAY include a Schema.org JSON-LD block describing itself as a `WebPage`. When present, the block SHOULD include at minimum:

- `@context: https://schema.org`
- `@type: WebPage`
- `name`: the title computed in §4.2 step 3
- `description`: a short prose description identifying the resource as an agent-directory file
- `url`: the canonical URL from §4.2 step 6

This block is OPTIONAL because Schema.org is an extension; the transport requirement (§4) does not depend on it.

### 4.4 Wrapping at serve time

The wrapper SHOULD be applied at serve time (e.g. by a reverse proxy, edge worker, or CDN function) rather than by modifying the source file. Wrapping at serve time keeps the source agent-directory file in its native plain-text form for tools that parse it directly, while delivering the HTML form to HTTP clients that request it.

---

## 5. Discovery: list in sitemap.xml

If the host publishes a `sitemap.xml` (per the Sitemaps 0.9 protocol), the host SHALL include each agent-directory file as a `<url>` entry in `sitemap.xml`.

### 5.1 Why sitemap inclusion

Crawlers that discover URLs via `sitemap.xml` (the dominant pattern for large-scale crawl) have no reliable signal that an agent-directory file exists if it is absent from the sitemap. Root-directory speculative fetches are not universal and not guaranteed; sitemap inclusion is.

### 5.2 Sitemap entry shape

The sitemap entry SHALL identify the agent-directory file's canonical URL (the `text/html` URL from §4). The entry MAY include `<lastmod>`, `<changefreq>`, and `<priority>` per the Sitemaps protocol.

### 5.3 Hosts without sitemap.xml

A host that does not publish `sitemap.xml` is not in scope for §5. Such hosts SHOULD reach Level 1 (transport) and Level 3 (in-page link) instead; the sitemap requirement of Level 2 is moot when no sitemap exists.

---

## 6. In-page resilience: <link rel="..."> in every page

Every HTML page on the host SHOULD include in its `<head>` a `<link>` element referencing each agent-directory file the host publishes:

```html
<link rel="<directory-name>" href="/<directory-filename>">
```

### 6.1 Why per-page links

Three discovery paths reinforce each other. Sitemap inclusion (§5) covers crawlers that read sitemaps. HTML serving (§4) makes the file ingestible once reached. The in-page link covers everything in between: agents that arrive via a deep link, JS-only single-page applications whose `<body>` is empty until script execution completes, and crawlers that do not read the sitemap.

For headless and JavaScript-rendered sites the per-page link is particularly load-bearing. The `<head>` is delivered before any script runs and is therefore the only part of the response many crawlers and agents see.

### 6.2 The rel attribute

The `rel` attribute value SHALL be the bare directory name without the `.txt` extension. Example values:

- `rel="llms-txt"` for `/llms.txt`
- `rel="ai-txt"` for `/ai.txt`
- `rel="humans-txt"` for `/humans.txt`

Using kebab-case for the rel value matches existing IANA link relation conventions (e.g. `rel="dns-prefetch"`, `rel="alternate-stylesheet"`).

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

This is RECOMMENDED only when the file is short (under approximately 1KB after escaping) and stable. For longer or frequently-updated directory files, prefer the link form (§6) plus sitemap inclusion (§5).

---

## 7. Why not depend on a new crawler standard

A host could reasonably ask why this note specifies HTML transport, sitemap inclusion, and `<link>` discovery instead of asking crawlers to learn one new convention specific to agent-directory files. The answer is the principle on which the rest of the MX framework rests: **do not require new infrastructure when existing infrastructure already handles the problem**.

Crawlers already understand HTML responses. Sitemaps already signal what matters on a host. The `<link>` element already carries arbitrary `rel` values that machine readers can dispatch on. Specifying that agent-directory files SHOULD use those mechanisms costs the host a one-time configuration change and costs the crawler nothing at all. Specifying a new mechanism would require the host to wait for the crawler ecosystem to update before the directory file becomes useful, which (per the inference-time observation in §3.2) it largely is not anyway.

This note is therefore not novel discovery infrastructure. It is a configuration discipline that makes existing discovery infrastructure work for a class of files that has, until now, fallen between the cracks.

---

## 8. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 8.1 Worker-based wrapping

A reverse-proxy worker (Cloudflare Worker, Fastly Compute, Vercel Edge Function, Akamai EdgeWorker, etc.) is well suited to serve-time wrapping. The worker intercepts the agent-directory URL, fetches the source plain-text file from the origin, applies the wrapper described in §4, and returns the result with `Content-Type: text/html`.

The wrapper logic SHOULD be expressed as a pure function (string in, string out) so that it can be unit-tested independently of the worker runtime.

### 8.2 Sitemap automation

When the host generates `sitemap.xml` from a content tree, the agent-directory entry can be added by a single rule that detects files at the host root matching the directory naming convention.

### 8.3 In-page link injection

For static-site generators, the `<link>` element SHOULD be added to the canonical page template. For server-rendered applications, it SHOULD be added to the layout component that wraps every page. For client-rendered applications (SPAs), it SHOULD be present in the static `index.html` shell, not injected by JavaScript at runtime, since runtime injection is invisible to crawlers that do not execute scripts.

### 8.4 Verification

A host can verify Level 1 conformance by issuing `curl -I` against the directory URL and checking that the `Content-Type` header begins with `text/html`. Level 2 verification is a `grep` against `sitemap.xml`. Level 3 verification is a `grep` against the rendered `<head>` of any sample page.

---

## 9. Security and privacy considerations

The wrapper described in §4 ingests the agent-directory file's text and emits it as part of an HTML document. Implementations MUST HTML-escape the text (§4.2 step 2) to prevent any `<` or `&` character in the directory text from being interpreted as markup.

The agent-directory file is, by design, public. This note adds no fields that carry confidential data. Hosts that wish to restrict access to the directory file should do so via HTTP authentication or IP allowlisting; that is a host policy, not a content-type concern.

The optional inline content form (§6.4) places the full directory content into every page response. Hosts SHOULD consider whether the directory's content is appropriate to repeat across the entire response surface, and SHOULD NOT use the inline form for content that is not safe to expose alongside any page on the host.

---

## 10. IANA considerations

This note proposes no new IANA registrations. The `<link rel>` values described in §6 are agent-directory-specific identifiers that do not require IANA registration; they are dispatched on by name within consuming software.

If the agent-directory community wishes to register specific rel values (e.g. `llms-txt`) with the IANA Link Relation Type registry, that registration is owned by the directory's defining community, not by this note.

---

## 11. References

### 11.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) — Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) — Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110) — HTTP Semantics.
- [Sitemaps XML format 0.9](https://www.sitemaps.org/protocol.html).
- [HTML Living Standard](https://html.spec.whatwg.org/).

### 11.2 Informative

- [llms.txt proposal (Jeremy Howard, 2024)](https://llmstxt.org/) — the agent-directory format that motivated this note.
- [RFC 9309](https://datatracker.ietf.org/doc/html/rfc9309) — Robots Exclusion Protocol; the canonical example of an agent-directory file with established discovery and indexing conventions.
- [Schema.org](https://schema.org/) — structured-data vocabulary used in §4.3.
- [Common Crawl](https://commoncrawl.org/) — the open web crawl ingested by most major LLM training pipelines.

---

## 12. Acknowledgements

The author thanks the llms.txt and Common Crawl communities, whose work and conventions this note builds upon. Earlier prose treatment of the underlying problem appears at <https://mx.allabout.network/blog/llms-txt-guide.html>.
