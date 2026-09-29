---
name: owasp_mobile
title: OWASP Mobile Top 10 & MASVS
when_to_load: |
  Load when the subsystem is, or talks to, a mobile or desktop client
  application installed on an end user's own device — Android, iOS, or a
  packaged desktop app. Signals: app, client, frontend, GUI, local storage,
  keystore, keychain, biometrics, device binding, attestation, push
  notification, deep link, jailbreak/root, sideloading, screen lock, clipboard,
  screenshot, backup, IPC. Also load for German-language corpora using terms
  such as Frontend des Versicherten, FdV, Endgerät, Gerätebindung,
  Schlüsselspeicher, Bildschirmsperre.
adds_fields:
  - MASVS_CONTROL
  - OWASP_MOBILE
stride_letters: [S, T, R, I, D, E]
source: https://owasp.org/www-project-mobile-top-10/ and https://mas.owasp.org/MASVS/
version: Mobile Top 10 2024 / MASVS v2.1
---

# OWASP Mobile Top 10 & MASVS — Reference Card

You loaded this card because the subsystem runs on, or exchanges data with, an
application installed on an end user's own device. The device is not
infrastructure the operator controls: it may be lost, shared, observed, rooted,
or running hostile software.

For each threat you emit from this subsystem, attach:

- `owasp_mobile` — the most specific applicable risk ID from the Top 10 below
  (e.g. `M9`), or omit if none fits.
- `masvs_control` — the applicable MASVS control group (e.g. `MASVS-STORAGE`),
  or omit if none fits.

Both are arguments to `report_threat`. Mobile threats are also where the
`implemented_mitigations` / `recommended_mitigations` split matters most: platform
protections are frequently documented as present, and a finding is only useful if
it says which of them already apply.

> **Anti-hallucination requirement.** Use only the IDs and control-group names
> exactly as written in this card. Do not invent identifiers, do not paraphrase
> group names, do not cite MASVS v1 groups (MSTG-*) — this card is v2.1.

---

## OWASP Mobile Top 10 (2024)

**M1 — Improper Credential Usage.** Hardcoded credentials in the app package;
credentials transmitted or stored insecurely; credentials reused across users or
installs; shared secrets recoverable from a distributed binary. STRIDE: Spoofing,
Elevation of Privilege.

**M2 — Inadequate Supply Chain Security.** Malicious or vulnerable code
introduced through SDKs, libraries, build pipelines or distribution. Tampered
builds, unofficial distribution channels, sideloaded copies. STRIDE: Tampering,
Elevation of Privilege.

**M3 — Insecure Authentication/Authorization.** Authentication performed or
decided client-side; missing re-authentication for sensitive operations;
authorization checks the client can skip; weak session or token lifetime; step-up
authentication absent where the operation warrants it. STRIDE: Spoofing,
Elevation of Privilege.

**M4 — Insufficient Input/Output Validation.** Untrusted data from servers, other
apps, deep links, QR codes, files or IPC processed without validation; injection
into local queries, WebViews or rendering; output encoding failures. STRIDE:
Tampering, Elevation of Privilege.

**M5 — Insecure Communication.** Missing or weak TLS; certificate validation
disabled or misconfigured; cleartext fallback; sensitive data in URLs or headers
visible to intermediaries; metadata exposure on the wire. STRIDE: Information
Disclosure, Tampering, Spoofing.

**M6 — Inadequate Privacy Controls.** Collection or transmission of personal data
beyond what the user consented to; identifiers that permit tracking or
correlation; telemetry containing sensitive content; data shared with third
parties without a legal basis; consent that is unclear, pre-checked or
irrevocable. STRIDE: Information Disclosure.

**M7 — Insufficient Binary Protections.** Reverse engineering of the distributed
package; extraction of embedded secrets, endpoints or business logic; patched
builds that remove client-side checks. STRIDE: Tampering, Information Disclosure.

**M8 — Security Misconfiguration.** Debug features or verbose logging left
enabled; permissive platform settings (exported components, allowBackup, arbitrary
loads); over-broad permissions; default or weak configuration shipped to
production. STRIDE: Information Disclosure, Elevation of Privilege.

**M9 — Insecure Data Storage.** Sensitive data written to unprotected local
storage, databases, caches, logs, screenshots, clipboards or platform backups;
data readable by other apps or by anyone with the unlocked device; residue not
cleared on logout or deregistration. STRIDE: Information Disclosure.

**M10 — Insufficient Cryptography.** Weak or obsolete algorithms; keys derived
from predictable material; keys stored outside hardware-backed storage; IV/nonce
reuse; homegrown cryptography; missing integrity protection alongside encryption.
STRIDE: Information Disclosure, Tampering.

---

## MASVS v2.1 control groups

Use these to name *what protection is missing*, where the Top 10 names *what goes
wrong*. A threat commonly carries one of each.

**MASVS-STORAGE** — Secure storage of sensitive data on the device, and the
absence of unintended leakage into logs, caches, backups, screenshots, the
clipboard, and IPC-reachable locations.

**MASVS-CRYPTO** — Correct use of cryptography: current algorithms and modes,
proper key generation, storage in hardware-backed keystores, sound key lifecycle
including rotation and destruction.

**MASVS-AUTH** — Authentication and authorization performed and enforced where it
can be trusted; biometric and local authentication bound to a cryptographic
operation rather than a boolean result; sessions and tokens with appropriate
lifetime, binding and revocation.

**MASVS-NETWORK** — Secure network communication: current TLS, correct
certificate and hostname validation, no cleartext fallback, and protection of
data in transit against an attacker positioned on the network path.

**MASVS-PLATFORM** — Safe interaction with the platform and other apps: IPC and
exported components, deep links and custom URL schemes, WebView configuration,
notifications and lock-screen exposure, screen capture and overlay protection,
clipboard handling, permission use.

**MASVS-CODE** — Sound handling of data and dependencies within the app: input
validation at every entry point, current libraries, no dead or debug code paths
in release builds, memory-safety-sensitive handling of parsed input.

**MASVS-RESILIENCE** — Resistance to reverse engineering and runtime tampering on
a hostile device: integrity verification, anti-tampering, obfuscation, detection
of rooted/jailbroken or emulated environments, defence in depth for client-side
checks.

**MASVS-PRIVACY** — Minimisation, transparency and user control over personal
data: collecting only what is needed, disclosing what is collected, honouring
consent and withdrawal, avoiding persistent identifiers that enable tracking.

---

## Where mobile threats concentrate

Prioritise analysis at these points. They produce the highest yield per unit of
effort in mobile threat models.

1. **The unlocked-device scenario.** Someone other than the owner has the device
   in hand, unlocked. What can they read, send, authorise, or change? Which
   operations demand re-authentication and which do not?
2. **Local persistence.** Every place the app writes: databases, preference
   stores, caches, temporary files, logs, crash reports, platform backups. What
   lands there, for how long, and what removes it.
3. **Key material lifecycle.** Generation, storage location, access-control flags,
   what unlocks a key, what happens on device deregistration, on app reinstall,
   on OS restore to a new device.
4. **The trust transition at the network edge.** What the app believes about the
   server, what the server believes about the app, and what an observer on the
   path learns from metadata even when the payload is protected.
5. **Data arriving from outside the app.** Deep links, push payloads, IPC, files
   opened from elsewhere, QR codes, responses from directories and third-party
   services. Each is an untrusted input crossing into the app's trust zone.
6. **Platform surfaces that bypass the app's own UI.** Notification previews on
   the lock screen, screenshots in the task switcher, accessibility services,
   overlays, clipboard, autofill.
7. **Consent and privacy mechanics.** What is collected before consent, what
   consent text actually covers, whether withdrawal takes effect retroactively,
   and which identifiers persist across sessions and installs.

## Framing guidance

The device belongs to the user, not to the operator. Client-side controls —
attestation, root detection, integrity checks, obfuscation — raise attacker cost;
they do not establish trust. When a threat's only barrier is a client-side check,
say so explicitly in the threat's impact: the control is defence in depth, and
anything that must be *guaranteed* has to be enforced server-side.

Conversely, when platform protections are documented as present and in use
(hardware keystore, screen-lock requirement, attestation), do not report their
absence as a threat. Report instead the consequences of their bypass, the data
that falls outside their coverage, and how correctly the application uses them.
