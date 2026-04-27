---
title: "MX Workflow Contracts note"
docname: draft-cranstoun-mx-workflow-contracts
date: 2026-04-27
consensus: false
keyword:
  - mx
  - workflow
  - contracts
  - approvals
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
---

# MX Workflow Contracts note

**Version:** 1.0-draft
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 27 April 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies a small set of optional public-extension fields used by **workflow contract cogs** — documents that declare an executable approval, review, or procedural workflow.

The fields defined here are top-level (Zone 1) so that they form part of the cog's signed contract surface: their values name the limits, the people, and the procedures that the cog promises to honour. The shape of each field's object is domain-specific; an explicit `schema` reference on the cog (a top-level pointer to a JSON Schema, YAML schema, or Schema.org type) is the authoritative source for the precise structure.

All fields in this note are conformance level MAY. They apply only to documents that explicitly declare a workflow contract.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

A workflow contract cog is **conformant** with this note if every field it declares from §3 satisfies the field's own type and placement requirements (top-level Zone 1 placement, type as declared, value-shape consistent with the cog's `schema` reference where present).

A document that does not declare any of the fields in §3 makes no claim against this note and is unaffected by it.

### 2.1 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions, conformance requirements, and normative rules in this note are working drafts — stable enough to build against, expected to evolve through review.

---

## 3. Workflow contract fields

All fields in this section appear at the **top level** of the cog's frontmatter, not under `mx:`, so that they form part of the cog's signed contract surface.

### 3.1 `x-mx-thresholds`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 1 (top-level) |
| **Profile** | x-mx-public |
| **Conformance** | MAY |

**Definition:** Domain workflow thresholds attached to a contract cog. Object whose keys are domain-named limits (e.g. `autoApproveBelow`, `requireSecondaryAbove`, `escalateAt`) and whose values are numeric, monetary, or categorical bounds. The cog's referenced `schema` specifies which keys are required and what types they accept.

**Example:**

```yaml
x-mx-thresholds:
  autoApproveBelow: 1000
  requireSecondaryAbove: 10000
```

### 3.2 `x-mx-approvers`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 1 (top-level) |
| **Profile** | x-mx-public |
| **Conformance** | MAY |

**Definition:** Mapping of approver-role names to identities. Keys are role names (`primary`, `secondary`, `finance`, `legal`, ...). Values are user identities, group references, or role descriptors resolved by the cog runtime.

**Example:**

```yaml
x-mx-approvers:
  primary: line-manager
  secondary: finance-director
```

### 3.3 `x-mx-approvalProcedure`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 1 (top-level) |
| **Profile** | x-mx-public |
| **Conformance** | MAY |

**Definition:** Approval procedure declaration. Object with `runner` (the engine that executes the procedure) and `phases` (an ordered array of phase objects, each typically with an `id` and optional inputs).

**Example:**

```yaml
x-mx-approvalProcedure:
  runner: cog-runtime
  phases:
    - id: triage
    - id: review
    - id: sign-off
```

### 3.4 `x-mx-reviewProcedure`

| Property | Value |
|----------|-------|
| **Type** | object |
| **Zone** | 1 (top-level) |
| **Profile** | x-mx-public |
| **Conformance** | MAY |

**Definition:** Review procedure declaration. Same shape as `x-mx-approvalProcedure` (`runner` + `phases`) applied to review rather than approval workflows.

**Example:**

```yaml
x-mx-reviewProcedure:
  runner: cog-runtime
  phases:
    - id: collect-feedback
    - id: revise
    - id: publish
```

### 3.5 `x-mx-targetEnvironment`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 1 (top-level) |
| **Profile** | x-mx-public |
| **Conformance** | MAY |

**Definition:** Named deployment or runtime environment a contract cog targets (e.g. `production`, `staging`, `test`). Resolved by the cog runtime to apply environment-specific settings.

**Example:**

```yaml
x-mx-targetEnvironment: production
```

### 3.6 Group rules

The following rules apply across all fields in §3:

- These fields are typically named in a `contractFields` array on the cog so that changes to thresholds, approvers, procedures, or target environment invalidate any signature attached to the cog.
- Implementations MUST resolve the cog's `schema` reference before validating the inner shape of these objects; the schema is authoritative for required keys.
- The `runner` value in `x-mx-approvalProcedure` and `x-mx-reviewProcedure` MUST be a registry-resolvable runtime identifier.

---

## 4. Out of scope

- **Namespace policy** — the `x-mx-` and `x-mx-p-` prefix conventions used by these fields are governed externally.
- **Signing mechanics** — the canonical-JSON, hash, and signature procedure that protects a workflow contract is governed externally. This note specifies only the contract-content fields, not the cryptography.
- **Per-domain schemas** — the precise required keys and value types for `x-mx-thresholds`, `x-mx-approvers`, `x-mx-approvalProcedure`, and `x-mx-reviewProcedure` are domain-specific. They belong in the schema referenced by the cog, not in this note.
- **Runner implementations** — the engines named by `runner` are out of scope. This note does not specify how `cog-runtime` or any other runner executes phases.
- **Identity resolution** — how an approver name (`line-manager`, `finance-director`) resolves to a real user, group, or service is the runtime's responsibility, not the contract format's.

---

## 5. Security and privacy considerations

- Workflow contracts may name individuals by role title or identity. Authors SHOULD consider whether publishing a contract cog exposes identifying information about staff, approval limits, or organisational structure.
- A workflow contract is only as trustworthy as the signature attached to it, if any. A cog declaring `x-mx-thresholds` without a signature carries no integrity guarantee — the threshold values may have been altered after authoring.
- Threshold values can encode commercially sensitive information (auto-approval limits, escalation triggers). Operators SHOULD treat unsigned, unprotected contract cogs as advisory only.

---

## 6. References

### 6.1 Normative references

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to indicate requirement levels

### 6.2 Informative references

- [JSON Schema](https://json-schema.org/) — schema referenced by `schema` for object shape validation
- [Schema.org](https://schema.org/) — alternative schema vocabulary for `schema` references
- [WCAG 2.1](https://www.w3.org/TR/WCAG21/) — Web Content Accessibility Guidelines (conformance level model)

---

## 7. Changelog

- **2026-04-27 (v1.0-draft)** — Initial draft. Five workflow-contract fields extracted from the MX Extensions note (where they were §10.4 in v1.1-proposed) into a standalone note: `x-mx-thresholds`, `x-mx-approvers`, `x-mx-approvalProcedure`, `x-mx-reviewProcedure`, `x-mx-targetEnvironment`. All Zone 1 (top-level), conformance MAY.

---

*End of MX Workflow Contracts note draft.*
