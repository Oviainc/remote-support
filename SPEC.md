# Specification — Remote Support

This document is the requirement set for the product described in
`README.md`. Wording is normative: **must** is a requirement, **should** is a
strong default that a phase may defer only by recording the deferral in
`ROADMAP.md`.

## 1. Purpose and scope

A remote support system that runs entirely on infrastructure owned by the
operator. The system provides attended support (a person is present at the
device being helped) and managed support for enrolled fleets. It provides no
component hosted by, dependent on, or reporting to any third-party service.

Out of scope: silent or covert monitoring of any kind; access to a device
without the knowledge of the person at it (see §6); consumer file sharing;
general-purpose remote administration unrelated to support sessions.

## 2. Components

1. **Host agent** — installed on each supported device; shares the screen
   and accepts input during an approved session.
2. **Viewer** — the supporter's application (desktop, mobile, or web).
3. **Rendezvous and relay server** — one self-hosted binary. It introduces
   hosts and viewers, brokers direct connections, relays traffic when a
   direct path is impossible, holds the fleet register, and serves updates.

## 3. Functional requirements by platform

### 3.1 Windows, macOS, Linux (desktop hosts)

- FR-D1. The host agent must capture the full desktop, including multi-monitor
  layouts, and let the supporter switch or span monitors.
- FR-D2. The host must accept keyboard and pointer input from the viewer
  with latency low enough for interactive work on a normal connection.
- FR-D3. The host must support attended sessions (per-session approval, §6)
  and, for enrolled fleet devices, managed sessions under the fleet's stated
  policy — with the session indicator visible in both cases (§6).
- FR-D4. The agent must detect missing operating-system permissions (screen
  recording, accessibility/input control, and equivalents) and guide the
  device owner to the exact setting, in plain language.
- FR-D5. Clipboard transfer and file transfer during a session should be
  supported, each visible to the person at the host and recorded in the
  session log.

### 3.2 Android

- FR-A1. The host app must provide full control (screen share plus touch
  and typing input) using the platform's accessibility and screen-capture
  facilities, with the owner's consent captured at setup and again at
  session start for attended support.
- FR-A2. The host app must run as a foreground service with the standard
  battery exemptions so vendor power management does not kill it between
  sessions; setup must walk the owner through the exemption on vendors
  known to be aggressive.
- FR-A3. The Android app must also function as a viewer.

### 3.3 iPhone / iPad (iOS)

iOS offers no general remote-control facility to third-party software. The
product therefore specifies three tiers, and the tier of every session must
be shown in plain words (**View only** / **Control available**).

- FR-I1. **View and guide (any device).** The owner starts screen broadcast
  from within the app (system broadcast facility). The supporter sees the
  screen in real time and can overlay a pointer and annotations. No input
  is sent to the device in this tier.
- FR-I2. **Managed control (enrolled devices).** Devices enrolled in
  advance may be controlled through the platform's UI-automation route
  (WebDriverAgent driven over the local network or the owner's relay):
  real taps, swipes, typing, app launch, and screen reading.
  - Enrolment is a one-time provisioning step requiring a Mac with the
    platform development tools, developer mode enabled on the device, and
    the automation runner signed to the device by the fleet operator.
  - Enrolment must never be a step in an attended support flow. A device is
    either enrolled beforehand or it is a view-and-guide device.
  - Enrolled devices must still show the session indicator and follow §6.
- FR-I3. **Bridge control (stock devices, proximity).** A device paired
  with the system's Bluetooth HID bridge may be controlled as if by a
  physical keyboard and pointer: screen arrives by mirroring to the
  bridge's receiver, input returns over Bluetooth Low Energy using
  standard HID profiles, with pointer calibration per device.
  - This tier requires the bridge to be within Bluetooth range of the
    phone. It requires no modification, no developer tooling, and no
    installed control software on the phone beyond enabling the pointer
    accessibility feature.
  - Gesture fidelity is pointer-grade: taps, drags and typing are
    supported; multi-touch gestures beyond what a pointer and keyboard can
    express are not.
- FR-I4. The iOS app must function as a viewer for sessions hosted on any
  other platform.

### 3.4 Web viewer / console

- FR-W1. A viewer must run in a current web browser with no installation,
  able to view and control desktop and Android hosts and to view iOS
  broadcasts.
- FR-W2. The console must list the fleet register (device, platform, tier,
  online state) and start sessions from it, subject to §6 consent rules.
- FR-W3. The web viewer is served by the owner's own server. It must not
  load code or assets from third-party CDNs.

## 4. Connection requirements

- FR-C1. Hosts and viewers must require only outbound connections to the
  owner's rendezvous/relay server.
- FR-C2. The system should establish a direct host↔viewer path where the
  network allows (NAT traversal) and must fall back to relaying through the
  owner's server when it does not. The fallback must be automatic and must
  not end the session.
- FR-C3. Sessions must reconnect and resume automatically after a network
  interruption, without new approval, for a bounded grace period.
- FR-C4. The video pipeline must adapt quality to available bandwidth,
  preferring input responsiveness over image fidelity.

## 5. Deployment requirements

- FR-P1. One installer, or a single portable binary, per desktop platform.
  Silent-install flags must be provided for fleet rollout (MDM / group
  policy / configuration management).
- FR-P2. The rendezvous and relay server must be a single self-contained
  binary with no external database or service dependencies.
- FR-P3. Device pairing must be completable with a short code or a QR code,
  once per device, producing an entry in the fleet register.
- FR-P4. Install-to-connected (agent installed, server reachable, device
  paired, first session possible) must be under five minutes on a normal
  network for a single device.
- FR-P5. A fleet operator must be able to enrol devices in bulk with a
  configuration file or enrolment link carrying the server address and
  fleet policy — no per-device manual server entry.

## 6. Session and consent rules

These rules are absolute and apply to every platform and tier.

- FR-S1. **No silent access.** There is no mode in which a session is
  invisible to the person at the device.
- FR-S2. **Visible session indicator.** From session start to session end,
  the host device shows a persistent, unmissable indicator that a support
  session is live and who is connected.
- FR-S3. **Per-session approval for attended support.** On a device that is
  not enrolled under a managed policy, every session starts only after the
  person at the device approves a prompt naming the supporter.
- FR-S4. **Managed sessions are policy-bound, not silent.** Enrolled fleet
  devices may start sessions under their fleet's written policy, but FR-S1
  and FR-S2 still apply, and the device owner must have been informed of
  the policy at enrolment.
- FR-S5. **The person at the device can end the session at any time**, with
  one obvious action, effective immediately.
- FR-S6. **Audit log.** Every session is written to an audit log held on
  the owner's server: device, supporter identity, tier, start and end
  times, approvals given, file transfers, and whether the session was
  recorded. Logs are append-only from the clients' side.
- FR-S7. **Recording is opt-in and announced.** A session may be recorded
  only when recording was enabled for that session, and the indicator
  (FR-S2) must show recording state distinctly.

## 7. Updates

- FR-U1. Agents and viewers update from the owner's own server only. No
  component contacts any third-party update service.
- FR-U2. Updates must be signed; clients verify signatures before applying.
- FR-U3. Fleet operators can stage rollouts (pilot group before fleet) and
  defer updates per group. A failed update must roll back automatically.

## 8. Non-functional requirements

- NFR-1. End-to-end encryption of all session traffic (see `SECURITY.md`).
- NFR-2. No telemetry, analytics, or usage reporting to any third party.
  Diagnostics stay on the owner's infrastructure.
- NFR-3. The relay server must run comfortably on modest hardware (a small
  VPS or spare office machine) for a small fleet.
- NFR-4. All user-facing text is plain language; states are named, not
  coded ("View only", "Control available", "Waiting for approval").
