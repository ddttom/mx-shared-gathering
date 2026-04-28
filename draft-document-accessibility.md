---
title: "MX Document Accessibility note"
docname: draft-cranstoun-mx-document-accessibility
date: 2026-04-28
consensus: false
keyword:
  - mx
  - accessibility
  - document
  - pdf
  - pdf-ua
  - pdf-a
  - eaa
  - wcag
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Document Accessibility note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 28 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies how a publisher SHOULD declare and demonstrate the accessibility of a non-HTML document carrier so that AI training pipelines, search indexers, agent crawlers, and assistive technologies (screen readers, screen magnifiers, switch interfaces, refreshable braille displays) can reach, parse, and present the document on equal terms with sighted readers using a standard browser.

The note is motivated by a regulatory and demographic gap. Around one in four people in the European Union has some form of disability; the figure is approximately one in six globally. Document carriers other than HTML (PDF being the dominant case, with an installed base of several trillion files and four hundred billion opened in a single vendor's tooling in a single year) are routinely shipped without the structure tags, logical reading order, or language declarations that assistive technologies require. The European Accessibility Act (Directive (EU) 2019/882) entered into force on 28 June 2025 and applies financial penalties of approximately ten thousand euros for minor violations and one hundred thousand euros or more for major or repeated non-compliance, with significantly higher figures available for large enterprises.

The note does not redefine PDF, DOCX, EPUB, or any other carrier format. Each format remains owned by its own standards body (ISO 32000 for PDF, ISO/IEC 29500 for OOXML, IDPF for EPUB). The note does not redefine accessibility either. WCAG 2.1, EN 301 549, and the per-format accessibility standards (ISO 14289 PDF/UA, the WCAG PDF Techniques) remain authoritative. This note specifies a metadata layer that declares which of those standards the artefact claims to satisfy, and a verification discipline that distinguishes a self-declared claim from an independently checked one.

The note is carrier-agnostic. PDF is treated normatively in section 4 because the regulatory pressure and installed base make it the urgent case. DOCX (section 5) and EPUB (section 6) are described informatively with the same conformance levels and a documented migration path so that the same metadata declarations work across all three formats without per-carrier reinvention.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A publisher claiming conformance to this note for a given artefact SHALL satisfy:

- **Level 1 (Tagged):** the carrier embeds a structure tree and a logical reading order in its format-native way. For PDF that is a `/StructTreeRoot` per ISO 14289-1 with `/MarkInfo /Marked true` and a populated reading-order tree. For DOCX it is the use of paragraph styles for headings (Heading 1, Heading 2, etc.) plus alternative text on every non-decorative inline image. For EPUB it is semantic XHTML5 with `epub:type` attributes on structural elements.
- **Level 2 (Declared):** Level 1 plus the source artefact (or its sidecar metadata) carries the open-standard MX fields `accessibilityConformance`, `accessibilityFeature`, `accessibilityHazard`, `accessibilitySummary`, and `lang`. The carrier's native metadata mirrors the declarations where the carrier supports it (PDF XMP `pdfuaid:part` and `dc:language`; DOCX `core.xml` language; EPUB `dc:language` and `meta property="schema:accessibilityFeature"`).
- **Level 3 (Verified):** Level 2 plus the result of an independent accessibility check is recorded in the artefact's provenance metadata (`provenancePedigree.checks[]` array, with each entry naming the checker, the checker's version, the date of the check, and the outcome). The note does not name specific checkers; verifier categories include PDF/UA validators, WCAG PDF Techniques runners, and equivalent checkers for DOCX and EPUB.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Implementations of this draft SHOULD note the draft version number they implemented against.

---

## 3. Scope

### 3.1 In scope

- The metadata fields a publisher SHOULD attach to a non-HTML document artefact to declare its accessibility characteristics.
- The mapping from those metadata fields to each carrier's native metadata representation (XMP for PDF, core.xml for DOCX, OPF metadata for EPUB).
- The verification discipline that distinguishes a self-declared claim (Level 2) from an independently checked claim (Level 3).
- The relationship between a carrier-format declaration (e.g. `accessibilityConformance: PDF/UA-1`) and the underlying ISO standard the declaration refers to.

### 3.2 Out of scope

- The format-internal mechanics of how to embed a structure tree, alt text, or reading order in PDF, DOCX, or EPUB. Those mechanics are owned by ISO 14289 (PDF/UA), ISO 32000 (PDF), the OOXML accessibility annex, and the EPUB accessibility specification.
- The pedagogical question of what makes good alt text. Each carrier-format community has its own guidance (W3C alt-text decision tree, EPUB accessibility guidelines, Microsoft Office accessibility checker rules); this note defers to them.
- HTML accessibility. WCAG 2.1, the WAI-ARIA specification, and the existing HTML semantics already cover HTML; this note does not duplicate them. The MX Carrier Formats note (the sister Gathering draft for code carriers) and the broader MX accessibility narrative in the published MX manuscripts cover HTML separately.
- Procurement-side compliance language for the European Accessibility Act, the Americans with Disabilities Act, the UK Equality Act, or any other jurisdiction-specific regulation. This note describes the technical metadata; the regulatory question of which artefacts fall under which jurisdiction is owned by the publisher and their legal counsel.

### 3.3 Relationship to existing standards

This note relies on:

- **[ISO 14289-1](https://www.iso.org/standard/64599.html)** (PDF/UA-1) for PDF accessibility.
- **[ISO 32000-2](https://www.iso.org/standard/75839.html)** (PDF 2.0) for the underlying PDF specification.
- **[ISO 19005](https://www.iso.org/standard/63534.html)** series (PDF/A) for archive-grade PDF, which shares much of the accessibility tagging machinery.
- **[WCAG 2.1](https://www.w3.org/TR/WCAG21/)** and the [W3C PDF Techniques](https://www.w3.org/WAI/WCAG21/Techniques/#pdf) for the cross-format accessibility success criteria.
- **[BCP 47](https://www.rfc-editor.org/info/bcp47)** for the language tags used in the `lang` field and in carrier-native language declarations.
- **[Schema.org accessibility properties](https://www.w3.org/2021/a11y-discov-vocab/latest/)** (`accessibilityFeature`, `accessibilityHazard`, `accessibilitySummary`, `accessibilityAPI`, `accessibilityControl`) as the canonical vocabulary for the MX `accessibilityFeature`, `accessibilityHazard`, and `accessibilitySummary` fields.
- **[Directive (EU) 2019/882](https://eur-lex.europa.eu/eli/dir/2019/882/oj)** (the European Accessibility Act) as the regulatory context that motivates publisher adoption.

This note adds nothing to those standards; it specifies how their metadata combines for the document-format accessibility case.

---

## 4. PDF (normative)

A PDF carrier claiming conformance to this note SHALL satisfy the following requirements at the level claimed.

### 4.1 Level 1: Tagged PDF (normative)

The PDF SHALL embed a `/StructTreeRoot` per ISO 14289-1 §7.1, populated with a tag tree that mirrors the document's logical structure (headings, paragraphs, lists, tables, figures, captions, links). The PDF SHALL declare itself tagged via `/MarkInfo << /Marked true >>` in the document catalog.

The tag tree SHALL define a logical reading order distinct from the visual layout order. Multi-column documents (newspaper layouts, two-column scientific papers, side-by-side translation panels) MUST set the reading order so that screen readers traverse columns in the author's intended sequence rather than across each visual row.

Every non-decorative image, figure, formula, or other non-text content element SHALL carry alternative text via the `/Alt` entry on the structure element. Decorative content (borders, dividers, purely visual ornament) SHALL be marked as artefacts (`/Artifact`) so that assistive technologies skip them.

### 4.2 Level 2: Declared (normative)

The source artefact (or its sidecar metadata, where the PDF is generated from a source carrying MX frontmatter) SHALL declare:

- `accessibilityConformance` set to `PDF/UA-1` or `PDF/UA-2`.
- `accessibilityFeature` listing the Schema.org accessibility features the PDF carries (e.g. `taggedPDF`, `readingOrder`, `alternativeText`, `displayTransformability`).
- `accessibilityHazard` listing any Schema.org accessibility hazards present, or the negative declarations (`noFlashingHazard`, `noSoundHazard`, `noMotionSimulationHazard`) where the publisher knows the PDF is hazard-free.
- `accessibilitySummary` carrying a one-paragraph human-readable description of the PDF's accessibility characteristics.
- `lang` carrying the BCP 47 language tag of the PDF's primary language.

The PDF's XMP packet SHALL mirror the declarations where the XMP namespace supports them: `pdfuaid:part` MUST equal the integer part of the declared `accessibilityConformance` value (1 for `PDF/UA-1`, 2 for `PDF/UA-2`); `dc:language` MUST equal the `lang` value.

A PDF carrying these declarations without satisfying Level 1 is non-conformant. Validators SHALL reject the claim and SHOULD report the missing structure-tree or reading-order element specifically rather than a generic failure.

### 4.3 Level 3: Verified (normative)

An independent accessibility check SHALL have run against the PDF byte-stream, and the check's outcome SHALL be recorded in the artefact's provenance metadata under `provenancePedigree.checks[]`. Each entry in the array SHALL carry at minimum:

- `checker`: the name (and where applicable the version) of the checker that ran.
- `standard`: the standard the checker validated against (e.g. `PDF/UA-1`, `WCAG-2.1-AA`).
- `date`: the ISO 8601 date the check ran.
- `outcome`: `pass`, `pass-with-warnings`, or `fail`.

The note does not name specific checkers; verifier categories include PDF/UA validators, WCAG PDF Techniques runners, and accessibility-checker tools shipped by major office and graphics vendors.

A `fail` or `pass-with-warnings` outcome MAY still be Level 3 conformant if the publisher records the outcome honestly. The point of Level 3 is verifiability, not perfection.

---

## 5. DOCX (informative)

A DOCX carrier conforms to this note at Level 1 by using the document's heading styles consistently for headings (Heading 1, Heading 2, etc. rather than directly-formatted bold text), by carrying alternative text on every non-decorative inline image (the alt-text dialogue in mainstream office authoring tools), and by setting the document's primary language in `core.xml`.

Level 2 declarations are carried in the same way as for PDF: the source artefact (the DOCX itself, since DOCX is its own source) carries the MX accessibility fields in either a sidecar metadata file or in the `core.xml` extended-properties block. The DOCX accessibility checker tools provided by mainstream office suites can serve as Level 3 verifiers when their outcome is recorded in `provenancePedigree.checks[]`.

The OOXML accessibility annex covers the format-internal mechanics. This note does not duplicate it.

---

## 6. EPUB (informative)

An EPUB carrier conforms to this note at Level 1 by using semantic XHTML5 throughout, with `epub:type` attributes on structural elements (chapter, part, glossary, etc.) per the EPUB Structural Semantics Vocabulary, by carrying alternative text on every non-decorative image, and by declaring the language in the OPF `dc:language` element.

Level 2 declarations are carried in the OPF metadata block as `meta` elements with `property="schema:accessibilityFeature"`, `property="schema:accessibilityHazard"`, and `property="schema:accessibilitySummary"`, mirroring the MX field names. The EPUB accessibility specification (EPUB Accessibility 1.1) covers the format-internal mechanics.

---

## 7. Why three reinforcing layers

A publisher could reasonably ask why this note specifies three conformance levels rather than a single binary "is it accessible".

The answer is that accessibility claims fail in three distinct ways and each layer catches a different one. A PDF can be byte-level tagged (Level 1) but carry no machine-readable declaration of what standard it claims, so a downstream consumer has to parse the file to find out (the Level 2 problem). A PDF can carry an explicit `accessibilityConformance: PDF/UA-1` declaration in its source frontmatter and its XMP packet but be untagged at the byte level (the Level 1 problem; the declaration is a lie). A PDF can be tagged and declared but never independently checked, so the publisher's claim is unverified (the Level 3 problem; trust is asserted but not earned).

The three layers reinforce each other: structural truth (Level 1), declared truth (Level 2), and verified truth (Level 3). A publisher claiming Level 3 is making the strongest claim available without this note proposing a new verification infrastructure; the note relies entirely on existing checker tooling and existing standards.

The European Accessibility Act, the Americans with Disabilities Act, the UK Equality Act, and analogous regulations in other jurisdictions create a legal incentive for Level 3 specifically: a self-declaration without independent verification carries less defensive weight than an independently-checked one.

This note is therefore not novel infrastructure. It is a metadata discipline that makes existing accessibility infrastructure auditable.

---

## 8. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 8.1 Generation pipelines

Most publishers generate PDFs from a higher-level source (Markdown, LaTeX, Word, web-to-PDF). The accessibility-conscious choice is to generate tagged output by default rather than retrofit tagging onto an untagged production file.

For Markdown-to-PDF pipelines, a typesetting engine that supports the `tagpdf` package (lualatex with `\usepackage{tagpdf}`) produces tagged output natively. Alternatives include weasyprint and chromium-headless print-to-PDF, both of which produce tagged output by default for HTML input.

For Word-to-PDF pipelines, the office suite's "Save As PDF" or "Export as PDF" function respects the source document's heading styles and alt text when accessibility is enabled at export time.

### 8.2 Verification

A heuristic Level 1 check on a PDF can be performed with `qpdf --json <pdf>` (qpdf is widely available open-source tooling) and inspecting the output for `/StructTreeRoot`, `/MarkInfo /Marked true`, and `/Lang` entries on the document catalog, plus the XMP `pdfuaid:part` declaration. A heuristic check is not a Level 3 verification; Level 3 requires a checker that validates the tag tree's correctness against the standard, not just the presence of the byte-level structures.

A full Level 3 verifier for PDF/UA-1 is available from several open-source and commercial sources. The verifier identity SHOULD be recorded in `provenancePedigree.checks[]` so a downstream reader can re-run the same verifier and reproduce the outcome.

### 8.3 Authoring-tool alt text

Mainstream authoring tools and several AI-assisted authoring services now suggest alt text automatically. Suggested alt text is often technically correct but lacks document context. A common failure mode: a photograph of a named person is described as "older businessman looking professional" rather than the person's name and role. A bar chart is described as "blue bars on a white background" rather than the dataset and the comparison the chart conveys.

Authors SHOULD review every auto-suggested alt-text value before shipping. Authors SHOULD mark genuinely decorative images with `decorative: true` (the MX field) so that authoring tools and verifiers know to skip them rather than emit alt text the screen reader would then announce as noise.

### 8.4 Scope: which documents need conformance

Not every artefact needs to satisfy this note. Internal-only documents, ephemeral working notes, and machine-generated artefacts that are never read by a human or an assistive technology are out of practical scope.

The artefacts that benefit most from conformance are customer-facing documents published to the open web (white papers, product datasheets, academic preprints, regulatory filings, government forms) and high-touch internal documents that must be readable by every employee regardless of disability (HR policies, company handbooks, training materials, payroll documents, signed contracts).

---

## 9. Security and privacy considerations

The fields described in this note carry no confidential data. Accessibility declarations are by their nature public claims about an artefact intended for public consumption.

The provenance entries recorded at Level 3 (`provenancePedigree.checks[]`) name the checker tool and outcome. Publishers SHOULD consider whether any checker-emitted diagnostic detail (e.g. a list of specific tag-tree errors on a failed check) is appropriate to expose alongside the artefact. The recommended pattern is to record the outcome in the public provenance and keep the diagnostic detail in private build logs.

The declarations described here do not expose any user data, do not require the artefact to track or fingerprint readers, and do not create any new attack surface beyond the carrier format's own.

---

## 10. IANA considerations

This note proposes no new IANA registrations. The Schema.org accessibility properties referenced in this note are defined and maintained by Schema.org under their published process; the carrier-format-specific properties (`pdfuaid:part`, `pdfaid:part`, `dc:language`) are defined and maintained by their respective standards bodies.

---

## 11. References

### 11.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) — Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) — Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [BCP 47](https://www.rfc-editor.org/info/bcp47) — Tags for identifying languages.
- [ISO 14289-1](https://www.iso.org/standard/64599.html) — Document management applications: Electronic document file format enhancement for accessibility (PDF/UA-1).
- [ISO 32000-2](https://www.iso.org/standard/75839.html) — Document management: Portable document format (PDF 2.0).
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines.

### 11.2 Informative

- [ISO 19005](https://www.iso.org/standard/63534.html) series — PDF/A archive-grade PDF.
- [W3C PDF Techniques](https://www.w3.org/WAI/WCAG21/Techniques/#pdf) — Accessibility techniques for PDF.
- [Schema.org accessibility properties](https://www.w3.org/2021/a11y-discov-vocab/latest/) — Accessibility metadata vocabulary.
- [EPUB Accessibility 1.1](https://www.w3.org/TR/epub-a11y-11/) — Accessibility specification for EPUB publications.
- [Directive (EU) 2019/882](https://eur-lex.europa.eu/eli/dir/2019/882/oj) — European Accessibility Act.
- [EN 301 549](https://www.etsi.org/deliver/etsi_en/301500_301599/301549/) — Accessibility requirements for ICT products and services (the EU public-sector procurement standard).

---

## 12. Acknowledgements

The author thanks the PDF/UA, WCAG, and EPUB accessibility communities, whose decades of work this note builds on. The note's three-level conformance shape is modelled on the WCAG Level A / AA / AAA pattern and on the MX Agent Directory Discovery note's transport / discovery / resilience structure.
