---
title: "MX Offer Terms note"
docname: draft-cranstoun-mx-offer-terms
date: 2026-10-19
consensus: false
keyword:
  - mx
  - offer-terms
  - cancellation
  - loyalty
  - agent-commerce
  - schema-org
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-offer-terms.md
---

# MX Offer Terms note

**Version:** 0.1
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 19 October 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

An agent booking a table, a room or a ticket for its owner skips the page design and the countdown timer and asks three things: what cancelling costs, what the full price is, and what the loyalty programme is worth. Schema.org answers most of the second through its price specifications. It answers part of the third and none of the first.

This note defines two optional blocks in Zone 2 (the `mx:` object) that fill those gaps: `cancellationTerms`, which states what cancelling a booking or order costs and until when cancelling is free, and `loyaltyPointTerms`, which states what a loyalty point is worth, how points are earned, and when they lapse or cannot be spent. Both defer to Schema.org for everything it already covers.

Both blocks are claims by the business that publishes the document. They summarise the binding terms and never replace them.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A document that carries either block SHALL satisfy:

- **Level 1 (Declared):** each block present carries its REQUIRED keys (§4), and the document carries `currency` wherever an amount or a point value appears.
- **Level 2 (Linked):** Level 1 plus `termsUrl` in any `cancellationTerms` block, and `earnRate` and `pointValue` in any `loyaltyPointTerms` block.
- **Level 3 (Current):** Level 2 plus, for every fee or point value that changes on its own cadence, a declared validity window and a place to fetch the current figure (§5.4).

A reader conforms when it applies §5 to every block it reads, whatever level the document reaches.

### 2.2 Draft status

This note is a draft offered for review. Its keys are in use in one reference implementation under the same names, marked experimental there, so that early implementations share one spelling. They may change shape or be withdrawn before ratification.

---

## 3. Scope

### 3.1 In scope

- The published cost of cancelling a booking or order, and the period during which cancelling is free.
- The published terms of a points programme: how points are earned and redeemed, when they expire, and when they cannot be spent.
- The behaviour a reader owes those terms when it reports them to an owner or acts on them.

### 3.2 Out of scope

- One member's balance, tier or history. That is personal data and belongs in the member's own record; see the **MX Personal Cogs note**.
- Returns of goods, which `schema:MerchantReturnPolicy` covers.
- Prices, taxes and fees, which Schema.org price specifications cover.
- Whether any stated term meets a legal requirement. The blocks state terms; they are not evidence of compliance.

### 3.3 Relationship to existing standards

Schema.org covers much of an offer's terms but leaves two gaps.

- **Cancellation cost.** `schema:MerchantReturnPolicy` describes returning goods: the window, the method, who pays for return shipping. It does not describe cancelling a booking or a service. A `schema:Reservation` can say it was cancelled, through `reservationStatus: ReservationCancelled`, but no property states what cancelling costs or until when it is free.
- **Loyalty-point value.** `schema:MemberProgram`, `schema:MemberProgramTier`, `hasTierBenefit` and `validForMemberTier` describe a programme, its tiers and the offers each tier gets. None of them states what a point is worth, how many points a unit of spend earns, when points expire, or when they cannot be redeemed.

`loyaltyPointTerms.program` references a `schema:MemberProgram` and duplicates none of its properties. Where Schema.org later defines a property for either gap, a writer SHOULD use it, and the matching key here SHOULD be deprecated in its favour.

Durations and intervals follow ISO 8601, fixed times RFC 3339, and currency codes ISO 4217.

---

## 4. Field definitions

All fields belong in Zone 2 (the `mx:` object). A reader MUST NOT read them from the top level of the frontmatter. Amounts are in the document's `currency` field, an ISO 4217 code.

### 4.1 `cancellationTerms`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | OPTIONAL |
| **Keys** | `freeUntil`, `fee` (REQUIRED), `termsUrl` |

**Definition:** What cancelling a booking or order the document offers costs, and until when cancelling is free, so an agent can read the terms before it commits on its owner's behalf.

```yaml
mx:
  currency: GBP
  cancellationTerms:
    freeUntil: PT24H
    fee: 15.00 GBP
    termsUrl: https://example.com/booking-terms
```

- The block summarises the binding terms at `termsUrl`. It MUST NOT be read as replacing them.
- An absent block says nothing about the terms. A reader MUST NOT read its absence as free cancellation.

#### 4.1.1 `cancellationTerms.freeUntil`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Conformance** | OPTIONAL |
| **Format** | ISO 8601 duration, or RFC 3339 date-time |

**Definition:** How long before the booked start a cancellation stays free, as an ISO 8601 duration, or the fixed RFC 3339 time after which a fee applies.

- `P0D` means a fee applies from the moment of booking.
- A writer SHOULD omit it when `fee` is `0`.

#### 4.1.2 `cancellationTerms.fee`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Conformance** | REQUIRED when `cancellationTerms` is present |
| **Format** | an amount with its ISO 4217 code (`25.00 GBP`), a percentage (`100%`), or `0` |

**Definition:** The charge for cancelling after `freeUntil`.

- A percentage is of the total price including fees and taxes.
- `0` means cancelling is free at any time.

#### 4.1.3 `cancellationTerms.termsUrl`

| Property | Value |
|----------|-------|
| **Type** | string (URL) |
| **Conformance** | RECOMMENDED (Level 2) |

**Definition:** The address of the full cancellation terms the block summarises.

### 4.2 `loyaltyPointTerms`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Conformance** | OPTIONAL |
| **Keys** | `program` (REQUIRED), `earnRate`, `pointValue`, `expiresAfter`, `expiryBasis`, `blackoutDates` |

**Definition:** What a loyalty point earned on the document's offers is worth, how points are earned, and when they lapse or cannot be spent, so an agent can calculate a programme's value instead of guessing it.

```yaml
mx:
  currency: GBP
  loyaltyPointTerms:
    program: https://example.com/#regulars
    earnRate: 1
    pointValue: 0.05
    expiresAfter: P12M
    expiryBasis: last-activity
    blackoutDates:
      - 2026-12-24/2026-12-26
```

- The block declares the published terms of the programme. It MUST NOT carry any one member's balance, tier, history or identifier.

#### 4.2.1 `loyaltyPointTerms.program`

| Property | Value |
|----------|-------|
| **Type** | string (URL or `@id`) |
| **Conformance** | REQUIRED when `loyaltyPointTerms` is present |
| **Aligns with** | `schema:MemberProgram` |

**Definition:** The Schema.org `MemberProgram` the points belong to, as its URL or `@id`.

#### 4.2.2 `loyaltyPointTerms.earnRate`

| Property | Value |
|----------|-------|
| **Type** | number |
| **Conformance** | RECOMMENDED (Level 2) |

**Definition:** The number of points earned for each one unit of the document's `currency` spent.

#### 4.2.3 `loyaltyPointTerms.pointValue`

| Property | Value |
|----------|-------|
| **Type** | number |
| **Conformance** | RECOMMENDED (Level 2) |

**Definition:** The redemption value of one point, in the document's `currency`.

- Where redemption rates vary, it is the lowest figure every member can rely on, with the detail in the programme terms.

#### 4.2.4 `loyaltyPointTerms.expiresAfter`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Conformance** | OPTIONAL |
| **Format** | ISO 8601 duration |

**Definition:** How long a point stays valid, counted from the date `expiryBasis` names.

- A writer MUST omit it when points do not expire.

#### 4.2.5 `loyaltyPointTerms.expiryBasis`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Conformance** | REQUIRED when `expiresAfter` is present |
| **Valid values** | earned, last-activity |

**Definition:** Whether `expiresAfter` counts from the date a point was `earned` or from the member's `last-activity` (their last earning or redemption).

#### 4.2.6 `loyaltyPointTerms.blackoutDates`

| Property | Value |
|----------|-------|
| **Type** | array of strings |
| **Conformance** | OPTIONAL |
| **Format** | ISO 8601 intervals |

**Definition:** The periods when points cannot be redeemed.

---

## 5. Reader behaviour

A reader that reports offer terms to an owner, or acts on them on the owner's behalf:

1. MUST NOT report cancellation as free when `cancellationTerms` is absent. It reports the terms as not declared.
2. MUST treat a block as malformed when it lacks a REQUIRED key, or when it states an amount or point value and the document has no `currency`, and MUST NOT act on it.
3. Where the block and the binding terms at `termsUrl` disagree, MUST treat the binding terms as governing, and SHOULD record the difference as a finding.
4. Where the validity window declared for a fee or point value has passed, MUST fetch the current figure before quoting it.
5. SHOULD set a quoted price, fee or point value beside the declared terms and treat a difference as a finding.
6. MUST NOT send a member's balance or tier to the business as part of reading these blocks, unless the member's own disclosure rules allow it.

---

## 6. Worked example (informative)

A restaurant's page states its terms in prose:

- Cancel free of charge up to 24 hours before the booking; after that, £15 per booking.
- Earn 1 point per £1 spent, each worth 5p.
- Points stay valid while the member visits at least once every 12 months.
- Points cannot be redeemed from 24 to 26 December.

The same terms as metadata:

```yaml
mx:
  currency: GBP
  cancellationTerms:
    freeUntil: PT24H
    fee: 15.00 GBP
    termsUrl: https://example.com/booking-terms
  loyaltyPointTerms:
    program: https://example.com/#regulars
    earnRate: 1
    pointValue: 0.05
    expiresAfter: P12M
    expiryBasis: last-activity
    blackoutDates:
      - 2026-12-24/2026-12-26
```

From this an agent can tell its owner, before booking a table for Friday at 19:00, that cancelling after 19:00 on Thursday costs £15, and that a £60 bill earns 60 points worth £3.00 off a later visit, though not on 24 to 26 December. It reads none of that from the page copy, and it can compare a second restaurant on the same terms.

---

## 7. Rationale (informative)

**Why a single threshold for cancellation.** Most published booking terms state one free period and one charge. A single threshold covers them and keeps the comparison simple. Stepped schedules exist; whether to model them is an open question (§11).

**Why the lowest guaranteed point value.** A programme whose points are worth more on some redemptions than others gives an agent no single figure. The floor every member can rely on is the figure an agent can act on without overstating the programme to its owner.

**Why an absent block says nothing.** Reading absence as free cancellation would reward a business for leaving the block out.

**Why no member data.** A programme's terms are public; a member's balance is personal data. Keeping them apart lets a business publish its terms without touching member records, and keeps the balance under the member's control.

**Alternatives considered.** Extending `MerchantReturnPolicy` to bookings was rejected because its properties describe goods coming back, not a service not taken up. Stating the worth of a point as a `hasTierBenefit` string was rejected because free text is not comparable across programmes. Leaving both to `termsUrl` alone was rejected because prose behind a URL is the gap this note addresses.

---

## 8. Security and privacy considerations

- **Stated terms can be wrong.** A business can declare terms that differ from what it charges. The blocks are claims; a reader that sees a difference records it (§5), and a decision that depends on the terms SHOULD look for corroboration.
- **Bait terms.** A business could declare generous terms in metadata and stricter ones at `termsUrl`. §5 makes the binding terms govern and asks the reader to record the difference, so the mismatch becomes visible rather than silently resolved in the business's favour.
- **Member data.** §4.2 keeps balances and tiers out of the block, and §5 keeps the reader from sending them to the business as a side effect of reading it.

---

## 9. IANA considerations

This note proposes no new IANA registrations.

---

## 10. References

### 10.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) - Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) - Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) - Date and time on the internet.
- ISO 8601 - Date and time format (durations and intervals).
- ISO 4217 - Currency codes.

### 10.2 Informative

- **MX Core Metadata note** - the Zone 1 / Zone 2 model and the `currency` field.
- **MX Personal Cogs note** - the member's own record, where a balance and tier belong.
- [Schema.org `MerchantReturnPolicy`](https://schema.org/MerchantReturnPolicy)
- [Schema.org `Reservation`](https://schema.org/Reservation)
- [Schema.org `MemberProgram`](https://schema.org/MemberProgram) and [`MemberProgramTier`](https://schema.org/MemberProgramTier)

---

## 11. Open questions

Raised for community review:

1. Should `cancellationTerms` allow a stepped schedule (for example free until two days before, 50% until one day before, 100% after) as a list, or stay a single threshold with the detail at `termsUrl`, as proposed?
2. Should a no-show charge that differs from the late-cancellation fee have its own key?
3. Should `loyaltyPointTerms` carry per-tier earn rates keyed to `schema:MemberProgramTier`, or stay at programme level, as proposed?
4. Should `pointValue` keep the lowest guaranteed value, as proposed, or allow a range?
