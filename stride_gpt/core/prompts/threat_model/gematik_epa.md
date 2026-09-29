---
name: gematik_epa
title: gematik TI / ePA Domain Card
when_to_load: |
  Load when the subsystem is part of the German Telematikinfrastruktur (TI) or
  the electronic patient record (elektronische Patientenakte, ePA). Signals:
  ePA, ePA-FdV, Aktensystem, VAU, Vertrauenswürdige Ausführungsumgebung,
  Access Gateway, Kostenträger, KVNR, GesundheitsID, sektoraler IdP,
  Befugnis, Einwilligung, Widerspruch, Vertretung, Leistungserbringer, LEI,
  eGK, gematik, epa4all, VZD, FHIR-Directory, MIO, eRP, E-Rezept, DiGA,
  Signaturdienst, Protokolldaten, Gerätebindung, MHD, XDS.
adds_fields:
  - TI_DOMAIN_AREA
stride_letters: [S, T, R, I, D, E]
source: https://gemspec.gematik.de/ (gemSpec_ePA_FdV, gemSpec_Krypt, gemSpec_IDP_Sek, gemSpec_DS_Hersteller)
version: 2026-09
---

# gematik TI / ePA — Domain Reference Card

You loaded this card because the subsystem belongs to the German
Telematikinfrastruktur, most likely the electronic patient record (ePA) and its
insurant-facing frontend (Frontend des Versicherten, **FdV**).

This card supplies domain structure, not a vulnerability catalogue. Use it to
know what the components are for, where the sensitive transitions sit, and what
vocabulary to use so that findings are legible to specialists in this domain.

When you emit a threat, set `TI_DOMAIN_AREA` to the area it belongs to from the
list under "Domain areas" below, or omit the field if none fits.

Use German terms as the specifications use them, with a short English gloss on
first use in a threat. Write *Befugnis (entitlement)*, not *permission object*.

---

## What makes this domain distinctive

**The record operator is not trusted with the contents.** This is the central
design decision and the source of the most interesting threats. The ePA is
operated by a provider on behalf of a Kostenträger (statutory health insurer),
but that operator must not be able to read insurant medical data. Confidentiality
rests on the **VAU** (Vertrauenswürdige Ausführungsumgebung — trusted execution
environment) and on hardware-backed key storage, not on the operator's good
behaviour.

Practical consequence for analysis: any insurant data, or any metadata permitting
inference about insurant data, that becomes readable *outside* the VAU is a
finding, even when no external attacker is involved. Traffic patterns, timing,
request sizes, identifiers in headers, error messages, and operational logs all
count as metadata.

**Authorisation decisions are made by the patient and are legally binding.**
Einwilligung (consent), Widerspruch (objection/opt-out), Befugnis (entitlement
granted to a service provider), and Vertretung (representation by another person)
are not configuration settings. They determine lawful access to medical data. A
forged, replayed, silently expired, or misdisplayed authorisation is a severe
integrity finding — often more severe than an eavesdropping finding on the same
interface.

**Identity is federated and sector-specific.** The insured person authenticates
against a sectoral IdP (sektoraler IdP) operated by their own Kostenträger, using
the **GesundheitsID**. The record system trusts the tokens that IdP issues. The
insurant is identified by **KVNR** (health insurance number), which is stable,
long-lived and directly identifying — the linkage of KVNR to record contents is
the core privacy harm in this system.

**Access is bound to registered devices.** Device registration
(Geräteregistrierung/Gerätebindung) establishes a device token that acts as a
second factor. Registration, re-registration, listing and deregistration of
devices are therefore security-relevant operations, not convenience features.

---

## Domain areas

Use one of these as `TI_DOMAIN_AREA`.

- `authentication` — GesundheitsID, sectoral IdP federation, token issuance and
  validation, eGK + PIN on desktop, local authentication at the FdV.
- `device_management` — registration, attestation, device token lifecycle,
  listing, deregistration, notification of new devices.
- `vau_channel` — establishment and operation of the VAU channel, user session,
  Akten-ID handling, session timeout, what is and is not visible outside it.
- `authorization` — Befugnisse for Leistungserbringerinstitutionen (LEI) and
  DiGA, EU cross-border access, Vertretung, display and revocation of
  authorisations.
- `consent` — Einwilligung, Widerspruch, secondary-use purposes
  (Sekundärnutzungszwecke), hiding of documents and categories (Verbergen).
- `documents` — upload, search, download, deletion, metadata modification,
  XDS/MHD metadata, document formats, collections.
- `medication` — medication list retrieval and related specialist services.
- `audit_log` — Protokolldaten: recording, retrieval, export, retention.
- `notifications` — push notifications via Push Gateway and platform providers,
  e-mail notifications, notification content.
- `directory` — VZD-FHIR-Directory lookups of service providers.
- `signature` — the Signaturdienst and signature-requiring operations.
- `privacy_telemetry` — tracking, UX measurement data, consent for tracking,
  National Health Portal (NGP) integration.

---

## Where to concentrate

**1. The VAU boundary.** For every data flow, determine what is visible at the
Access Gateway versus inside the VAU. Look for: identifiers in transport
metadata, request/response sizes that reveal document properties, timing that
reveals user behaviour, error and status codes that leak record state, logging
and monitoring that crosses the boundary, key material reachable outside the HSM.

**2. Authorisation integrity, not just confidentiality.** For Befugnis,
Einwilligung, Widerspruch and Vertretung ask: who can create one, who can modify
one, is it cryptographically bound to the insurant who granted it, can it be
replayed or reinstated after revocation, does revocation take effect immediately
and retroactively, and does what the user sees in the app match what the backend
enforces. A UI that misrepresents who currently holds access is a genuine
finding.

**3. The representation relationship (Vertretung).** A representative acts on
another person's record. Establishment, scope, duration, revocation, and the
representative's own device binding are each attack surfaces. Vertretung spans
several specification chapters and is easy to analyse incompletely.

**4. Device lifecycle edges.** What happens on: registration of a second device,
registration from a device the insurant does not control, deregistration while a
session is live, loss of the only registered device, restore of an OS backup to
new hardware, and reinstallation of the app. Each is a transition where access
can be gained or irrecoverably lost.

**5. Token handling across the federation.** Where ID_TOKEN and access tokens
live, how long they last, what binds them to a device and to a session, what the
record system validates before trusting them, and what an attacker holding one
can do.

**6. Data that leaves the TI trust domain.** Push notifications traverse Apple
and Google infrastructure. Notification content, and the mere fact of a
notification, can disclose treatment information to parties outside the TI and to
anyone reading a lock screen. Directory lookups and NGP content are further
egress points.

**7. The audit log as an asset in its own right.** Protokolldaten record who
accessed the record and when. That is both the accountability mechanism and,
itself, sensitive medical-adjacent data. Analyse its confidentiality, its
integrity (can an access be omitted or forged), and its availability to the
insurant.

**8. Irreversibility.** Medical data disclosed cannot be undisclosed, and a KVNR
cannot be rotated like a password. Let this weight your impact statements: in
this domain a confidentiality breach of record content is generally more severe
than a comparable breach in a commercial system.

---

## Regulatory framing

Findings land better when they name the consequence in the domain's own terms.
Beyond technical impact, consider: unlawful processing of health data under GDPR
Article 9 (besondere Kategorien personenbezogener Daten), breach of the
specification requirements the product is approved against (Zulassung), loss of
the confidentiality guarantee the VAU design exists to provide, and erosion of
the insurant's legal control over who may see their record.

Do not report "the specification is not implemented" as a threat on its own —
conformance is assumed. Report threats that exist *despite* correct
implementation, and threats arising where the specification leaves a decision to
the implementer.
