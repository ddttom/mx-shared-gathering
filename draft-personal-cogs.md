---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Personal Cogs note"
docname: draft-cranstoun-mx-personal-cogs
date: 2026-10-04
consensus: true
keyword:
  - mx
  - personal
  - privacy
  - disclosure
  - identities
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-personal-cogs.md
---

# MX Personal Cogs note

**Version:** 1.0
**Status:** Ratified by The Gathering on 4 October 2026
**Date:** 4 October 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

Most MX documents describe something published: a menu, a product, a policy, a page. A **personal cog** describes the person reading them. It is a record a person keeps about themselves (an access need, an allergy, a consent choice, a preference, a skill) and holds on their own device. A reader application matches it against the documents it meets, on the device, so no profile of the person is built anywhere else.

Without a shared vocabulary, every reader application invents its own answer to the questions that matter most for such a record: what it is about, which of the person's facets it belongs to, how far it may travel, and whether its earlier versions survive a revision. A record carried between applications then loses the very limits that made it safe to write down.

This note defines a `personal` profile, the record kind `personal-cog`, and seven fields in Zone 2 (the `mx:` object): `personalKind`, `disclosureTier`, `personalIdentity`, `revisionTraces`, `disclosesAs`, `discloseFor` and `declaredBy`.

The model is the palimpsest: a manuscript written over an earlier one that was scraped away, though never completely, so traces of the first writing remain. A personal cog is the layer a person writes over the content they receive. The publisher's attested layer stays beneath it and can still be verified. The personal layer directs what the person is shown; it never alters what the publisher signed.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A personal cog SHALL satisfy:

- **Level 1 (Declared):** the record carries `type: personal-cog`, the Zone 1 identity fields, `personalKind` and `disclosureTier`.
- **Level 2 (Scoped):** Level 1 plus `personalIdentity` and `revisionTraces`, and, when `disclosureTier` is `derived`, a non-empty `disclosesAs`.
- **Level 3 (Bounded):** Level 2 plus, when `disclosureTier` is `guarded`, a non-empty `discloseFor`.

A reader application conforms when it applies §5 to every personal cog it holds, whatever level the cog reaches.

### 2.2 Ratification status

The Gathering ratified this note at version 1.0 on 4 October 2026. A later change to its text re-opens review and is published as a new version. Implementations SHOULD note the version they implemented against.

---

## 3. Scope

### 3.1 In scope

- The record a person keeps about themselves, and the limits on how far it travels.
- The behaviour a reader application owes those limits when it matches, renders, sends a prompt to a model, or discloses to another party.

### 3.2 Out of scope

- The values of individual preferences (a marketplace, a currency, a broadcast category). They live in the body of a `preferences` or `reception` cog, or in a vendor extension; this note describes the record and its disclosure, not every preference a person may hold.
- Transport, storage encryption and device synchronisation. A personal cog is a file; how it is synchronised is a deployment choice. §4.4 states the one obligation synchronisation carries.
- Inference about a person. No field in this note holds a value a system inferred from behaviour or location.

### 3.3 Relationship to existing standards

- The record kind is a value of the OKF `type` field, and the profile follows the model the **MX Core Metadata note** defines.
- `personalKind` values align with the data categories of the GDPR in one direction only: `health` is a special category (Article 9). This note grants no legal basis for processing, and conformance to it is not compliance with any regulation.
- The neutral preferences of `disclosesAs` are rendering preferences, in the spirit of the WCAG 2.2 user-preference media features (`prefers-reduced-motion` and similar), never the condition behind them.

> **Note:** This section describes regulatory frameworks in general terms only. Nothing here is legal advice. Requirements vary by jurisdiction, organisation type, and use case. Consult qualified legal specialists for guidance specific to your situation.

---

## 4. Field definitions

All fields live in Zone 2 (the `mx:` object). A reader MUST NOT read them from the top level of the frontmatter.

### 4.1 `personalKind`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST (Level 1) |
| **Valid values** | accessibility, consent, dietary, health, interests, preferences, reception, skills |

**Definition:** What a personal cog declares about its owner.

```yaml
mx:
  personalKind: dietary
```

- `reception` is a push-reception policy: which broadcast categories the person accepts, and how often.
- `preferences` covers locale, marketplace and display choices, language and currency among them.
- A record that needs two kinds SHOULD be two cogs, so each carries its own `disclosureTier`.

### 4.2 `disclosureTier`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST (Level 1) |
| **Valid values** | private, derived, guarded |

**Definition:** How far a personal cog may travel from its owner's device.

- **`private`**: the cog never leaves the device. It shapes what the person sees and nothing else.
- **`derived`**: only the neutral preferences named in `disclosesAs` leave the device, in place of the cog.
- **`guarded`**: the cog MAY leave the device whole, only on a transaction the owner starts and only through the reader application's disclosure step (§5). The purpose MUST be one named in `discloseFor`; when `discloseFor` is absent, it MUST be the one the owner confirms at that moment.

```yaml
mx:
  personalKind: accessibility
  disclosureTier: derived
  disclosesAs: [audio-first, described-images]
```

- A reader that finds no `disclosureTier`, or a value outside the valid set, MUST treat the cog as `private`.
- A cog with `personalKind: health` SHOULD be `private` or `derived`.

### 4.3 `personalIdentity`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |
| **Default** | base |

**Definition:** The identity that holds a personal cog. Identity names are the owner's own (personal, consultant, author, or any other); the value `base` places the cog in every identity.

```yaml
mx:
  personalKind: skills
  personalIdentity: consultant
  disclosureTier: guarded
  discloseFor: [client-meeting]
```

- Only the active identity's cogs, and `base` cogs, take part in a match. One identity MUST NOT be used to profile another.
- Where a reader also organises cogs by folder, the declared `personalIdentity` takes precedence over the directory that holds the file.

### 4.4 `revisionTraces`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | SHOULD (Level 2) |
| **Valid values** | kept, scraped |
| **Default** | kept |

**Definition:** Whether earlier versions of a personal cog stay recoverable by its owner after a revision (`kept`) or are erased when it is revised (`scraped`).

```yaml
mx:
  personalKind: health
  disclosureTier: private
  revisionTraces: scraped
```

- Traces are for the owner alone. Only the current version of a cog is matched or disclosed, whatever this field says.
- When a cog is `scraped`, the reader application MUST NOT retain an earlier version on the device. A synchronisation mechanism it controls MUST NOT carry one forward either.

### 4.5 `disclosesAs`

| Property | Value |
|----------|-------|
| **Type** | array of string |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST when `disclosureTier` is `derived` (Level 2); MAY otherwise |

**Definition:** The neutral preferences that travel in place of a `derived` cog, each saying what the content should do rather than why the owner needs it.

- Values are rendering preferences such as `audio-first`, `caption-ready`, `reduced-motion`, `plain-language`, `described-images`.
- A value MUST NOT name a condition, a diagnosis or a cause.
- A `derived` cog with no `disclosesAs` discloses nothing.

### 4.6 `discloseFor`

| Property | Value |
|----------|-------|
| **Type** | array of string |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST when `disclosureTier` is `guarded` (Level 3); MAY otherwise |

**Definition:** The kinds of transaction for which a `guarded` cog may be disclosed, in the owner's own words (`dining`, `travel`, `client-meeting`).

- The list narrows disclosure and never widens it: a `private` cog stays private whatever this lists.
- With no `discloseFor`, a reader MUST ask the owner before each disclosure and name the purpose. The confirmation covers that purpose and that transaction only. It does not extend `discloseFor`, and a reader MUST NOT reuse it later.

### 4.7 `declaredBy`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Conformance** | MAY |
| **Valid values** | self, delegate |
| **Default** | self |

**Definition:** Who made the declaration: `self` for the owner, or `delegate` for someone acting with the owner's authority, such as a carer or a parent. A `delegate` declaration SHOULD name the delegate as a `parties` entry with role `delegate`.

---

## 5. Reader behaviour

A reader application that holds personal cogs:

1. MUST treat any destination off the owner's device as a disclosure. A venue, a server, another person's phone and a hosted language model are each off the device. A model running on the device is not.
2. MUST NOT disclose during discovery. Hearing a beacon, scanning a code or loading a page discloses nothing; disclosure happens only on a transaction the owner starts.
3. MUST apply `disclosureTier` before any other sharing rule, as §4.2 defines it.
4. MUST include only the active identity's cogs and `base` cogs in a match.
5. SHOULD report to the owner, after each disclosure, what it shared and what it withheld.
6. MUST read the policy of a cog from its own copy of the record, never from a value supplied by the recipient.

---

## 6. Security and privacy considerations

- The fields describe limits; they do not enforce them. A personal cog copied to a non-conforming application keeps its fields and loses their protection. Owners SHOULD keep sensitive cogs `private`.
- A `derived` preference can still narrow who a person is when combined with other signals. Readers SHOULD send the smallest set of `disclosesAs` values a transaction needs.
- A hosted model that receives a prompt is a recipient like any other. A reader that routes between models MUST apply §5 per destination, including when it falls back from a local model to a hosted one.
- `revisionTraces: scraped` is a deletion obligation on the reader and its synchronisation. Copies already disclosed to a third party are outside its reach.

---

## 7. IANA considerations

This note proposes no new IANA registrations.

---

## 8. References

### 8.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) - Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) - Ambiguity of uppercase vs lowercase in RFC 2119 key words.

### 8.2 Informative

- **MX Core Metadata note** - the Zone 1 / Zone 2 model and the profile mechanism this note uses.
- **MX Provenance note** - the `parties` field a `delegate` declaration names.
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/) and [Media Queries Level 5](https://www.w3.org/TR/mediaqueries-5/) - user-preference media features.
- [Regulation (EU) 2016/679 (GDPR)](https://eur-lex.europa.eu/eli/reg/2016/679/oj) - Article 9 special categories of personal data.

---

## 9. Author quickstart (Informative)

```yaml
---
title: "Dietary needs"
description: "Coeliac and no nuts. Matched against menus and product labels."
author: Jane Doe
created: 2026-10-04
modified: 2026-10-04
type: personal-cog
mx:
  personalKind: dietary
  personalIdentity: base
  disclosureTier: guarded
  discloseFor: [dining, groceries]
  revisionTraces: kept
---
```

---

## 10. Common authoring mistakes (Informative)

```yaml
# WRONG - the reason travels: a derived cog names its condition
mx:
  disclosureTier: derived
  disclosesAs: [blind]
```

```yaml
# WRONG - guarded with no purposes: every disclosure stops to ask the owner
mx:
  disclosureTier: guarded
```

```yaml
# WRONG - fields at the top level are not read; this cog is private
disclosureTier: guarded
```
