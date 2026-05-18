---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Policy Records note"
docname: draft-cranstoun-mx-policy-records
date: 2026-05-17
consensus: false
keyword:
  - mx
  - policy
  - regulation
  - records
  - versioning
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-policy-records.md
---

# MX Policy Records note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 17 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies the **policy record** — a cog that publishes a regulatory code, code of practice, or governance policy in a form that other records can cite by reference. A policy record carries a stable identifier, semver versioning, jurisdictional scope, an effective-from date, and addressable clauses; consumers reasoning about whether a decision followed the policy in force at a given moment resolve the citation deterministically.

Decision records cite policy records by `policyRef` (per the **MX Decision Records note**); the same citation discipline applies to any other record class that asserts a policy was followed (compliance claims, audit reports, attestation extensions). Policies that are *not* published as policy records resolve to moving targets: the regulator's PDF is replaced silently, the clause is renumbered, and the citation chain breaks under audit. Publishing the policy as a record fixes that.

The note is carrier-neutral: a regulator's HTML publication with JSON-LD structured data is a valid policy record; so is a Markdown file with YAML frontmatter; so is a JSON document. The Markdown carrier wraps every field below `mx:` per the **MX Cogs note**; other carriers express the same fields without that envelope.

---

## 2. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) when, and only when, they appear in all capitals.

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of the draft set). That pattern governs the structural template for every field; this note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

A publisher or consumer claiming conformance to this note SHALL satisfy:

- **Level 1 (record format)** — Produced policy records carry the §4.2 required fields, well-formed; absent fields are not silently defaulted. Field names follow `camelCase`. The `clauses` array enumerates every addressable fragment.
- **Level 2 (versioning discipline)** — Level 1 plus: `policyVersion` follows semver; `effectiveFrom` is RFC 3339; supersession of an earlier version uses the `supersedes` mechanism rather than in-place edit.
- **Level 3 (addressable clauses)** — Level 2 plus: every clause carries a stable fragment identifier the body anchor matches, so consumers citing a clause by `policyRef.clause` resolve to the same text the publisher intended.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Implementations SHOULD note the draft version they implemented against.

---

## 3. Scope

### 3.1 In scope

- The fields of a policy record (§4.2).
- The relationship between `cogUrn` (universal identifier) and `policyId` (publisher-internal short reference) (§4.3).
- Clause addressability (§4.4): how a decision record's `policyRef.clause` resolves to a specific fragment.
- Versioning and supersession (§5).
- Third-party publication by The Gathering or other intermediaries when the issuing body has not yet published its own (§6).

### 3.2 Out of scope

- The cryptographic apparatus over the record — signatures, witnesses, batched proofs — defer to the **MX Attestation Records and Verification note**.
- The schema of records that cite a policy — defers to the **MX Decision Records note** and to future companion notes for compliance claims and audit reports.
- Editorial interpretation of any specific regulatory code; this note specifies the record format, not what any policy says.
- Translation handling. A policy record published in one language and translated into another is a separate record citing the original via `derivedFrom`; the translation mechanism is out of scope for this note.

### 3.3 Relationship to existing standards

| Standard | Relationship |
|----------|--------------|
| **MX Cogs note** | Policy records are cogs. The universal cog fields (`cogUrn`, `schemaVersion`, `partOf`, supersession) apply per the MX Cogs note. |
| **MX Core Metadata note** | Doc-identity fields (`title`, `description`, `author`, `created`, `modified`) apply per the MX Core Metadata note. |
| **MX Decision Records note** | Decision records cite policy records via `policyRef`. |
| **MX Attestation Records and Verification note** | A trust-layer attestation MAY wrap a policy record by reference. |
| [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) | Date and time format used by `effectiveFrom`. |
| [W3C DID Core](https://www.w3.org/TR/did-core/) | Identifier format used by `issuingBody`. |
| [ISO 3166-1 alpha-2](https://www.iso.org/iso-3166-country-codes.html) | Country code format used by `jurisdiction`. |

---

## 4. Policy records

### 4.1 Position

A policy record is the durable referent that every citing record's `policyRef` resolves against. The record fixes three things that an HTML publication or a PDF on a regulator's website does not fix: a stable identifier that survives URL changes, a versioning model that lets consumers reason about which clause text was in force at a given moment, and addressable fragments so a citation like "clause 3.2" resolves deterministically.

A policy record records the **text** and **structure** of a policy. It does not assert that the policy is sound, that the policy is currently in force in any jurisdiction, or that any particular decision did or did not follow it. Those assertions, where needed, live in citing records (decision records, compliance claims, audit reports) and in their attestation wrappers.

### 4.2 Required fields

A policy record declares `contentType: policy` in its `mx:` frontmatter. The following fields are REQUIRED for a valid policy record regardless of carrier. Field names follow `camelCase`.

| Field | Type | Semantics |
|-------|------|-----------|
| `contentType` | string | Fixed value: `policy`. |
| `schemaVersion` | string | Semver of the policy-record schema this record conforms to (per **MX Cogs note** §6.5.4). |
| `cogUrn` | string | Universal identifier in the form `cog:<publisher-namespace>:<local-id>` (per **MX Cogs note** §6.5.3). For third-party renderings by The Gathering, the publisher namespace is typically `web:tg.community`. |
| `policyId` | string | Publisher's internal reference to the policy; typically the `<local-id>` portion of `cogUrn`. Provides a short stable handle for human reference. |
| `policyVersion` | string | Semver of this policy version. Distinct from `schemaVersion` (which versions the record schema) and from `version` (which versions the document file). |
| `issuingBody` | string | DID or resolvable identifier of the body that issued the policy (the regulator, the standards body, the operator publishing an internal policy). |
| `jurisdiction` | string | [ISO 3166-1 alpha-2](https://www.iso.org/iso-3166-country-codes.html) country code; or `EU`; or `international` for policies with no single-jurisdiction scope. |
| `effectiveFrom` | string | [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) date the policy version takes effect. |
| `clauses` | array | Addressable fragments. Each entry is an object with `id`, `title`, and `bodyAnchor`. See §4.4. |

The following field is OPTIONAL but RECOMMENDED when applicable:

| Field | Type | Semantics |
|-------|------|-----------|
| `supersedes` | object | Reference to a previous policy version this one replaces. Carries `cogUrn` and `version` of the superseded record. See §5. |

A policy record MUST NOT carry cryptographic apparatus inline. Attestation of a policy record — typically by the issuing body or by The Gathering acting as third-party publisher per §6 — lives in a separate attestation record per the **MX Attestation Records and Verification note**.

### 4.3 `cogUrn` and `policyId`

A policy record carries both a `cogUrn` (the universal identifier per **MX Cogs note** §6.5.3) and a `policyId` (the publisher's short stable handle). They serve different purposes:

- `cogUrn` is the universal long-form identifier used by registries, clients, verifiers, and anything outside the publisher's own systems. It is namespace-qualified and globally unique.
- `policyId` is the publisher's local handle, typically equal to the `<local-id>` portion of `cogUrn`. It exists for human reference and for compactness in citation-heavy prose where the full `cogUrn` would clutter.

In most implementations the `<local-id>` portion of `cogUrn` and `policyId` SHOULD be equal; publishers who want a separation (because their internal IDs leak information they prefer to keep internal) MAY choose distinct values.

### 4.4 The `clauses` array

The `clauses` array makes individual policy fragments addressable. Each entry is an object with the following sub-fields:

| Sub-field | Type | Semantics |
|-----------|------|-----------|
| `id` | string | Fragment identifier. Stable across versions: where a clause is preserved verbatim across a policyVersion bump, its `id` MUST NOT change. |
| `title` | string | Human-readable title of the clause. |
| `bodyAnchor` | string | Anchor name within the record's body that points at the clause text. In a Markdown carrier this is typically a heading slug; in an HTML carrier it is an element `id`. |

A citing record references a specific clause by setting `policyRef.clause` to the clause's `id`. Consumers MUST resolve `policyRef.clause` to the corresponding `clauses[i].bodyAnchor` and retrieve the clause text from the body of the policy record. A `clauses` entry whose `bodyAnchor` does not match a real anchor in the body is a conformance failure.

Where a clause is amended within a major policy version (without renumbering), the publisher SHOULD bump `policyVersion` (minor or patch as appropriate) and preserve the clause `id` so existing citations continue to resolve. Where a clause is removed or renumbered, the publisher MUST bump the major component of `policyVersion`; the previous version remains queryable via supersession (§5) so historical citations still resolve.

### 4.5 Illustrative expression

```yaml
---
title: "EU AI Act, Article 12: event recording"
description: "Policy record rendering of EU AI Act Article 12 on automatic event recording."
author: did:web:digital-strategy.ec.europa.eu
created: 2026-08-02
modified: 2026-08-02

mx:
  contentType: policy
  schemaVersion: 1.0
  cogUrn: cog:web:tg.community:eu-ai-act-art-12
  policyId: eu-ai-act-art-12
  policyVersion: 1.0.0
  issuingBody: did:web:digital-strategy.ec.europa.eu
  jurisdiction: EU
  effectiveFrom: 2026-08-02T00:00:00Z
  clauses:
    - id: para-1
      title: Record-keeping requirement
      bodyAnchor: record-keeping-requirement
    - id: para-2
      title: Automatic event recording
      bodyAnchor: automatic-event-recording
---

## Record-keeping requirement {#record-keeping-requirement}

(Body text for paragraph 1 goes here.)

## Automatic event recording {#automatic-event-recording}

(Body text for paragraph 2 goes here.)
```

The same record expressed as JSON carries the same field set flat; the schema is identical, only the envelope differs.

---

## 5. Versioning and supersession

A policy record is amended through the supersession mechanism described in the **MX Cogs note**. A new policy record carrying `supersedes: { cogUrn: <previous cogUrn>, version: <previous policyVersion> }` replaces the earlier version; both remain queryable, and consumers reasoning about historical state retrieve whichever version was in force at the moment they care about.

The interaction between `policyVersion` and supersession:

- **Patch and minor** versions remain inside the same logical record (`cogUrn` unchanged; new `policyVersion`; `supersedes` references the previous `policyVersion` of the same `cogUrn`). The new version inherits clause ids from the previous version.
- **Major** versions MAY also remain inside the same logical record, but publishers MAY choose to mint a new `cogUrn` when the policy changes scope significantly (e.g. one regulation splitting into two). When a new `cogUrn` is minted, `supersedes` references the predecessor's `cogUrn` and version explicitly.

A consumer performing time-pinned resolution (asking "what was the policy in force on date X") retrieves whichever version's `effectiveFrom` is the latest non-future date relative to X. The registry serves; consumers reason.

A withdrawn policy (a record formally retracted by the issuing body) is recorded by a superseding record carrying `withdrawn: true`. The original is not deleted; consumers reasoning about decisions made before the withdrawal still resolve to the original text.

---

## 6. Third-party publication

Many regulators do not currently publish their codes as policy records. Until they do, an intermediary (The Gathering, a sector body, an academic institution) MAY publish a **third-party rendering** of a policy as a policy record, signed by the intermediary rather than by the issuing body.

A third-party rendering MUST:

- Declare the original issuing body in `issuingBody` (the regulator's DID, not the renderer's).
- Be clearly labelled in its `description` field as a third-party rendering, not source-of-truth.
- Cite the source publication (a URL, a regulatory document number, or both) in the body text.
- Be signed by the rendering intermediary's key per the **MX Attestation Records and Verification note**; the attestation's `issuedBy` is the renderer, not the issuing body.

When the issuing body eventually publishes its own signed version, the third-party rendering is superseded via §5. The new record carries the issuing body's signature; the third-party record remains queryable for historical citation but is no longer the recommended target for new citations.

This is the same pattern Certificate Transparency used: independent monitors started logging first, authorities later participated.

---

## 7. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 7.1 Choosing `policyId` and `cogUrn`

For a regulatory code, a reasonable convention is `policyId` = the regulator's short reference for the code (e.g. `eu-ai-act-art-12`, `iso-27001`, `nist-800-53-rev5-ac-3`), and `cogUrn` = `cog:<renderer-namespace>:<policyId>`. This keeps citations human-readable while preserving universal uniqueness.

### 7.2 Stable clause ids

A publisher anticipating that consumers will cite specific clauses SHOULD adopt clause ids that match the policy's own internal numbering (e.g. `para-1`, `sub-clause-3-2-a`, `annex-iii-point-5`). Inventing renderer-specific clause ids that diverge from the policy's own numbering creates a translation burden every citing record must handle.

### 7.3 Effective-from versus enacted-on

`effectiveFrom` is when the policy version becomes binding on regulated entities, not when the legal instrument was passed. A policy enacted on date X with a one-year transition period has `effectiveFrom: X + 1 year`. Consumers reasoning about whether a decision followed the policy in force at decision time MUST use `effectiveFrom`, not the enactment date.

### 7.4 Verification

A consumer fetches a policy record and checks: that `clauses[].bodyAnchor` resolves to a real anchor in the body for every clause; that `effectiveFrom` is RFC 3339; that `policyVersion` is semver; and (where the record carries an attestation) that the attestation's signature verifies against the renderer's published key.

---

## 8. Security and privacy considerations

- A third-party rendering signed by an intermediary attests provenance and integrity of the rendering, not the legal correctness of the policy text. Even when the attestation verifies cleanly, downstream consumers MUST NOT treat the text as legally authoritative — it is the renderer's faithful copy, not the issuing body's instrument. User-facing language SHOULD say *attested rendering*, not *official policy*.
- A policy record carrying clause text in its body is plain content with no privacy obligations; policy text is typically public. However, where a policy record is published with internal-policy scope (an operator's own decision-making rules, not a regulator's code), the body content MAY be sensitive. Internal-policy records SHOULD declare `confidential: true` per the **MX Core Metadata note** and be served from access-controlled endpoints.
- The `issuingBody` field is operator-attested, not authority-verified. A record claiming `issuingBody: did:web:fictitious-regulator.example` is conformant by this note's rules even if no such regulator exists. Consumers reasoning about regulatory weight MUST verify the `issuingBody` DID resolves to the genuine regulator through external means (the regulator's known web property, an authoritative directory of regulatory bodies, or a trusted intermediary's vouching).
- Withdrawal of a policy via `withdrawn: true` is published by the same publisher that published the original. A publisher who has lost the operational capability to publish (organisation dissolved, signing key compromised beyond recovery) cannot withdraw their own policies; consumers reasoning about long-superseded policies MAY find them still present in the registry but should treat them as historical, not current.

---

## 9. IANA considerations

This note proposes no new IANA registrations.

---

## 10. References

### 10.1 Normative

- [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119), [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) — key words for use in RFCs to indicate requirement levels.
- [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) — date and time on the Internet, timestamp format.
- [ISO 3166-1 alpha-2](https://www.iso.org/iso-3166-country-codes.html) — country codes for the `jurisdiction` field.
- [W3C DID Core 1.0](https://www.w3.org/TR/did-core/) — identifier format for `issuingBody`.
- **MX Cogs note** — universal cog fields (`cogUrn`, `schemaVersion`, `partOf`, supersession).
- **MX Decision Records note** — record class that cites policy records via `policyRef`.
- **MX Attestation Records and Verification note** — schema of the trust-layer wrapper.
- **MX Field Definition Pattern note** — structural template for field definitions.

### 10.2 Informative

- [Semantic Versioning 2.0.0](https://semver.org/) — version-number format used by `policyVersion` and `schemaVersion`.
- [Certificate Transparency (RFC 6962)](https://datatracker.ietf.org/doc/html/rfc6962) — analogue for the third-party-publication pattern in §6.

---

## 11. Acknowledgements

This note draws on the documentation-evidence framing that emerged from internal review of AI governance regimes by Tom Cranstoun and Scott McGregor at Digital Domain Technologies Ltd, and on community feedback at The Gathering's technical sessions.
