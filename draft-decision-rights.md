---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Decision Rights note"
docname: draft-cranstoun-mx-decision-rights
date: 2026-10-19
consensus: false
keyword:
  - mx
  - governance
  - accountability
  - decision-rights
  - ai-management-system
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-decision-rights.md
---

# MX Decision Rights note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 19 October 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

An organisation that governs its AI systems can name an owner for every system and every risk and still have nobody accountable for the decisions that cross them: which AI investments are funded, whether a tool one team made should serve every team, which AI uses are approved for release. Such decisions are often shared by two executives, and a decision two people share is in practice one nobody owns. When something goes wrong, the first question is who was supposed to own it, and the answer is rebuilt from memory.

This note defines a record that answers the question in advance. A **decision right** covers one class of AI decision and names exactly one role accountable for it, the roles consulted before it is taken, and the condition under which it goes up a level. A record that carries a sign-off can cite the decision right it falls under, so a reader can compare who signed with who owns the decision.

The note defines the record kind `aims:decisionRight`, the profile `aims-decision-right`, two accountability roles (`decisionOwner`, `consulted`), and three fields in Zone 2 (the `mx:` object): `decisionScope`, `escalation` and `decisionRightRef`. It also states how the record uses `parties` and the record identity fields.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

A decision-right record SHALL satisfy:

- **Level 1 (Allocated):** the record carries `type: aims:decisionRight`, the Zone 1 identity fields, `recordId`, `status`, `decisionScope`, `approvedAt`, and a `parties` list with exactly one entry whose role is `decisionOwner`.
- **Level 2 (Bounded):** Level 1 plus `escalation` and `reviewBy`.

A record that carries a sign-off conforms to this note at **Level 3 (Linked)** when it carries `decisionRightRef`. A reader that compares sign-offs conforms when it applies §6 to every record carrying `decisionRightRef`.

### 2.2 Draft status

This note is a draft. Implementations SHOULD record the version they implemented against, and SHOULD expect field shapes to change before ratification.

---

## 3. Scope

### 3.1 In scope

- The record of who owns one class of AI decision, who is consulted, and when the decision goes up a level.
- The link from a record that carries a sign-off to the decision right it falls under.
- The comparison a reader runs between the two.

### 3.2 Out of scope

- How an organisation chooses its owners. The record holds an allocation, never whether the allocation is sound.
- The records that carry sign-offs (risk register entries, statements of applicability, impact assessments). This note adds one field to them and defines nothing else about them.
- Delegation. A delegated sign-off is reported as a mismatch until the delegation is itself recorded as a decision right.

### 3.3 Relationship to existing standards

- The record kind is a value of the OKF `type` field, and the record follows the zone model of the **MX Core Metadata note**.
- [ISO/IEC 42001:2023](https://www.iso.org/standard/81230.html) Annex A control A.3.2 asks an organisation to define and allocate roles and responsibilities for AI. A decision-right record is one place to keep that allocation. Carrying the record is not conformance to the standard.
- The [NIST AI Risk Management Framework 1.0](https://doi.org/10.6028/NIST.AI.100-1) asks for documented roles and lines of communication (GOVERN 2.1), for executive leadership to take responsibility for decisions about AI risk (GOVERN 2.3), and for roles in human-AI configurations to be defined (GOVERN 3.2). The record supports those outcomes; it does not establish them.

---

## 4. Terminology

- **Decision right:** a record allocating one class of AI decision to one accountable role.
- **Sign-off:** the act, recorded on another record, of a person accepting or approving something within a class of decision.
- **Signer:** the person a record names as having made its sign-off.
- **Owner:** the person a decision right names with the role `decisionOwner`.

---

## 5. Fields

### 5.1 `decisionScope`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | aims-decision-right |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none - explicit declaration required)* |

The class of AI decision the record allocates, stated so that a person or an agent can tell whether a given decision falls inside it. One class per record.

```yaml
mx:
  decisionScope: Whether an AI tool requested by one team is built once as a shared service or left to that team.
```

- A decision that fits no recorded scope has no owner yet. A reader SHOULD report it rather than infer one.

### 5.2 `parties`

| Property | Value |
|----------|-------|
| **Type** | array of object |
| **Zone** | 2 (mx:) |
| **Conformance** | MUST (Level 1) |
| **Default** | *(none - explicit declaration required)* |

The people and bodies accountable for the record. An entry holds `role` and `name`, and MAY add `identifier` (a URL for the person), `organisation` and `email`. This note defines two values of `role`: `decisionOwner`, the one role accountable for the class of decision, and `consulted`, a role whose view is taken before the decision without sharing ownership of it. An `approver` entry names who agreed the allocation.

```yaml
mx:
  parties:
    - role: decisionOwner
      name: Chief Information Officer
    - role: consulted
      name: Chief Technology Officer
    - role: approver
      name: AI Governance Council
```

- A decision-right record MUST carry exactly one entry with role `decisionOwner`. A record with none, or with more than one, does not conform at any level.
- `consulted` MAY appear any number of times.

### 5.3 `escalation`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 2 (mx:) |
| **Profile** | aims-decision-right |
| **Conformance** | SHOULD (Level 2) |
| **Default** | *(none)* |

The condition under which a decision leaves its owner, and where it goes.

| Key | Type | Value |
|-----|------|-------|
| `trigger` | string | The condition, in one sentence, under which the decision goes up a level. REQUIRED when `escalation` is present. |
| `to` | string | A `parties` role on the same record, or a reference to the decision-right record of the forum above. REQUIRED when `escalation` is present. |

```yaml
mx:
  escalation:
    trigger: The tool would process personal data, or the owner and the consulted role still disagree.
    to: approver
```

- An agent acting within a decision SHOULD test `trigger` before acting and hand the decision to `to` when it holds.

### 5.4 `decisionRightRef`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | aims-risk-register, aims-statement-of-applicability, aims-impact-assessment |
| **Conformance** | MAY (Level 3) |
| **Default** | *(none)* |

A reference, on a record that carries a sign-off, to the decision right whose owner should have made it. The value is an https URI (normally the decision right's `canonicalUri`), a registry identifier, or a repository-relative path.

```yaml
mx:
  decisionRightRef: https://example.org/aims/decisions/dr-003.md
```

### 5.5 Record identity: `recordId`, `status`, `approvedAt`, `reviewBy`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | aims-decision-right |
| **Conformance** | MUST (Level 1) for `recordId`, `status` and `approvedAt`; SHOULD (Level 2) for `reviewBy` |
| **Default** | *(none - explicit declaration required)* |

| Field | Value |
|-------|-------|
| `recordId` | The organisation's own identifier for the record, stable across its versions. |
| `status` | The record's standing: `active`, `superseded`, `withdrawn` or `draft`. Only an `active` record allocates a decision. |
| `approvedAt` | The date the allocation was agreed, ISO 8601 (YYYY-MM-DD). |
| `reviewBy` | The date by which the allocation is checked again, ISO 8601 (YYYY-MM-DD). |

```yaml
mx:
  recordId: DR-003
  status: active
  approvedAt: 2026-10-19
  reviewBy: 2027-04-19
```

---

## 6. Comparing a sign-off with its decision right

A reader that holds a record carrying `decisionRightRef` and the decision right it names SHALL compare them as follows, and SHALL report one of three verdicts with its reason.

1. **Resolve the right.** Find the record whose identifier or `canonicalUri` equals `decisionRightRef`. If none is held, if it is not of kind `aims:decisionRight`, if its `status` is not `active`, or if it does not name exactly one `decisionOwner`, the verdict is **unresolved**.
2. **Find the signer.** The signing role depends on the record kind: on a risk register entry it is the role named in `residualRisk.acceptedBy`; on a statement of applicability, `approver`; on an impact assessment, `assessor`. If the record names no signing role yet, or no `parties` entry holds it, or more than one does, the verdict is **unresolved**.
3. **Compare people.** When the signer and the owner both carry `identifier`, they are the same person when the identifiers are equal. Otherwise they are the same person when their names are equal after trimming, collapsing internal spaces and ignoring case. An empty name never matches.
4. **Report.** The verdict is **match** when they are the same person and **mismatch** otherwise.

A mismatch reports that two records disagree. It does not say which one is in error: someone other than the owner may have signed, or a delegation may be unrecorded. A reader MUST NOT treat an unresolved verdict as a match.

---

## 7. Worked example

A decision right, and a risk register entry that cites it:

```yaml
---
title: "Decision right: release of high-risk AI uses"
description: "Who approves a high-risk AI use for release."
author: Example Group
created: 2026-10-19
modified: 2026-10-19
type: aims:decisionRight
mx:
  canonicalUri: https://example.org/aims/decisions/dr-007.md
  status: active
  recordId: DR-007
  decisionScope: Whether an AI use rated high risk is approved for release.
  parties:
    - role: decisionOwner
      name: Chief Executive Officer
    - role: consulted
      name: Chief Information Security Officer
  escalation:
    trigger: The use would make decisions about individuals without human review.
    to: https://example.org/aims/decisions/dr-001.md
  approvedAt: 2026-10-19
  reviewBy: 2027-04-19
---
```

```yaml
---
title: "Risk: summaries misstate dosage"
type: aims:riskRegisterEntry
mx:
  recordId: R-014
  status: active
  decisionRightRef: https://example.org/aims/decisions/dr-007.md
  parties:
    - role: riskOwner
      name: Chief Information Security Officer
  residualRisk:
    level: low
    basis: Every summary is reviewed before release.
    acceptedBy: riskOwner
---
```

The signer is the Chief Information Security Officer and the owner is the Chief Executive Officer, so the verdict is **mismatch**: the risk was accepted by a role the decision right lists as consulted.

---

## 8. Security and privacy considerations

- A decision-right record names people or roles. A publisher SHOULD name roles in records it publishes beyond the organisation and keep personal names in records held internally.
- The comparison in §6 reads names; a reader SHOULD NOT treat a name match as proof of identity where an `identifier` is available.
- A record is an assertion by its publisher. A reader that acts on a decision right SHOULD check who published it before relying on the allocation.

---

## 9. References

### 9.1 Normative references

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119): Key words for use in RFCs to Indicate Requirement Levels.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174): Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words.
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html): Date and time format.

### 9.2 Informative references

- [ISO/IEC 42001:2023](https://www.iso.org/standard/81230.html): Information technology, Artificial intelligence, Management system.
- [NIST AI 100-1](https://doi.org/10.6028/NIST.AI.100-1): Artificial Intelligence Risk Management Framework (AI RMF 1.0).
