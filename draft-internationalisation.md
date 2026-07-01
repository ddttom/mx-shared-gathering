---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Internationalisation and Translation note"
docname: draft-cranstoun-mx-internationalisation
date: 2026-07-01
consensus: false
keyword:
  - mx
  - internationalisation
  - localisation
  - language
  - currency
  - translation
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-internationalisation.md
---

# MX Internationalisation and Translation note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 1 July 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note defines how an MX document declares the natural-language and locale facts a machine needs to read it correctly, and how a translated document is linked to its source. It covers four things a document meets an agent without: the language of its content, the currency of any money it names, the convention by which its human-facing dates are written, and, when the document is a translation, a verifiable pointer back to the work it was translated from.

MX does not invent vocabulary where a stable standard exists. This note defers to BCP 47 for language, ISO 4217 for currency, ISO 8601 for machine-read dates, and the Schema.org translation properties for translation lineage. It fills only the gap those standards leave: a single, document-level place to declare each fact, carried in the document's own metadata so it travels with the file.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

Field definitions in this note conform to the MX Field Definition Pattern note. This note adopts the pattern's authoring rules and does not restate them. The fields defined here are externally-aligned fields in the sense of the MX Core Metadata note §7: their semantics are owned by, or modelled after, an established external vocabulary, recorded per field below.

---

## 3. Language

### 3.1 inLanguage

| Property | Value |
|----------|-------|
| Type | string |
| Requirement | SHOULD (MUST on a translation) |
| Aligns with | Schema.org `inLanguage`, Dublin Core `dc:language` |

`inLanguage` declares the primary natural language of the document's content as a language tag conforming to [BCP 47](https://www.rfc-editor.org/rfc/rfc5646.html) (for example `en`, `es`, `en-GB`, `pt-BR`). A document not written in its estate's or publisher's default language SHOULD declare `inLanguage`; a translation (see §6) MUST declare it. The value is the same tag Schema.org `inLanguage` and Dublin Core `dc:language` carry, so an MX document projects into either vocabulary without transformation.

`inLanguage` describes document content. It is distinct from the HTML `lang` attribute, which a carrier sets on its own root element; a well-formed HTML carrier of an MX document sets both to the same tag.

---

## 4. Currency

### 4.1 currency

| Property | Value |
|----------|-------|
| Type | string |
| Requirement | MAY |
| Aligns with | ISO 4217 (the code Schema.org `priceCurrency` uses) |

`currency` declares the document's default currency for monetary references, as an [ISO 4217](https://www.iso.org/iso-4217-currency-codes.html) three-letter code (for example `EUR`, `USD`, `GBP`). It is a document-level default that lets a machine resolve an unqualified amount in the body; it does not replace a per-price currency where one is stated explicitly (an offer's `priceCurrency`, a table cell's own code). A document that names money in more than one currency SHOULD qualify each amount at the point of use rather than rely on the default.

---

## 5. Dates

### 5.1 Machine-read dates are ISO 8601 (Normative)

Every date in an MX document's metadata MUST be written in [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) form: `YYYY-MM-DD`, or `YYYY-MM-DDTHH:MM:SSZ` with an explicit timezone for a timestamp. This is unconditional. A machine reading a metadata date never has to disambiguate a value like `01/07/2026`, because the metadata never carries that form.

### 5.2 dateFormat

| Property | Value |
|----------|-------|
| Type | string |
| Requirement | OPTIONAL |
| Valid values | `iso-8601` (default), `dmy`, `mdy`, `ymd` |

`dateFormat` declares the convention by which dates are written in the document's human-facing body, so a machine reading a rendered date in prose can resolve its order. It does not affect metadata dates, which are always ISO 8601 per §5.1. When absent, `iso-8601` is assumed. A document that displays dates to a reader in a locale convention (for example `dmy` for a UK letter) SHOULD declare `dateFormat` so the body remains machine-resolvable.

---

## 6. Translation

A translation is a distinct document, with its own metadata, linked to the work it was translated from. The link is bidirectional and carries a provenance record of the translation event.

### 6.1 translationOfWork

| Property | Value |
|----------|-------|
| Type | string (URI) |
| Requirement | MUST on a translation |
| Aligns with | Schema.org `translationOfWork` |

On a translated document, `translationOfWork` carries the `canonicalUri` of the source work it was translated from. It is the machine's route from a translation back to the authoritative original.

### 6.2 workTranslation

| Property | Value |
|----------|-------|
| Type | array (URI) |
| Requirement | SHOULD on a translated source |
| Aligns with | Schema.org `workTranslation` |

On a source document, `workTranslation` lists the `canonicalUri` of each known translation of it. It is the inverse of `translationOfWork`. A source SHOULD declare its translations so an agent arriving at the original can find the reader's language without a separate search.

### 6.3 translator

| Property | Value |
|----------|-------|
| Type | string |
| Requirement | SHOULD on a translation |
| Aligns with | Schema.org `translator` |

`translator` names who or what performed the translation: a person's name, or a model identifier when the translation was produced by a machine. It is the accountable party for the translation's fidelity.

### 6.4 Translation provenance (Normative)

A translation MUST record the translation event as a provenance step in its provenance record, distinct from the linkage fields above. The step names, at minimum, the source document, the source content hash, the source language, and the target language, and declares whether a human reviewed the result. The linkage fields say that a translation relationship exists; the provenance step says who created it, from which exact bytes of the source, and when, so the relationship is verifiable rather than merely asserted.

---

## 7. Carrier projection

These fields project into each MX carrier by the rules of the MX Carrier Formats note. In HTML, `inLanguage` and the translation properties map to Schema.org JSON-LD on the document's primary entity, and `inLanguage` additionally informs the carrier's `lang` attribute. In a YAML sidecar or frontmatter, the fields sit in the MX operational block. In a PDF, they travel in the XMP packet alongside the rest of the MX metadata.

---

## 8. Security and privacy considerations

Declaring a document's language, currency, and translations exposes no information the document's content does not already reveal. A `translator` value that names an individual is personal data and follows the same handling as any other named party in MX metadata.

---

## 9. References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for use in RFCs
- [BCP 47 / RFC 5646](https://www.rfc-editor.org/rfc/rfc5646.html) - Tags for Identifying Languages
- [ISO 4217](https://www.iso.org/iso-4217-currency-codes.html) - Currency codes
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) - Date and time format
- [Schema.org inLanguage](https://schema.org/inLanguage), [translationOfWork](https://schema.org/translationOfWork), [workTranslation](https://schema.org/workTranslation), [translator](https://schema.org/translator)
