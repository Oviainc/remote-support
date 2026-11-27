# Architecture — Remote Support

## Overview

Three kinds of component, one shared core:

```
  Host agent ──┐                        ┌── Viewer (desktop / mobile)
               │   rendezvous, then     │
               ├──── direct path ───────┤
               │   (NAT traversal)      │
               └── relay fallback ──────┘
                        │
              Rendezvous + relay server
              (single self-hosted binary:
               registry, broker, relay,
               audit log, update service)
```

## Shared core (Rust)

One Rust codebase implements everything platform-independent, so behaviour
and security properties are identical everywhere:

- **Capture pipeline interface** — frames arrive from the platform layer in
  a common format with timestamps and damage regions.
- **Encoder** — adaptive video encoding: resolution and bitrate scale to
  measured bandwidth; keyframes on demand; quality yields before frame
  rate, frame rate before input latency. (Implements SPEC FR-C4.)
- **Input model** — a single normalised event stream (pointer, key, text,
  scroll, touch) that each platform layer translates to native input.
- **Encryption** — session keys negotiated per session between host and
  viewer; the relay forwards ciphertext only (see `SECURITY.md`).
- **Connection manager** — registration with the rendezvous server,
  direct-path negotiation, relay fallback, heartbeat, and automatic
  reconnect/resume within the grace window (SPEC FR-C2, FR-C3).
- **Session state machine** — requested → approved → live → ended, with
  the consent gates of SPEC §6 enforced in the core, not in UI code, so no
  client can skip them.

## Platform layers (thin)

Each platform contributes only what the operating system forces to be
different:

| Platform | Capture | Input | Notes |
| --- | --- | --- | --- |
| Windows | Desktop duplication API | SendInput | Service + tray agent |
| macOS | ScreenCaptureKit | CGEvent | Permission detection and guided grant flow (FR-D4) |
| Linux | PipeWire / X11 capture | XTest / uinput (Wayland) | Both X11 and Wayland paths |
| Android | MediaProjection | Accessibility gestures + key events | Foreground service; battery-exemption setup (FR-A2) |
| iOS | See iOS modules below | See below | Tiered by platform limits |
| Web | n/a (viewer) | Browser events | Viewer only; see web viewer section |

The rule: if logic can live in the core, it lives in the core. Platform
layers capture, inject, and report permissions — nothing else.

## Rendezvous and relay server

A single self-contained Rust binary, run by the owner on their own
hardware:

- **Registry** — enrolled devices with platform, tier, identity key
  fingerprint, and online state. Backed by an embedded store; no external
  database.
- **Broker** — receives session requests, checks them against the consent
  rules and fleet policy, runs the approval flow for attended sessions,
  and introduces host and viewer.
- **NAT traversal** — coordinates hole-punching so host and viewer connect
  directly where the network permits.
- **Relay** — when no direct path exists, forwards encrypted session
  traffic between the two ends. The relay never holds session keys and
  cannot read session content.
- **Audit log** — append-only record of every session (SPEC FR-S6).
- **Update service** — serves signed agent/viewer updates to the fleet
  (SPEC §7).

The server needs only one reachable port for clients; all client
connections are outbound from the clients' side.

## Identity and encryption model

- Every device generates a long-term identity key pair at enrolment; the
  public key is registered with the owner's server, the private key never
  leaves the device.
- Supporters hold identity keys the same way; a session prompt names the
  supporter by their registered identity, not by a self-asserted string.
- Session keys are ephemeral, negotiated per session between host and
  viewer (end-to-end). Compromise of the relay yields ciphertext and
  metadata, never content.
- Pairing codes and QR codes are single-use, short-lived, and bind a new
  device's identity key to the fleet registry.

## iOS modules

Three modules implement the tiers of SPEC §3.3:

1. **View and guide (ReplayKit).** The app hosts a broadcast upload
   extension; the owner starts the broadcast, frames flow to the core
   pipeline as a view-only stream, and the viewer's pointer/annotations
   are rendered as an overlay on the supporter's side and mirrored as
   guidance cues. No input path to the device exists in this module.
2. **Managed control (WebDriverAgent).** For enrolled devices, a control
   driver speaks to the on-device automation runner over the local network
   or through the owner's relay: taps, swipes, text entry, app launch, and
   accessibility-tree reads for precise targeting. Provisioning (Mac,
   development tools, signing, developer mode) is an enrolment-time
   tool, packaged separately from the support app, so the support flow
   itself stays one tap.
3. **BLE HID bridge.** A small hardware companion: Bluetooth Low Energy
   HID keyboard + pointer profiles toward the phone, screen frames in via
   mirroring receiver, control channel back to the viewer through the
   rendezvous server. Includes per-device pointer calibration. The bridge
   is a peripheral, not a computer — it holds no session content beyond
   the live stream it forwards, encrypted end to end.

## Web viewer

- Served as static assets by the owner's server; no third-party CDN or
  external asset loads (SPEC FR-W3).
- The browser client runs the same session state machine and encryption as
  native viewers (core compiled for the web target), so consent and
  security behaviour cannot diverge between native and web.
- Canvas-rendered video surface with the normalised input model captured
  from browser events.

## Failure handling (by design, not by hope)

- Direct path fails → relay fallback, automatically, mid-session if
  needed.
- Connection drops → heartbeat detects, both ends attempt resume inside
  the grace window; the session indicator persists through reconnects.
- Missing OS permission → detected at agent start and at session start;
  the session cannot begin until capture/input are actually possible, and
  the owner is shown the exact setting to change.
- Bandwidth collapses → encoder sheds quality, then frame rate; input
  channel is prioritised so control never queues behind video.
