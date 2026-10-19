---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Agent Cogs note"
docname: draft-cranstoun-mx-agent-cogs
date: 2026-10-19
consensus: false
keyword:
  - mx
  - agents
  - delegation
  - publishing-point
  - cms
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-agent-cogs.md
---

# MX Agent Cogs note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 19 October 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

An agent that acts for someone usually does so under a login. A login answers one question: who signed in. It does not say which agent is acting, which release of it, whose authority it carries, what it may do, what it may spend, which steps must wait for a person, where its actions are recorded, or how its authority is withdrawn. Authentication is not delegation. An agent with nothing but a borrowed login can only present itself as the person who lent it, and the publishing points it acts on cannot tell the two apart.

This note defines three profiles, one for each party to delegated action:

- **`agent`** (record kind `agent-cog`): the record an AI agent carries and presents when it acts, stating its identity, its principal and its mandate.
- **`publishing-point`** (record kind `publishing-point-cog`): the record a CMS, site, feed, API, newsletter or marketplace publishes to state the terms on which it admits agents.
- **`cms`** (record kind `cms-cog`): the record of the content management system that controls one or more publishing points, stating what the running instance emits for machines.

The three share one vocabulary of action. An agent MAY take an action at a publishing point only when its own mandate and the point's terms both name that action, and the action waits for a person when either record says it must.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

### 2.1 Conformance levels

An agent cog SHALL satisfy:

- **Level 1 (Declared):** the record carries `type: agent-cog`, the Zone 1 identity fields, `agentId`, `actingFor` and `mandate`.
- **Level 2 (Accountable):** Level 1 plus `agentVersion`, `actionLog` and `revocationUri`, and, when `mandate` includes `pay`, a `spendLimit`.
- **Level 3 (Bounded):** Level 2 plus `humanApprovalFor` naming every action in `mandate` whose effect cannot be reversed.

A publishing-point cog SHALL satisfy:

- **Level 1 (Declared):** the record carries `type: publishing-point-cog`, the Zone 1 identity fields, `publishingPointKind`, `agentAccess` and `agentActions`.
- **Level 2 (Routed):** Level 1 plus `agentInterface` and `actionLog`, and, when `agentAccess` is `listed`, `admittedAgents`.

A CMS cog SHALL satisfy:

- **Level 1 (Declared):** the record carries `type: cms-cog`, the Zone 1 identity fields, `cmsProduct` and a non-empty `publishingPoints`.
- **Level 2 (Checkable):** Level 1 plus `cmsVersion`, `cmsArchitecture` and `machineOutputs`.

A publishing point conforms when it applies §6 to every agent action it accepts, whatever level the agent's cog reaches.

### 2.2 Status

This note is a draft offered to The Gathering for review. It is not ratified. Implementations SHOULD note the version they implemented against.

---

## 3. Scope

### 3.1 In scope

- The record an agent presents to say who it is, whom it acts for, and what it may do.
- The record a publishing point publishes to say which agents it admits and on what terms.
- The record a content management system carries to say which points it controls and what it emits for machines.
- The behaviour a publishing point owes those records when an agent acts on it.

### 3.2 Out of scope

- Authentication. How an agent proves possession of its identifier (OAuth 2.0, signed requests, mutual TLS or another mechanism) is a deployment choice; this note states what the agent claims, not how the claim is secured.
- Payment rails. `spendLimit` states a ceiling; settling a payment is the business of the payment network.
- What an agent may touch while it acts (files, network, credentials). That belongs to the agent's security declaration, where one exists.
- The content an agent reads. Its metadata is the subject of the **MX Core Metadata note** and its sister notes.

### 3.3 Relationship to existing standards

- The record kinds are values of the OKF `type` field, and the profiles follow the model the **MX Core Metadata note** defines.
- The organisations behind an agent are named in the `parties` field of the **MX Provenance note**: the developer with role `provider`, the organisation that runs the agent with role `deployer`. The owner of a publishing point or CMS is named with role `systemOwner`.
- `agentInterface` MAY name an endpoint described by an Agent2Agent (A2A) agent card or served as a Model Context Protocol server; this note does not redefine either.
- Protocols that authenticate an agent to a business (for example the Personal Agent Protocol) settle the agent's authority to connect. An agent cog states the mandate in the same metadata the content carries, so a publishing point can compare the two without a protocol-specific mapping.

---

## 4. The action vocabulary

`mandate`, `agentActions` and `humanApprovalFor` take their values from one closed list. A reader MUST treat a value outside the list as absent.

| Value | Meaning |
|-------|---------|
| `read` | Retrieve content or data. |
| `draft` | Create or change content that its audience cannot yet see. |
| `write` | Change stored content or records. |
| `submit` | Send a form, an order or a request to another party. |
| `publish` | Make content visible to its audience. |
| `unpublish` | Withdraw published content. |
| `delete` | Remove content or records. |
| `message` | Send a communication to a person. |
| `pay` | Commit money. |

An operation counts as every action whose effect it has, and a publishing point MUST apply §6 to each of them. A `write` that makes content visible to its audience is also `publish`; a `delete` of published content is also `unpublish`; a `submit` that commits money is also `pay`. Changing content its audience cannot see is `draft`, whether or not the content is already stored.

---

## 5. Field definitions

All fields live in Zone 2 (the `mx:` object). A reader MUST NOT read them from the top level of the frontmatter.

### 5.1 `agentId`

| Property | Value |
|----------|-------|
| **Type** | string (URI) |
| **Zone** | 2 (mx:) |
| **Profile** | agent |
| **Conformance** | MUST (Level 1) |

**Definition:** The stable identifier of the agent, as a URI its developer controls.

- One identifier per agent, kept across releases; `agentVersion` names the release.
- The identifier is the developer's claim until something corroborates it.

### 5.2 `actingFor`

| Property | Value |
|----------|-------|
| **Type** | object (`name`, optional `id`) |
| **Zone** | 2 (mx:) |
| **Profile** | agent |
| **Conformance** | MUST (Level 1) |

**Definition:** The principal whose authority the mandate carries.

- The principal MUST NOT be the developer, unless the developer is also the party the agent acts for.
- An agent acting for a private individual SHOULD name that person only as far as the publishing point needs.

### 5.3 `mandate`

| Property | Value |
|----------|-------|
| **Type** | array of §4 values |
| **Zone** | 2 (mx:) |
| **Profile** | agent |
| **Conformance** | MUST (Level 1) |

**Definition:** The kinds of action the agent holds its principal's authority for. An action not listed is outside the mandate.

```yaml
mx:
  agentId: https://agents.example.com/editorial-assistant
  actingFor:
    name: Example News Ltd
    id: https://news.example.com/
  mandate: [read, draft, write]
```

### 5.4 `agentVersion`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | agent |
| **Conformance** | SHOULD (Level 2) |

**Definition:** The release of the agent the cog describes. A mandate given to one release MUST NOT be read as given to another. When `agentVersion` is absent, the cog makes no distinction between releases: its mandate covers whichever release presents it, and a publishing point SHOULD record in its `actionLog` the release the agent reports. A principal who needs a mandate bound to one release sets `agentVersion`.

### 5.5 `operatesAt`

| Property | Value |
|----------|-------|
| **Type** | array of host patterns or URIs |
| **Zone** | 2 (mx:) |
| **Profile** | agent |
| **Conformance** | MAY |

**Definition:** The publishing points the mandate covers. A host pattern is an exact host or a leading wildcard (`*.example.com`). A malformed pattern covers nothing. Absent, the mandate names no venue, and a publishing point that admits only agents mandated for it reads the absence as not covering itself.

### 5.6 `spendLimit`

| Property | Value |
|----------|-------|
| **Type** | object (`amount`, `currency`, `period`) |
| **Zone** | 2 (mx:) |
| **Profile** | agent |
| **Conformance** | MUST when `mandate` includes `pay` (Level 2) |

**Definition:** The most a `pay` mandate may commit. `currency` is an ISO 4217 code; `period` is one of `transaction`, `day`, `week`, `month`.

```yaml
mx:
  mandate: [read, submit, pay]
  spendLimit:
    amount: 500
    currency: GBP
    period: month
```

### 5.7 `humanApprovalFor`

| Property | Value |
|----------|-------|
| **Type** | array of §4 values |
| **Zone** | 2 (mx:) |
| **Profile** | agent, publishing-point |
| **Conformance** | SHOULD (agent Level 3) |

**Definition:** The kinds of action that stop for a named person's approval before they take effect. On an agent cog, the stops the principal set; on a publishing-point cog, the stops the point sets for every agent it admits. Absent, the record declares no stop; a reader MUST NOT take the absence to mean an action is safe to run unattended.

### 5.8 `actionLog`

| Property | Value |
|----------|-------|
| **Type** | string (URI or path) |
| **Zone** | 2 (mx:) |
| **Profile** | agent, publishing-point |
| **Conformance** | SHOULD (Level 2) |

**Definition:** Where actions are recorded, each attributed to the `agentId` and `agentVersion` that took it, the principal it was taken for, and any approval it waited on. On an agent cog, the agent's record; on a publishing-point cog, the point's record of agent actions taken through it. A log SHOULD be append-only, so concurrent entries cannot overwrite one another.

### 5.9 `revocationUri`

| Property | Value |
|----------|-------|
| **Type** | string (URI or path) |
| **Zone** | 2 (mx:) |
| **Profile** | agent |
| **Conformance** | SHOULD (Level 2) |

**Definition:** Where the current status of the mandate is published, so the principal can withdraw it and a publishing point can check it. The record's `expires` field sets the date the mandate lapses without withdrawal.

### 5.10 `publishingPointKind`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | publishing-point |
| **Conformance** | MUST (Level 1) |
| **Valid values** | cms, website, feed, api, newsletter, social-channel, marketplace, document-store |

**Definition:** The kind of publishing point the record describes.

### 5.11 `agentAccess`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | publishing-point |
| **Conformance** | MUST (Level 1) |
| **Valid values** | open, declared, listed, closed |

**Definition:** Which agents the point admits. `open` admits any agent; `declared` admits an agent that presents an agent cog; `listed` admits only the agents in `admittedAgents`; `closed` admits none.

### 5.12 `admittedAgents`

| Property | Value |
|----------|-------|
| **Type** | array of `agentId` values |
| **Zone** | 2 (mx:) |
| **Profile** | publishing-point |
| **Conformance** | MUST when `agentAccess` is `listed` (Level 2) |

**Definition:** The agents the point admits under `listed`. An empty or absent list under `listed` admits no agent.

### 5.13 `agentActions`

| Property | Value |
|----------|-------|
| **Type** | array of §4 values |
| **Zone** | 2 (mx:) |
| **Profile** | publishing-point |
| **Conformance** | MUST (Level 1) |

**Definition:** The most an admitted agent may do at the point. A ceiling, never a grant.

```yaml
mx:
  publishingPointKind: cms
  agentAccess: listed
  admittedAgents:
    - https://agents.example.com/editorial-assistant
  agentActions: [read, draft, write]
  humanApprovalFor: [publish, unpublish, delete]
  agentInterface: https://cms.news.example.com/api/agents
```

### 5.14 `agentInterface`

| Property | Value |
|----------|-------|
| **Type** | string (URI or path) |
| **Zone** | 2 (mx:) |
| **Profile** | publishing-point |
| **Conformance** | SHOULD (Level 2) |

**Definition:** The interface the point offers agents in place of its human interface: an API, a Model Context Protocol server, a feed, or a submission folder.

### 5.15 `cmsProduct`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cms |
| **Conformance** | MUST (Level 1) |

**Definition:** The content management system, as its vendor names it.

### 5.16 `publishingPoints`

| Property | Value |
|----------|-------|
| **Type** | array of URIs or paths |
| **Zone** | 2 (mx:) |
| **Profile** | cms |
| **Conformance** | MUST (Level 1) |

**Definition:** The publishing points the CMS controls, as the locations of their publishing-point cogs, or as the URLs they serve where no cog exists. The list MUST contain at least one entry. A point listed without a cog has made no statement to agents.

### 5.17 `cmsVersion`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cms |
| **Conformance** | SHOULD (Level 2) |

**Definition:** The release of the CMS the instance runs.

### 5.18 `cmsArchitecture`

| Property | Value |
|----------|-------|
| **Type** | string |
| **Zone** | 2 (mx:) |
| **Profile** | cms |
| **Conformance** | SHOULD (Level 2) |
| **Valid values** | coupled, headless, decoupled, static-generator, hybrid |

**Definition:** How the CMS delivers content, which tells an agent where the authoritative copy lives.

### 5.19 `machineOutputs`

| Property | Value |
|----------|-------|
| **Type** | array |
| **Zone** | 2 (mx:) |
| **Profile** | cms |
| **Conformance** | SHOULD (Level 2) |
| **Valid values** | json-ld, mx-frontmatter, llms-txt, sitemap, feed, robots-txt, agent-card, mcp-server, markdown-mirror |

**Definition:** The machine-readable outputs the running instance emits for its publishing points, as configured, not as the product could be configured. The list applies to every point in `publishingPoints`. A CMS whose points emit different outputs SHOULD carry one CMS cog per group of points that share the same outputs, so each list stays true of every point it names. The list is a statement an outside-in audit can check against what the points serve.

---

## 6. Publishing-point behaviour

A publishing point that admits agents MUST, for each action an agent requests, apply these checks in order, and MUST refuse the action at the first check it fails:

1. **Admission.** The agent is admitted under the point's `agentAccess`: any agent under `open`; an agent that presents an agent cog under `declared`; an agent whose `agentId` is in `admittedAgents` under `listed`; none under `closed`.
2. **Binding.** For any action other than `read`, the point has bound the presented `agentId` to the caller it authenticated, and has obtained the agent cog from a source it trusts for that agent: by resolving `agentId` over an authenticated channel, by verifying a signature from the principal or the developer, or through an established relationship with the principal. A cog the point cannot bind and source this way MUST NOT authorise any action beyond `read`.
3. **Venue.** When the cog carries `operatesAt`, the point's own host or address matches one of its entries.
4. **Standing.** The mandate is not past its `expires` date, and is not known to be withdrawn. The point MUST check `revocationUri` before any irreversible action and before any action at high risk, and MUST refuse when the address reports the mandate withdrawn or cannot be reached. For other actions it MAY rely on a status it checked within a period it declares, and SHOULD check at least at the start of each session. Once the point learns a mandate is withdrawn, it MUST refuse every later action under it.
5. **Authority.** Every action the operation counts as (§4) is named in both the agent's `mandate` and the point's `agentActions`.
6. **Spend.** For `pay`, the cog carries a `spendLimit`, the payment is in the limit's currency, and the payment plus every payment already committed under the mandate in the current period does not exceed `amount`. The point MUST count concurrent payments against the same limit so that together they cannot exceed it. A point MUST NOT assume a ceiling in place of a missing `spendLimit`.
7. **Approval.** Any action the operation counts as that either record lists in `humanApprovalFor` waits for a named person's approval.

The point MUST read its own terms from its own copy of the publishing-point cog, never from a value the agent supplies. It SHOULD record each action in its `actionLog` with the agent, its release, the principal and any approval, and SHOULD NOT report an action complete to the agent before that entry is durable.

A publishing point that carries no publishing-point cog has made no statement. An agent MUST NOT read the absence as permission.

---

## 7. Security and privacy considerations

- The records describe authority; they do not enforce it. An agent that ignores a publishing point's terms can still drive its human interface. The value of the records is that the honest route exists and an agent that bypasses it, presenting itself as a person, is plainly outside the terms.
- An agent cog is an assertion until it is bound and sourced as §6 step 2 requires. A self-authored cog that names someone else in `actingFor` is the attack that step exists to stop.
- `actingFor` can identify a private individual. Agents SHOULD disclose only what the point needs; a personal record held under the **MX Personal Cogs note** carries the rest under its own disclosure tier.
- `revocationUri` and `actionLog` are addresses chosen by whoever wrote the agent cog. A point MUST fetch them only over HTTPS, MUST resolve them through a resolver that refuses loopback, private and link-local addresses, and SHOULD accept only addresses on the same host as `agentId` or on a host it has approved. It MUST take them only from a copy of the agent cog it obtained under §6 step 2.
- `machineOutputs` can overstate. An audit that finds a claimed output missing has found a defect in the record or in the instance, and either is worth reporting to the owner.

---

## 8. IANA considerations

This note proposes no new IANA registrations.

---

## 9. References

### 9.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) - Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) - Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [ISO 4217](https://www.iso.org/iso-4217-currency-codes.html) - Currency codes.

### 9.2 Informative

- **MX Core Metadata note** - the Zone 1 / Zone 2 model and the profile mechanism this note uses.
- **MX Provenance note** - the `parties` field and its roles.
- **MX Personal Cogs note** - the record a person keeps about themselves, and its disclosure tiers.
- [Agent2Agent (A2A) protocol](https://a2a-protocol.org/) - agent cards.
- [Model Context Protocol](https://modelcontextprotocol.io/) - tool servers an `agentInterface` may name.
- [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749) - OAuth 2.0, one way an agent may authenticate.

---

## 10. Author quickstart (Informative)

```yaml
---
title: "Editorial assistant"
description: "Drafts and corrects articles in the newsroom CMS for Example News."
author: Jane Doe
created: 2026-10-19
modified: 2026-10-19
type: agent-cog
mx:
  agentId: https://agents.example.com/editorial-assistant
  agentVersion: 2.1.0
  actingFor:
    name: Example News Ltd
  mandate: [read, draft, write, publish]
  humanApprovalFor: [publish]
  actionLog: https://agents.example.com/editorial-assistant/log
  revocationUri: https://agents.example.com/editorial-assistant/status
  expires: 2027-10-19
---
```

```yaml
---
title: "Newsroom CMS"
description: "The CMS behind Example News: what it runs, what it controls, and what it emits for machines."
author: Jane Doe
created: 2026-10-19
modified: 2026-10-19
type: cms-cog
mx:
  cmsProduct: Example CMS
  cmsVersion: "10.4"
  cmsArchitecture: headless
  publishingPoints:
    - https://news.example.com/.well-known/publishing-point.md
  machineOutputs: [json-ld, sitemap, feed, llms-txt]
---
```

---

## 11. Common authoring mistakes (Informative)

```yaml
# WRONG - the developer named as principal: the mandate's authority is unstated
mx:
  actingFor:
    name: Example Agents Ltd   # the company that built the agent
```

```yaml
# WRONG - a pay mandate with no ceiling: a conforming point refuses it
mx:
  mandate: [read, submit, pay]
```

```yaml
# WRONG - listed with no list: no agent is admitted
mx:
  agentAccess: listed
  agentActions: [read, draft]
```

## 12. Open questions for review

1. Is the action vocabulary in §4 the right closed list, and how should it be extended?
2. Should `agentId` require a resolvable URI that serves the agent cog, or may it be an opaque identifier?
3. Should §6 step 4 require a fresh `revocationUri` check before every action, at the cost of a request per action?
4. Should `machineOutputs` keep its vocabulary in this note, or defer to a registry of output kinds?
