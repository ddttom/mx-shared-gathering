---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Decision Records note"
docname: draft-cranstoun-mx-decision-records
date: 2026-05-17
consensus: false
keyword:
  - mx
  - decision
  - ai-accountability
  - records
  - audit
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-decision-records.md
---

# MX Decision Records note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 17 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies the **decision record** — a cog produced by an operator of an automated decision system that records one decision (or one batched group of decisions) against the policy in force at the time. The record carries enough metadata for an auditor to reason about whether the published policy was followed at the moment the decision was made; it does not carry the decision's input data, nor does it assert the correctness of the decision.

The note defines the record's fields, the relationship between a decision record and the policy it cites, and the boundary between what the record claims (an event occurred, against a named policy version, with a named model) and what it does not claim (that the decision was correct, that the model was suitable, or that the policy itself was sound).

Decision records combine with **MX Policy Records note** (the cited policy) and **MX Attestation Records and Verification note** (the cryptographic wrapper around the record) to give regulators, auditors, and subjects of decisions a queryable evidence chain. Each of those three records is a separate cog joined by reference; none of them substitutes for any of the others.

---

## 2. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this document are to be interpreted as described in [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) when, and only when, they appear in all capitals.

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of the draft set). That pattern governs the structural template for every field; this note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

An operator or consumer claiming conformance to this note SHALL satisfy:

- **Level 1 (record format)** — Produced decision records carry the §4.2 required fields, well-formed; absent fields are not silently defaulted. Field names follow `camelCase`.
- **Level 2 (referential integrity)** — Level 1 plus: every `policyRef` resolves to an existing policy record at the `policyRef.version` named, and the policy record's `effectiveFrom` is at or before the decision's `issued` timestamp.
- **Level 3 (completeness signalling)** — Level 2 plus: records declare a `sequence` field that monotonically increases per `(model, inputClass)` pair, supporting gap-detection by consumers.

A claim of conformance SHALL state the level reached.

### 2.2 Draft status

This is a draft note. Conformance levels and field semantics MAY change in response to community review. Implementations SHOULD note the draft version they implemented against.

---

## 3. Scope

### 3.1 In scope

- The fields of a decision record (§4.2).
- The reference direction between a decision record and the policy it cites (§4.4).
- The boundary between what the record claims and what it does not (§4.5, §4.6).
- The relationship to the cited policy record (§4.4) and to a wrapping attestation record (§5).

### 3.2 Out of scope

- Cryptographic apparatus over the record — signatures, batch roots, inclusion proofs — defer to the **MX Attestation Records and Verification note**.
- The schema of the cited policy itself — defers to the **MX Policy Records note**.
- Privacy treatment of the record's body content (`outputSummary`, human-readable description) where it contains personal data. Compliance with [GDPR Article 5(1)(c)](https://gdpr-info.eu/art-5-gdpr/) data minimisation is the operator's responsibility, not this note's.
- Subject-counter-signing mechanisms by which a subject corroborates a decision; that lives in a future companion note.
- Sector-specific profiles (financial services, medical devices, employment) — these are downstream specialisations, not core record semantics.

### 3.3 Relationship to existing standards

| Standard | Relationship |
|----------|--------------|
| **MX Cogs note** | Decision records are cogs. The universal cog fields (`cogUrn`, `schemaVersion`, `partOf`) apply per the MX Cogs note. |
| **MX Core Metadata note** | Doc-identity fields (`title`, `description`, `author`, `created`, `modified`) apply per the MX Core Metadata note. |
| **MX Policy Records note** | A decision record's `policyRef` resolves against a policy record. |
| **MX Attestation Records and Verification note** | A trust-layer attestation wraps a decision record by reference; the decision record itself carries no cryptographic apparatus. |
| [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) | Date and time format used by `issued`. |
| [W3C DID Core](https://www.w3.org/TR/did-core/) | Identifier format used by `operator`. |
| [SHA-256](https://datatracker.ietf.org/doc/html/rfc6234) | Hash function used by `inputHash`. |
| [ULID](https://github.com/ulid/spec) | RECOMMENDED form for the local-id portion of `decisionId`. |

---

## 4. Decision records

### 4.1 Position

A decision record is a structured statement of one decision made by an automated system, sufficient to let an auditor reason about whether the published policy was followed at that moment. The record carries identification, the citation of the policy in force, the model that made the decision, an input commitment (a hash, not the input itself), and a short factual description of the output.

A decision record records **what** the system decided. It does not claim that the decision was correct, that the model was suitable, or that the policy itself was sound. It is a faithful record of an event, nothing more.

The record is carrier-neutral. The fields and their semantics are what constitute the record, not the file format. Any carrier that can hold the fields validly is acceptable: a Markdown file with YAML frontmatter, a JSON document, an HTML page with `<meta>` and JSON-LD, a database row's metadata column, or a log line. The Markdown carrier wraps every field below `mx:` per the MX Cogs note; other carriers express the same fields without that envelope.

### 4.2 Required fields

A decision record declares `type: decision` at the top level (Zone 1b). The following fields are REQUIRED for a valid decision record regardless of carrier. Field names follow `camelCase`.

| Field | Type | Semantics |
|-------|------|-----------|
| `type` | string | Fixed value: `decision`. Top-level (Zone 1b). |
| `schemaVersion` | string | Semver of the decision-record schema this record conforms to (per **MX Cogs note** §6.5.4). |
| `cogUrn` | string | Universal identifier in the form `cog:<operator-namespace>:<local-id>` (per **MX Cogs note** §6.5.3). |
| `decisionId` | string | Operator's internal reference to the decision event; time-sortable. ULIDs are RECOMMENDED. Usually equals the `<local-id>` portion of `cogUrn`. |
| `operator` | string | DID or resolvable identifier of the operator (the AI system's deployer or controller). |
| `issued` | string | [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) timestamp of the decision. |
| `policyRef` | object | Reference to the policy record in force: registry URI, policy `cogUrn`, version, optional clause fragment. See §4.4. |
| `model` | object | Model identifier, version, and reference to the model card. See §4.3. |
| `inputHash` | string | SHA-256 of the canonicalised input. The input itself stays with the operator. |
| `inputClass` | string | Operator-defined category of input. Supports sequence-based completeness reasoning per §6. |
| `outputSummary` | string | Short factual description of the output. Human-readable. |
| `humanInLoop` | object | Whether human review was required, whether it fired, and an opaque reviewer reference. See §4.5. |

The following field is OPTIONAL but RECOMMENDED:

| Field | Type | Semantics |
|-------|------|-----------|
| `sequence` | integer | Per-`(model, inputClass)` monotonically increasing counter. Enables consumers to detect missing records by inspecting for gaps. See §6. |

A decision record MUST NOT carry trust-wrapper apparatus inline. Signatures, batch roots, and inclusion proofs live in a separate attestation record per the **MX Attestation Records and Verification note**, referencing the decision record by `cogUrn` and content digest. There is no `attestation` field on a decision record; there is no `batchRef` field.

### 4.3 The `model` block

The `model` field is an object with the following sub-fields:

| Sub-field | Type | Semantics |
|-----------|------|-----------|
| `id` | string | Stable identifier for the model (vendor namespace plus model name). |
| `version` | string | Version identifier of the specific model instance that produced the decision. |
| `cardRef` | string | `cogUrn` of the model card record that documents this model. |

The `model.cardRef` MUST resolve to a record describing the model (training data, evaluation, intended use). The decision record cites the model card by reference; it does not embed the model card.

### 4.4 The `policyRef` block

The `policyRef` field is an object with the following sub-fields:

| Sub-field | Type | Semantics |
|-----------|------|-----------|
| `registry` | string | URI of the registry that serves the cited policy record. |
| `cogUrn` | string | `cogUrn` of the policy record being cited. |
| `version` | string | Semver of the policy version in force at decision time. |
| `clause` | string | OPTIONAL. Fragment identifier of the specific clause within the policy. |

A `policyRef` resolves against the policy record as it stood at the decision's `issued` timestamp. Consumers performing time-pinned resolution use the registry's `asOf` query semantics to retrieve the version that was current at that moment, not whichever version is current today. The policy record's schema is specified by the **MX Policy Records note**.

### 4.5 The `humanInLoop` block

The `humanInLoop` field is an object with the following sub-fields:

| Sub-field | Type | Semantics |
|-----------|------|-----------|
| `required` | boolean | Whether the operator's policy required human review for this decision. |
| `fired` | boolean | Whether human review actually took place. |
| `reviewerRef` | string | OPTIONAL. Opaque reference to the reviewer (an internal identifier, not a name). |

When `required` is `true` and `fired` is `false`, the record is honestly signalling that policy-mandated review did not occur. This is a record of fact, not a defence; consumers reasoning about the record draw their own conclusions about whether the policy was followed.

### 4.6 Human-readable description

A decision record MUST include a human-readable description, in whatever way the carrier supports. In Markdown carriers, this is the body of the cog below the frontmatter. In JSON, this is a top-level `description` field. In HTML, this is the visible page content.

The description is part of the canonicalised payload. Where a trust layer attests the record per the **MX Attestation Records and Verification note**, the description is included in the signed bytes.

The description makes the record reviewable by humans — including the subject of the decision, where law requires that — without parsing the structured metadata.

### 4.7 Illustrative expression

The same record in the Markdown carrier:

```yaml
---
title: "Decision: claim-2026-05-15-184729"
description: "Routine insurance claim routed to standard review queue."
author: did:web:example-insurance.com
created: 2026-05-15T10:32:17Z
modified: 2026-05-15T10:32:17Z

type: decision
mx:
  schemaVersion: 1.0
  cogUrn: cog:web:example-insurance.com:01JC8X7K2N4P5Q6R7S8T9V0W1Y
  decisionId: claim-2026-05-15-184729
  operator: did:web:example-insurance.com
  issued: 2026-05-15T10:32:17Z
  policyRef:
    registry: https://registry.example
    cogUrn: cog:web:tg.community:eu-ai-act-art-12
    version: 1.0.0
    clause: para-1
  model:
    id: example-insurance/triage-classifier
    version: 2.3.1
    cardRef: cog:web:example-insurance.com:models-triage-v2-card
  inputHash: sha256:8f3a2b1c
  inputClass: insurance-claim-routine
  outputSummary: Routed to standard review queue
  humanInLoop:
    required: false
    fired: false
  sequence: 184729
---

This decision routed a routine insurance claim to the standard review queue based on the triage classifier's confidence score (above 0.85). No human review was required under the operator's policy for this input class.
```

The same record expressed as JSON carries the same field set flat (without the `mx:` envelope and without the Markdown body); the schema is identical, only the envelope differs.

---

## 5. Relationship to attestation

A decision record carries no signature. Where an operator chooses to attest the record cryptographically, the attestation is a separate cog produced by a trust layer and conforming to the **MX Attestation Records and Verification note**. The attestation references the decision record by `cogUrn` and content digest.

This separation matters: a decision record can be unattested, attested by one registry, or attested by several independent registries, without any change to the record's schema or bytes. Trust mechanisms evolve in the attestation layer; the decision record stays stable.

A decision record that has been superseded (a corrected record replacing an earlier one) is recorded via the supersession mechanism described in the **MX Cogs note**. The superseded record is not deleted; both remain queryable. Consumers reasoning about historical state resolve to either the original or the superseding version per the registry's `asOf` semantics.

---

## 6. Completeness

A single decision record speaks only for itself. Whether all decisions an operator made over a period are present in the registry is a separate question that no individual record can answer.

The `sequence` field supports completeness reasoning: operators assigning a monotonically increasing counter per `(model, inputClass)` enable consumers to detect missing records by inspecting for gaps. A registry serves the records; gap-detection happens at the consumer side.

Completeness reasoning is informative, not normative — the absence of a sequence number is not itself a conformance failure for an individual record. Operators committing to completeness signalling SHOULD declare `sequence`; consumers SHOULD respect operators who do.

Independent corroboration of a decision (a subject acknowledging that a decision was made about them, an auditor sampling against ground truth) is published as separate Verifiable Credentials per the [W3C VC Data Model](https://www.w3.org/TR/vc-data-model/), each referencing the decision record by `cogUrn`. This note does not specify the corroboration mechanism; companion future work will.

---

## 7. Why a separate record class

A decision record is structurally distinct from a content record (a document, a page, an asset). The fields a decision record carries are not optional decoration on a content record; they are the substance of what makes the record a faithful description of a decision.

Three properties motivate the separation:

- **Time-series shape.** Decision records are produced in high volume, time-ordered, by automated systems. Content records are produced in low volume by human or machine authors. The two classes have different operational characteristics that registries handle differently (batched ingest, sequence-based completeness reasoning, time-pinned retrieval).
- **Policy citation discipline.** A decision record's defining property is that it cites the policy it was made under. Content records have no equivalent obligation. Conflating the two would erode the discipline that makes decision records auditable.
- **Privacy posture.** A decision record carries `inputHash` rather than the input itself, by design. Content records typically carry their content. The schemas diverge on what is in the record.

Separating decision records into their own `contentType` keeps each class coherent and avoids leaking decision-specific obligations onto content records that have no use for them.

---

## 8. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 8.1 Producing decision records at high volume

Operators producing thousands of decisions per hour SHOULD batch the records for submission to a registry rather than submitting each one individually. The batching mechanism is specified by the **MX Attestation Records and Verification note**; a decision record is unchanged by being part of a batch.

### 8.2 Storage at the operator

The raw input that produced a decision stays with the operator. The `inputHash` field is the only thing about the input that reaches the registry. On later demand, an operator can produce the input and demonstrate the hash matches; the registry never holds the input.

### 8.3 Selective disclosure

Where a regulator's question is "how many decisions did this operator make in this category with this output distribution", the operator can publish a signed aggregate statement (a Verifiable Credential) that commits to the underlying decisions without disclosing them. The regulator verifies the credential using standard VC libraries.

### 8.4 Verification

A consumer fetches a decision record, computes its canonical bytes, and resolves the cited `policyRef` against the registry's `asOf` semantics. If the policy version in force at the decision's `issued` timestamp does not match the `policyRef.version`, the record has cited a policy version that was not in force; that is a discrepancy the consumer surfaces.

---

## 9. Security and privacy considerations

- The `outputSummary` and the human-readable description MAY contain personal data describing the subject of the decision. Operators are responsible for data minimisation per [GDPR Article 5(1)(c)](https://gdpr-info.eu/art-5-gdpr/) and equivalent provisions in other jurisdictions. A registry serving the record does not redact or filter content; the record is published as the operator submitted it.
- The `inputHash` commits the input cryptographically without disclosing it. An operator who later produces the input and demonstrates the hash matches has shown the record was not retrospectively edited to fit a different input. The hash itself reveals nothing about the input's contents.
- The `humanInLoop.reviewerRef` field is opaque by intent. A reviewer name in this field would create a personal-data disclosure with no audit benefit; an opaque reference (a UUID or an internal employee identifier with no external mapping) supports traceability without disclosure.
- Decision records are not authenticated by their own schema. Authentication lives in the attestation layer per the **MX Attestation Records and Verification note**. A consumer who needs to verify provenance of a decision record fetches the wrapping attestation, not the record alone.
- Sequence numbers leak operator throughput. An operator who values throughput privacy SHOULD batch its sequence increments at a coarser granularity, or omit `sequence` and accept the reduced completeness-detection capability that follows.

---

## 10. IANA considerations

This note proposes no new IANA registrations.

---

## 11. References

### 11.1 Normative

- [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) ([RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119), [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)) — key words for use in RFCs to indicate requirement levels.
- [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) — date and time on the Internet, timestamp format.
- [RFC 6234](https://datatracker.ietf.org/doc/html/rfc6234) — US Secure Hash Algorithms (including SHA-256).
- [W3C DID Core 1.0](https://www.w3.org/TR/did-core/) — identifier format for `operator`.
- **MX Cogs note** — universal cog fields (`cogUrn`, `schemaVersion`, `partOf`, supersession).
- **MX Policy Records note** — schema of the record cited by `policyRef`.
- **MX Attestation Records and Verification note** — schema of the trust-layer wrapper.
- **MX Field Definition Pattern note** — structural template for field definitions.

### 11.2 Informative

- [W3C Verifiable Credentials Data Model](https://www.w3.org/TR/vc-data-model/) — used for subject corroboration and selective-disclosure aggregates.
- [GDPR Article 5(1)(c)](https://gdpr-info.eu/art-5-gdpr/) — data minimisation principle.
- [ULID specification](https://github.com/ulid/spec) — recommended form for the local-id portion of `decisionId`.

---

## 12. Acknowledgements

This note draws on the documentation-evidence framing that emerged from internal review of AI governance regimes by Tom Cranstoun and Scott McGregor at Digital Domain Technologies Ltd, and on community feedback at The Gathering's technical sessions.
