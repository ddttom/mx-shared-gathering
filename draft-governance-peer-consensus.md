---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX governance note: peer consensus and community ownership"
docname: draft-cranstoun-mx-governance-peer-consensus
date: 2026-06-19
consensus: false
keyword:
  - mx
  - governance
  - peer-consensus
  - community-ownership
  - standards
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-governance-peer-consensus.md
---

# MX governance note: peer consensus and community ownership

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 19 June 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note records the governance design choice at the foundation of The Gathering: MX standards are ratified by community consensus among practitioners who have deployed them, not authored by an authority and published as a finished artefact. The note explains why this design choice was made, what governance model it produces, and why the rise of AI makes this approach more important, not less.

The central claim is this: AI has made expert advice abundant. A practitioner can obtain a competent senior-level analysis of any standards question in seconds. What AI cannot produce is the honest peer perspective - the account of someone who has deployed the standard and will report what they actually found. Community-owned standards exist to carry that perspective into the ratified record.

---

## 2. Status of this note

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. It describes the governance design choices already embedded in The Gathering's operating model; it does not specify new fields, protocols, or conformance requirements.

---

## 3. The governance model

The Gathering operates on the rough-consensus model in the lineage of the W3C and IETF. The key properties are:

- **Open drafts.** Draft notes are published openly for community review. Reading is open to anyone.
- **Rough consensus.** A draft progresses to ratification when no practitioner who has reviewed it holds a sustained, substantiated objection. Consensus is not unanimity; it is the absence of unresolved objection.
- **Community ownership.** The founder of The Gathering does not own it. The community that ratifies its standards owns them. No standard depends on any single authority's continued participation for its legitimacy.
- **Deferral to existing standards.** The Gathering never duplicates what existing open standards (W3C, IETF, Schema.org, Dublin Core, ISO, NIST) already specify. It fills gaps and records agreements about how existing standards combine.

This model is not chosen for cultural reasons. It is chosen because it produces a different kind of weight. A standard ratified by a community of practitioners who have deployed it carries the authority of collective experience. A standard published by a single authority - however competent - carries only that authority's opinion. At the decision point where a practitioner must choose what to build against, these are categorically different.

---

## 4. Why AI makes this more important

AI systems have changed the cost structure of expertise. A practitioner seeking guidance on metadata standards can now obtain senior-level analysis in seconds at negligible cost. The question "where do I find someone who knows?" has been answered, permanently and at scale.

The question AI cannot answer is: "where do I find someone who has been through this and will tell me what actually happened?"

Expert knowledge is correct. Peer knowledge is tested. These are not the same thing, and the difference matters most in a domain - like metadata standards - where the failure modes are subtle, emerge slowly, and only become visible once a standard has been deployed at scale. A standard that passes formal review but breaks under real-world implementation conditions is a standard the community has not yet ratified in any meaningful sense.

The right response to abundant AI-sourced expertise is not to produce more expert-authored standards. It is to build the infrastructure that collects, ratifies, and publishes the peer-tested alternative: what deployers found when the expert advice met the real world.

That is what The Gathering does.

---

## 5. Design principles that follow

Three principles follow from this governance design:

**Community ratification is the legitimacy mechanism.** The Gathering's standards derive their authority from the community of practitioners who have reviewed and ratified them - not from the identity of their authors, the credentials of their sponsors, or the approval of any single organisation.

**The ratification record is the audit trail.** Who reviewed a draft, what objections were raised, how objections were resolved, and when rough consensus was declared is part of the permanent record. A practitioner or auditor who needs to understand why a standard specifies what it does can trace the decision back to the ratification record.

**Peer participation is the service, not the governance overhead.** Participating in The Gathering's review process - reading drafts, raising objections, confirming deployment experience - is the primary value the community receives, not an obligation to earn access to the ratified result. The act of review is where the learning happens.

---

## 6. Acknowledgements

The governance model described in this note is informed by the W3C Process Document, the IETF's practice of rough consensus and running code, and by the observation - articulated in conversation by a practitioner convening a peer-learning community for digital professionals - that in a world where AI has made expertise abundant, the honest peer perspective is the scarce resource worth building institutional infrastructure to preserve.
