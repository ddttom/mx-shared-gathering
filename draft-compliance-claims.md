---
# cog v1 spec=https://mx.allabout.network/cog.html runtime=https://mx.allabout.network/cog-runtime.html
title: "MX Compliance Claims note"
docname: draft-cranstoun-mx-compliance-claims
date: 2026-05-08
consensus: false
keyword:
  - mx
  - compliance
  - claims
  - attestation
  - verification
author:
  - fullname: Tom Cranstoun
    organization: CogNovaMX
    email: info@cognovamx.com
canonicalUri: https://raw.githubusercontent.com/ddttom/mx-shared-gathering/main/draft-compliance-claims.md
---

# MX Compliance Claims note

**Version:** 1.0
**Status:** Draft by Tom Cranstoun, offered to The Gathering for review
**Date:** 8 May 2026
**Author:** Tom Cranstoun
**License:** MIT

---

## 1. Abstract

This note specifies how a digital artefact carries a verifiable claim of conformance to a named external standard. It defines the structure of a compliance claim, the authority model under which a claim is issued, the evidence it must reference, and the rules for expiry and revocation.

A compliance claim is a signed, dated, revocable assertion that a predicate holds true about a subject. The subject is an addressable artefact (a web page, a PDF, a video, an MX cog). The predicate names a published external standard and the conformance level reached. The authority is the party issuing the claim. The evidence is the documented basis on which the claim was made.

The note is deliberately framework-agnostic: it specifies the primitive — a signed, dated, revocable claim about a digital artefact — not any single regulator's rulebook. Mappings from this primitive to specific regimes (WCAG, EU AI Act, sector-specific compliance) are layered on top through predicate vocabularies that the claim references but does not embed.

The note motivates two authority types from the outset: **self-attestation**, where the publisher of the artefact issues a claim about their own work, and **third-party attestation**, where an independent party issues a claim about an artefact they did not produce. Both types use the same claim structure and signature mechanism. The standard specifies what each authority type MUST declare; it does not specify the procedures by which any specific accreditation body admits, monitors, or revokes third-party authorities, leaving that to operators of accreditation programmes built on this standard.

---

## 2. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) and [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

Field definitions in this note conform to the **MX Field Definition Pattern note** (the primary note of this draft set). That pattern governs the structural template for every field and is binding on this note. This note adopts the pattern's authoring rules and does not restate them.

### 2.1 Conformance levels

A compliance claim claiming conformance to this note SHALL satisfy:

| Level | Name | Claim requirement |
|-------|------|-------------------|
| Level 1 | **Self-Attested** | A signed claim with subject identity and hash, predicate identifier and version, authority of type `self` with verifiable signing identity, issuedAt timestamp |
| Level 2 | **Verified** | Level 1 plus documented evidence (method and report URI with report hash), expiresAt timestamp, and a revocation list URI |
| Level 3 | **Independently Attested** | Level 2 plus an authority of type `third-party` carrying a declared accreditation reference, where the third-party authority is distinct from the subject's publisher |

A claim of conformance SHALL state the level reached. A claim that does not reach Level 1 is not a claim under this standard.

The three levels are strictly nested: a Level 3 claim satisfies all Level 2 and Level 1 requirements. A consumer of claims MAY reject lower-level claims for high-stakes evaluations, and MAY accept Level 1 claims for low-stakes evaluations such as user-facing badges.

### 2.2 Draft status

This note is a draft authored by Tom Cranstoun and offered to The Gathering for review. It is not a ratified standard. Until The Gathering accepts it, the field definitions, conformance requirements, and normative rules in this note are working drafts — stable enough to build against, expected to evolve through review. Implementations of this draft SHOULD note the draft version number they implemented against.

---

## 3. Scope

### 3.1 In scope

- The structure of a compliance claim: required and optional fields, types, and serialisation.
- The authority model: `self` and `third-party` authority types, what each MUST declare, and how a third-party authority's accreditation is referenced.
- The evidence model: how a claim points to a documented basis without embedding it.
- The expiry and revocation model: how a claim becomes inert.
- The signature contract: which fields are covered, in what canonical form.
- The relationship between a claim and the artefact it is about: subject identification and hash binding.
- Predicate vocabulary requirements: what a published predicate vocabulary MUST declare for a claim to reference it.
- A worked example mapping the claim structure to the [WCAG 2.2](https://www.w3.org/TR/WCAG22/) Level AA conformance claim.

### 3.2 Out of scope

- **Predicate vocabulary content.** This note does not enumerate the predicates of any specific regulatory regime. WCAG, EU AI Act, sector-specific frameworks, and similar regimes are referenced as predicate vocabularies but are not specified here.
- **Accreditation programmes for third-party authorities.** Who admits a third-party authority, what bar they must meet, how their accreditation is monitored and revoked — these are operational concerns for accreditation programmes built on this standard, not for the standard itself.
- **Cryptographic mechanics of signing and verification.** Canonicalisation, signature algorithms, and signature verification procedures defer to the **MX Contract Fingerprinting and Signing note**.
- **Provenance of the artefact.** Who created the artefact, when, and on what basis is governed by the **MX Provenance note**. A compliance claim assumes provenance metadata may be present but does not redefine it.
- **Registry mechanics.** How claims are stored, indexed, and looked up by consumers is implementation-defined. This note specifies what a claim contains and how it is verified, not where it lives.
- **Badge presentation.** How a claim is rendered as a user-facing visual badge is implementation-defined.

### 3.3 Relationship to existing standards

This note relies on:

- **[RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119)** and **[RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174)** for normative key words.
- **[RFC 7515 JSON Web Signature](https://datatracker.ietf.org/doc/html/rfc7515)** for the baseline signature serialisation.
- **[RFC 8032 EdDSA](https://datatracker.ietf.org/doc/html/rfc8032)** as the RECOMMENDED signature algorithm.
- **[RFC 8615 Well-Known URIs](https://datatracker.ietf.org/doc/html/rfc8615)** for revocation list discovery.
- **[ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)** for date and time fields.
- **[W3C Decentralized Identifiers (DID) v1.0](https://www.w3.org/TR/did-core/)** for authority identifier syntax (informative; the field accepts URIs more broadly).
- **[C2PA Content Credentials v1.x](https://c2pa.org/specifications/specifications/1.0/specs/C2PA_Specification.html)** as the recommended interoperability target for content provenance attestations specifically; a claim about content provenance MAY embed or reference a C2PA manifest.

This note relies on the **MX Field Definition Pattern note** for field-definition discipline, the **MX Contract Fingerprinting and Signing note** for the cryptographic contract over the claim payload, and the **MX Provenance note** for prior-art on the difference between a content origin record and a conformance assertion.

This note adds nothing to the standards above; it specifies how they combine for the verifiable-compliance-claim case.

---

## 4. Claim structure

A compliance claim is a structured object carried alongside or within a digital artefact. Implementations MAY serialise the claim as JSON, YAML, or as a JWS payload over JSON; the field set and semantics in this section are normative across serialisations.

### 4.1 Top-level claim object

A claim object MUST declare the following top-level fields:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `claimId` | required | string | Stable, globally unique identifier for the claim. RECOMMENDED form: a URI under the issuing authority's domain. |
| `subject` | required | object | Identification of the artefact the claim is about. See §4.2. |
| `predicate` | required | object | Identification of the conformance assertion. See §4.3. |
| `evidence` | required at Level 2+ | object | Documented basis for the claim. See §4.4. |
| `authority` | required | object | Identification of the issuing party. See §5. |
| `issuedAt` | required | string (ISO 8601) | Date and time the claim was issued. |
| `expiresAt` | required at Level 2+ | string (ISO 8601) | Date and time the claim ceases to be valid without further action. |
| `revocation` | required at Level 2+ | object | Pointer to the revocation list under which this claim's current status can be checked. See §4.5. |
| `signature` | required | object | The signature over the canonical claim. See §4.6. |

Implementations MAY define additional fields under an extension prefix per the **MX Extensions note**. Implementations MUST NOT redefine or override the fields above.

### 4.2 Subject

The `subject` object identifies the artefact the claim is about. It MUST declare:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `uri` | required | string (URI) | Addressable identifier of the artefact. For web artefacts, the canonical URL. For non-web artefacts, a URI of any scheme that can be resolved by a consumer. |
| `hash` | required | string | Content hash of the artefact at the moment of claim issuance. |
| `hashAlgorithm` | required | string | The hashing algorithm used. RECOMMENDED: `sha-256`. |
| `mediaType` | RECOMMENDED | string | The artefact's media type at the moment of claim issuance. |

Where the artefact is an MX cog with a `schema:` annotation, the `hash` field MAY be the contract fingerprint defined by the **MX Contract Fingerprinting and Signing note**. Where it is not, the `hash` field MUST be the hash of the artefact's full byte sequence.

A claim is bound to a specific byte sequence. If the artefact is modified, the prior claim does not transfer; a new claim against the new hash is required. This is intentional and not an implementation defect.

### 4.3 Predicate

The `predicate` object identifies the conformance assertion: which standard, which level, which version. It MUST declare:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `vocabulary` | required | string (URI) | URI of the published predicate vocabulary the claim references. |
| `vocabularyVersion` | required | string | The version of the predicate vocabulary the claim references. |
| `claim` | required | string | An identifier defined by the predicate vocabulary, naming the specific assertion being made. |
| `parameters` | OPTIONAL | object | Vocabulary-defined parameters that further qualify the claim (for example a conformance level). |

A predicate vocabulary is any document that declares one or more `claim` identifiers, each with a defined meaning. A predicate vocabulary published outside this standard MUST declare:

- A versioned URI under which it is reachable.
- For each `claim` identifier: its name, its meaning, the parameters it accepts, and the evidence requirements that an issuer of this `claim` SHOULD meet.

The standard does not constrain who publishes a predicate vocabulary. A regulatory body, a standards organisation, or a sector consortium MAY publish a vocabulary. Consumers of claims judge a claim's weight partly by the vocabulary's source.

### 4.4 Evidence

The `evidence` object documents the basis for the claim. It is OPTIONAL at Level 1 and REQUIRED at Level 2 and above. Where present, it MUST declare:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `method` | required | string (enum) | One of: `selfTest`, `automatedScan`, `manualReview`, `independentAudit`. The method by which the claim's basis was established. |
| `report` | required | string (URI) | Pointer to the evidence report. The report MUST be retrievable by a consumer authorised to evaluate the claim. |
| `reportHash` | required | string | Content hash of the evidence report at issuance time. |
| `reportHashAlgorithm` | required | string | The hashing algorithm used for `reportHash`. |
| `tooling` | OPTIONAL | array of strings | Identifiers (name and version) of any tools used to produce the evidence. |

The `method` enum is not exhaustive; predicate vocabularies MAY define additional values. Vocabulary-defined values MUST be prefixed with the vocabulary's namespace per the **MX Extensions note** to avoid collision.

### 4.5 Revocation

The `revocation` object provides a mechanism for invalidating a claim before its `expiresAt`. It is REQUIRED at Level 2 and above. Where present, it MUST declare:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `listUri` | required | string (URI) | URI of the revocation list under which this claim's current status is published. |
| `checkInterval` | RECOMMENDED | string (ISO 8601 duration) | Recommended polling interval for high-assurance consumers. |

A revocation list is a signed document published by the issuing authority listing all revoked `claimId` values currently within their `expiresAt` window. The signature on the revocation list MUST be verifiable using the same authority identity declared in the claim's `authority.identity` field.

A consumer treating the claim as authoritative SHOULD check the revocation list before relying on the claim. A consumer that does not check the revocation list accepts the risk that a revoked claim is still being treated as valid.

### 4.6 Signature

The `signature` object carries the cryptographic signature over the canonical claim. It MUST declare:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `algorithm` | required | string | The signature algorithm. RECOMMENDED: `Ed25519`. |
| `value` | required | string | The signature value, base64url-encoded. |
| `publicKey` | RECOMMENDED | string | The public key under which the signature can be verified, in PEM or JWK form. MAY be omitted where the public key is discoverable from `authority.identity`. |

The canonical form of the claim payload over which the signature is computed is defined by the **MX Contract Fingerprinting and Signing note**. The `signature` object itself is not part of the canonical payload.

---

## 5. Authority model

The `authority` object identifies the party issuing the claim. The standard defines two authority types: `self` and `third-party`. Both types use the same `authority` object structure. The semantics of each type, and what each MUST declare, are specified below.

### 5.1 Authority object

The `authority` object MUST declare:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `type` | required | string (enum) | Either `self` or `third-party`. |
| `identity` | required | string (URI) | Identifier of the issuing party. RECOMMENDED forms: a DID, a URI under the issuer's domain, or an https URL serving the issuer's verifiable identity document. |
| `name` | RECOMMENDED | string | Human-readable name of the issuer. |
| `accreditation` | required when `type` is `third-party` | object | See §5.3. |

### 5.2 Self-attestation

An authority of type `self` is the publisher of the subject artefact attesting to a property of their own work. Self-attestation is the simplest authority type and the only one available at Level 1 of conformance.

A self-attesting authority MUST satisfy:

- The `authority.identity` field MUST resolve to an identity verifiably associated with the publisher of the subject artefact. A consumer MUST be able to confirm, by a means defined by the publisher's identity scheme, that the signing identity is the publisher's.
- The signature on the claim MUST be produced by a key controlled by the publisher.

Self-attested claims carry the assurance level of the publisher's identity. They are appropriate for user-facing badges and low-stakes consumer signals. They are not appropriate for evidence-trail use cases where the consumer's interest is in independent confirmation of the publisher's claim.

### 5.3 Third-party attestation

An authority of type `third-party` is a party distinct from the subject artefact's publisher, attesting to a property they have evaluated. Third-party attestation is REQUIRED for Level 3 conformance.

A third-party authority MUST satisfy:

- The `authority.identity` field MUST resolve to an identity verifiably distinct from the publisher of the subject artefact.
- The signature on the claim MUST be produced by a key controlled by the third-party authority and not by the subject's publisher.
- The `authority.accreditation` object MUST be present and MUST declare:

| Field | Cardinality | Type | Description |
|-------|-------------|------|-------------|
| `body` | required | string (URI) | URI of the accrediting body that certifies the third-party authority for the predicate vocabulary in question. |
| `reference` | required | string | Identifier under which the third-party authority's accreditation is recorded by the accrediting body. |
| `validFrom` | required | string (ISO 8601) | Date the accreditation became effective. |
| `validUntil` | required | string (ISO 8601) | Date the accreditation expires. |
| `lookupUri` | RECOMMENDED | string (URI) | URI under which a consumer can verify the accreditation in real time. |

The standard does not specify who runs an accrediting body, what bar a third-party authority MUST meet to be accredited, or how an accrediting body monitors and revokes accreditations. These are operational concerns for accreditation programmes built on this standard. The standard requires only that the claim declare the accreditation reference verifiably, so that a consumer can decide whether to trust the accrediting body.

A consumer evaluating a Level 3 claim SHOULD:

1. Verify the claim's signature against the third-party authority's identity.
2. Resolve the accreditation reference and confirm the accreditation is current (within `validFrom` and `validUntil` and not revoked).
3. Independently judge whether the accrediting body is one whose accreditations the consumer accepts.

The third step is the consumer's own decision and is not constrained by the standard. A consumer MAY accept claims accredited by some bodies and not others.

### 5.4 Authority hash separation

The standard intentionally separates authority verification (the claim is signed by the declared issuer) from accreditation verification (the issuer is accredited to make this kind of claim). Both verifications are independent. A claim MAY be signed by a verifiable identity that is not accredited, in which case it is a malformed Level 3 claim and consumers SHOULD treat it as Level 1 at most.

---

## 6. Predicate vocabularies

A predicate vocabulary defines the meaning of `predicate.claim` identifiers. This note does not constrain the content of any predicate vocabulary; it specifies the structural requirements a vocabulary MUST satisfy to be referenced by a compliant claim.

### 6.1 Vocabulary requirements

A predicate vocabulary that is referenced by a compliance claim MUST:

- Be reachable at a stable, versioned URI.
- Carry an explicit version identifier that distinguishes successive revisions.
- For each defined `claim` identifier, declare:
  - The identifier's exact spelling and case.
  - The conformance condition the identifier asserts.
  - The parameters the identifier accepts (with their types and meanings).
  - The evidence method or methods that are appropriate for the identifier (one or more of the values in §4.4 or vocabulary-defined extensions).

Vocabularies SHOULD additionally declare:

- A normative reference to the external standard the vocabulary describes (the standard whose conformance the claims assert).
- An expiry policy stating the recommended `expiresAt` window for claims using the vocabulary.

A claim MUST cite the vocabulary version it was issued against. Vocabularies SHOULD use semantic-style versioning so consumers can reason about compatibility across versions.

### 6.2 Worked example: WCAG 2.2 Level AA

The following illustrates a Level 2 self-attested claim asserting that a web page meets the [WCAG 2.2](https://www.w3.org/TR/WCAG22/) Level AA conformance requirements as evaluated by an automated scan.

```yaml
claimId: https://example.com/.well-known/mx/claims/2026-05-08-home-wcag22-aa
subject:
  uri: https://example.com/
  hash: 4b5c0a8d3e7f1c9b6a2d4e8f0c1b3a5d7e9f1c3b5a7d9e1f3c5b7a9d1e3f5c7b
  hashAlgorithm: sha-256
  mediaType: text/html
predicate:
  vocabulary: https://example.org/vocabularies/wcag-2.2-mx.v0.1
  vocabularyVersion: "0.1"
  claim: wcagConformance
  parameters:
    level: AA
    wcagVersion: "2.2"
evidence:
  method: automatedScan
  report: https://example.com/.well-known/mx/evidence/2026-05-08-home-wcag22-aa.json
  reportHash: 9a1c3e5b7d9f1a3c5e7b9d1f3a5c7e9b1d3f5a7c9e1b3d5f7a9c1e3b5d7f9a1c
  reportHashAlgorithm: sha-256
  tooling:
    - "axe-core 4.9.1"
authority:
  type: self
  identity: https://example.com/.well-known/mx/identity.json
  name: "Example Publisher Ltd"
issuedAt: 2026-05-08T10:30:00Z
expiresAt: 2027-05-08T10:30:00Z
revocation:
  listUri: https://example.com/.well-known/mx/revocations.json
  checkInterval: P1D
signature:
  algorithm: Ed25519
  value: "qS9PvL7eZ0kTu4yX3fA1nM6dC8bV2lH5gW0rJ4iE7sQ_..."
```

The same claim, issued by an independent third-party authority accredited by an accreditation body, would carry an `authority` object of:

```yaml
authority:
  type: third-party
  identity: did:web:auditor.example.org
  name: "Independent Audit Co"
  accreditation:
    body: https://accreditation-body.example.org/
    reference: "ACR-2026-7842"
    validFrom: 2026-01-01
    validUntil: 2027-12-31
    lookupUri: https://accreditation-body.example.org/registry/ACR-2026-7842
```

This second form satisfies Level 3 of conformance because the `authority.type` is `third-party`, the `authority.identity` is distinct from the subject's publisher, and a verifiable accreditation reference is declared. A consumer who accepts accreditations from `accreditation-body.example.org` would treat this as a stronger evidence trail than the self-attested form.

The standard does not name the WCAG predicate vocabulary's publisher. A vocabulary mapping WCAG 2.2 conformance into the MX claims structure may be published by W3C, by an accessibility consortium, by an accreditation body, or by any party meeting §6.1's requirements. Consumers judge weight by the vocabulary's provenance.

### 6.3 Vocabulary discoverability

A claim MAY include a copy of the predicate vocabulary's metadata (its title, version, and publisher) inline as informational fields under an extension prefix. Inline copies are not normative; the canonical reference is the `predicate.vocabulary` URI. A consumer MUST resolve the URI to determine the binding meaning of the claim.

---

## 7. Why a separate compliance-claims standard

A reader may reasonably ask whether existing standards already cover this ground. They cover adjacent ground; none cover this exactly.

[C2PA Content Credentials](https://c2pa.org/) specify how a content asset carries claims about its origin and modification history. They are tightly bound to media files (images, video, audio) and the C2PA manifest format. The MX Compliance Claims structure is broader: it covers any addressable artefact (a web page, an HTML document, an MX cog, a PDF, a dataset) and any predicate vocabulary (not only provenance assertions). A claim under this standard MAY embed a C2PA manifest as evidence, and a C2PA manifest MAY be the basis for a claim issued under this standard. The two are complementary.

[Verifiable Credentials (W3C VC) v2.0](https://www.w3.org/TR/vc-data-model-2.0/) specify a general model for credentials: an issuer asserts that a subject has properties, and the assertion is signed and verifiable. The VC model is more general than what a compliance claim needs and less specific about the dimensions the compliance use case requires (subject hashing, evidence URIs and report hashes, expiry-and-revocation defaults, predicate vocabularies as a first-class concept). A compliance claim under this note MAY be expressed as a Verifiable Credential by mapping the fields above to the VC model; the standard does not require that mapping but does not prohibit it.

[XAdES](https://www.w3.org/TR/XAdES/) and [PAdES](https://www.etsi.org/deliver/etsi_en/319100_319199/31914201/) specify advanced electronic signature formats for XML and PDF respectively. They describe how a signature is formatted and bound to the carrier. They do not specify the claim being signed. A compliance claim under this note MAY be signed using PAdES inside a PDF, but the claim's structure (subject, predicate, evidence, authority, expiry, revocation) is what this note defines.

Regulatory regimes (the EU AI Act's Article 13 transparency obligations, the European Accessibility Act's conformity assessments, sector-specific compliance) prescribe what evidence a regulated party MUST produce. They do not prescribe a portable, machine-readable, signed envelope for that evidence. The MX Compliance Claims structure is the envelope. A given regulatory regime is one possible predicate vocabulary; the standard supports any.

The combination of these gaps is the reason for a separate note. A reader who needs only one of these capabilities should use the matching existing standard. A reader who needs the combination — a portable, signed, dated, revocable, predicate-flexible claim about an arbitrary digital artefact — has not, until this note, had a single standard to point to.

---

## 8. Implementation guidance (informative)

This section is informative; it is not a normative part of the standard.

### 8.1 Claim discovery

A consumer who has an artefact's URI but not yet a claim about it can discover claims through several patterns:

- **Well-known URI.** The artefact's host serves a `claims.json` (or similar) under `/.well-known/mx/` listing claims about the host's content. RFC 8615 governs the well-known prefix.
- **In-band reference.** The artefact carries a `link rel="mx-compliance-claim"` (HTML), an XMP property (PDF), or an analogous carrier-format mechanism per the **MX Carrier Formats note** pointing at the claim.
- **Registry lookup.** A claim registry indexes claims by subject URI; the consumer queries the registry. This pattern is not part of the standard but is a common implementation.

The standard does not mandate a discovery mechanism; an artefact MAY be served with claims through any combination of the above.

### 8.2 Issuance pipeline

A typical issuance pipeline:

1. The publisher freezes the artefact (commits the bytes, generates the hash).
2. The publisher (for self-attestation) or the third-party authority (for third-party attestation) runs the evidence method against the artefact and produces an evidence report.
3. The issuer composes the claim object: subject, predicate, evidence, authority, issuedAt, expiresAt, revocation pointer.
4. The issuer canonicalises the claim payload per the **MX Contract Fingerprinting and Signing note**, signs it with their key, and attaches the signature.
5. The issuer publishes the claim alongside the artefact (or in a registry pointed at by the artefact).
6. The issuer retains the signing material and is prepared to publish revocations on the declared revocation list.

For high-volume publishers, steps 1 to 4 are commonly automated as part of a build pipeline. For audit-grade third-party claims, steps 2 and 3 are manual and reviewed.

### 8.3 Verification pipeline

A typical verification pipeline:

1. Resolve the claim from the artefact (or from the claim's own URI).
2. Recompute the subject hash from the artefact and confirm it matches `subject.hash`. If it does not, the claim does not apply to the bytes the consumer has.
3. Resolve `authority.identity` and obtain the public key.
4. Verify the signature on the canonical claim payload.
5. For Level 2+: confirm `issuedAt <= now <= expiresAt`.
6. For Level 2+: fetch the revocation list from `revocation.listUri`, verify its signature, and confirm the `claimId` is not present.
7. For Level 3: fetch the accreditation reference from `authority.accreditation.lookupUri` (or `body`) and confirm the accreditation is current.
8. Resolve `predicate.vocabulary` and `predicate.vocabularyVersion` to determine the binding meaning of `predicate.claim`.
9. Decide, based on the consumer's policy, whether to accept the claim.

The verification pipeline is implementation-defined in detail; the steps above are the typical shape.

### 8.4 Revocation policy

A practical revocation policy:

- The revocation list is published at a stable URI and refreshed at a known cadence.
- Each revocation entry carries the `claimId`, the `revokedAt` timestamp, and a `reason` field with a vocabulary-defined value.
- The revocation list is signed by the same authority as the claims it revokes.
- When an artefact is updated, prior claims about the older bytes are not automatically revoked; they remain valid for the bytes they were issued against. New claims are issued against the new bytes. Consumers verifying step 2 of §8.3 will detect that an old claim does not apply to new bytes without needing the revocation list.

---

## 9. Security and privacy considerations

This section is normative.

### 9.1 Signature compromise

If an authority's signing key is compromised, all claims signed by that key are suspect. The authority MUST revoke all outstanding claims and MUST publish a key compromise notice at the same URI as the revocation list. Consumers who have cached claims from the compromised authority SHOULD re-evaluate them on next use.

The standard does not require key rotation on a fixed schedule; it requires that the authority be operationally able to revoke the entire claim set on key compromise.

### 9.2 Subject substitution

A consumer who fails to recompute and compare the subject hash (step 2 of §8.3) accepts the risk that the bytes it has are not the bytes the claim was issued against. This failure mode is the most likely source of false-positive verifications and SHOULD be hard-coded into any compliant verification implementation.

### 9.3 Evidence retrieval

The `evidence.report` URI may be private (auditor-held, regulator-held) or public. A consumer who lacks authorisation to retrieve the evidence MAY still verify the claim's signature and binding to the subject; it cannot independently inspect the basis. This is an intentional two-tier model: signature verification is universal, evidence inspection is conditional.

A consumer that requires evidence inspection (a regulator, an internal compliance team) MUST establish access to the evidence URI through a means out of scope here.

### 9.4 Privacy

A claim necessarily exposes:

- The fact that the subject artefact was claimed against a specific predicate.
- The identity of the authority.
- The hash of the subject at issuance time.

A claim does not necessarily expose:

- The content of the evidence report (governed by `evidence.report` access control).
- The identity of human reviewers within the authority (governed by what the authority chooses to publish).

Where claim publication itself raises privacy concerns (a claim about a private dataset's GDPR conformance, for example), the issuer MAY publish the claim only to authorised consumers via access-controlled distribution, accepting that the claim is then not a public attestation. The standard accommodates both modes.

### 9.5 Replay and reuse

A claim is bound to a specific subject hash and has a specific `claimId` and `issuedAt`. It cannot be replayed against a different subject without breaking the signature. It can, however, be presented to multiple consumers; this is intentional. Consumers MUST NOT treat a claim as freshly issued to them; they MUST evaluate `issuedAt`, `expiresAt`, and revocation against current time.

### 9.6 Vocabulary substitution

A predicate vocabulary's URI is referenced but not embedded. A vocabulary publisher who modifies the meaning of an existing `claim` identifier without bumping `vocabularyVersion` undermines all outstanding claims against that identifier. Vocabulary publishers MUST treat existing identifiers as immutable within a version and MUST bump `vocabularyVersion` on any change to identifier meaning.

Consumers SHOULD pin to specific `vocabularyVersion` values for high-stakes evaluations and re-evaluate on vocabulary upgrade.

---

## 10. IANA considerations

This note proposes no new IANA registrations.

A future revision MAY propose:

- A `mx-compliance-claim` link relation type for in-band claim references.
- A `application/mx-compliance-claim+json` media type for serialised claims.
- A `/.well-known/mx/claims/` URI suffix.

These are deferred to a later revision once implementation experience justifies the registration.

---

## 11. References

### 11.1 Normative

- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) — Key words for use in RFCs.
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) — Ambiguity of uppercase vs lowercase in RFC 2119 key words.
- [RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) — JSON Web Signature (JWS).
- [RFC 8032](https://datatracker.ietf.org/doc/html/rfc8032) — Edwards-Curve Digital Signature Algorithm (EdDSA).
- [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615) — Well-Known Uniform Resource Identifiers.
- [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) — Date and time format.

### 11.2 Informative

- [W3C Decentralized Identifiers (DID) v1.0](https://www.w3.org/TR/did-core/) — Identifier syntax for decentralised authorities.
- [W3C Verifiable Credentials Data Model v2.0](https://www.w3.org/TR/vc-data-model-2.0/) — General-purpose credential model that compliance claims MAY be expressed under.
- [C2PA Content Credentials](https://c2pa.org/) — Content provenance manifests; relevant for claims whose predicate concerns content origin.
- [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/) — Web Content Accessibility Guidelines, used as the worked predicate vocabulary in §6.2.
- MX Field Definition Pattern note — Field-definition discipline this note adopts.
- MX Contract Fingerprinting and Signing note — Canonicalisation and signature mechanics this note defers to.
- MX Provenance note — Content origin and stewardship vocabulary.
- MX Carrier Formats note — Carrier-format mappings for claim references in HTML, PDF, and other formats.

---

## 12. Acknowledgements

This note builds on prior work in C2PA Content Credentials, W3C Verifiable Credentials, and the broader content-attestation community. It is informed by the practical experience of issuing and verifying conformance claims in regulated content programmes over many years, and by the gap-analysis exercise that motivated its drafting in early 2026.
