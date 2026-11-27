# Security — Remote Support

Remote support software is, by construction, software that lets one person
see and drive another person's machine. That makes its security model the
product. This document states the threat model and the non-negotiables.
Where this document and any other document disagree, this document wins.

## Threat model

Assets: the screens, files, credentials, and input of every supported
device; the integrity of the fleet registry; the audit log.

Adversaries considered:

- **A network attacker** between host, viewer, and server — assumed able
  to observe, delay, and modify traffic.
- **A compromised relay server** — the owner's server is trusted to broker
  and log, but the design must limit what its compromise exposes.
- **A rogue or careless supporter** — an authorised user attempting access
  beyond consent, outside policy, or without a record.
- **A malicious "supporter" from outside** — the classic tech-support
  scam: a stranger social-engineering a device owner into starting a
  session. The consent UX is a security control against exactly this.
- **A lost or stolen enrolled device** — its identity key must be
  revocable without disturbing the rest of the fleet.
- **A supply-chain attacker** — tampered installers or updates.

Out of scope: an attacker with sustained physical possession of an
unlocked host device, or full compromise of the host operating system
itself. No remote support tool can defend the session layer against those,
and this one does not claim to.

## Non-negotiables

1. **End-to-end encryption.** All session content (video, input, files,
   clipboard) is encrypted between host and viewer with ephemeral
   per-session keys. The relay forwards ciphertext only and cannot decrypt
   session content.
2. **Device identity keys.** Every device and every supporter holds a
   long-term identity key pair, generated on the device, private key never
   transmitted. Sessions authenticate both ends by key, not by password or
   by a self-asserted name. Keys are individually revocable at the server.
3. **Consent before control.** No session without the approval flow of
   SPEC §6: per-session approval on attended devices; written fleet policy
   plus enrolment-time notice on managed devices. There is no silent
   access mode, no hidden session, and no way to configure either.
4. **Visible session indicator.** A persistent, unmissable on-screen
   indicator for the full duration of every session, showing that a
   session is live, who is connected, and whether recording is on. The
   person at the device can end the session with one obvious action at
   any time.
5. **Recording is announced and opt-in.** Recording off by default;
   enabling it is a per-session choice, shown distinctly in the indicator,
   and noted in the audit log.
6. **Audit log of every session.** Device, supporter identity, tier,
   approvals, start/end, transfers, recording state — appended to the
   log on the owner's server for every session without exception,
   including failed and refused session attempts.
7. **No telemetry to any vendor.** The product sends nothing — no usage
   data, no crash reports, no analytics — to any third party. Diagnostics
   exist only on the owner's infrastructure, under the owner's control.
8. **Signed everything.** Installers and updates are signed; clients
   verify signatures before installing or applying. Updates come only
   from the owner's own server.
9. **Least privilege by tier.** View-only tiers have no input path in
   code, not merely a disabled one. The iOS view-and-guide module cannot
   send input because the capability is absent, not switched off.
10. **Pairing is single-use and short-lived.** Pairing codes and QR codes
    expire quickly, work once, and bind a device identity key to the
    registry; a photographed stale code is worthless.

## Design consequences

- The relay's compromise exposes session metadata and ciphertext, never
  content — a deliberate consequence of end-to-end keys held only by the
  endpoints.
- The consent prompt names the supporter by registered identity and shows
  the device's tier, because the prompt is the last line of defence
  against support-scam social engineering.
- Audit logging is in the session path, not an optional add-on: a session
  that cannot be logged does not start.

## Before any external licensing

Internal use does not lower the bar, but external licensing raises the
stakes: other organisations' devices and data would depend on these
claims. Therefore, before this product is licensed, supplied, or deployed
outside the owner's own organisation:

- an independent security review of the protocol, the encryption and
  identity implementation, and the consent flow must be completed;
- findings must be fixed or formally accepted in writing by the owner;
- the review must be repeated after any change to the session protocol,
  the key handling, or the consent model.

No external deployment on the strength of internal testing alone.
