# Roadmap — Remote Support

Phases are sequential by dependency, not by calendar. A phase is done when
its done-definition is met in full — including the consent, indicator, and
audit requirements of `SPEC.md` §6 and `SECURITY.md`, which apply from
Phase 1 and are never deferred.

## Phase 1 — Desktop hosts, viewer, and relay

Scope: Rust core; Windows, macOS, and Linux host agents; desktop viewer;
rendezvous + relay server (registry, broker, NAT traversal, relay
fallback, audit log).

**Done when:**
- A session runs end to end on each desktop platform: request, approval,
  visible indicator, control, one-action end, audit entry.
- A session survives a blocked direct path by falling back to the relay,
  and survives a forced disconnect by resuming inside the grace window.
- Install-to-connected for a single device measures under five minutes
  (SPEC FR-P4) on a clean machine.
- Missing-permission guidance (FR-D4) is demonstrated on macOS.

## Phase 2 — Android host

Scope: Android host app (full control) and Android viewer, against the
Phase 1 server.

**Done when:**
- Full-control sessions run on devices from at least three major Android
  vendors, including one known for aggressive battery management, with the
  agent still reachable after 24 hours idle.
- The battery-exemption and accessibility setup flow completes without
  developer assistance on each test vendor.
- Consent, indicator, and audit behaviour match desktop exactly.

## Phase 3 — Web viewer

Scope: browser viewer and fleet console, served by the owner's server.

**Done when:**
- A supporter can run a full-control session to desktop and Android hosts
  from a current browser with nothing installed, including on a
  Chromebook.
- The console lists the fleet register with platform, tier, and online
  state, and starts sessions under the same consent rules as native
  viewers.
- The served pages load zero third-party assets (verified by inspection).

## Phase 4 — iOS view and guide

Scope: iOS app as viewer; broadcast-based view-and-guide hosting
(ReplayKit) with supporter pointer and annotation overlay.

**Done when:**
- A one-tap broadcast from a stock iPhone produces a live view session;
  the tier reads "View only" on both ends.
- The person at the phone can end the broadcast instantly, and the audit
  log records the session and its tier.
- The iOS app completes full-control viewer sessions to desktop hosts.

## Phase 5 — iOS managed control and BLE bridge

Scope: enrolment tooling and WebDriverAgent control driver for managed
devices; Bluetooth HID bridge hardware companion with mirroring receiver
and per-device pointer calibration.

**Done when:**
- An enrolled iPhone completes a full-control session (tap, swipe, type,
  app launch) started from the console, with indicator and audit intact.
- Enrolment of a device, start to finish, is documented and repeatable by
  a fleet operator without engineering help.
- A stock iPhone paired to the bridge completes a full-control session
  within Bluetooth range, including pointer calibration on first pairing.
- Both tiers degrade honestly: loss of enrolment or bridge returns the
  device to view-and-guide, never to a broken session.

## Phase 6 — Fleet deployment polish

Scope: bulk enrolment, silent install across managed fleets, staged
update rollout from the owner's server, registry housekeeping (revocation,
renaming, grouping), and operator documentation.

**Done when:**
- A 50-device mixed fleet enrols from a configuration file with no
  per-device manual server entry.
- A signed update rolls out to a pilot group, then the fleet, with
  automatic rollback proven on a deliberately broken build.
- Revoking one device's key removes its access within one connection
  cycle and leaves the rest of the fleet untouched.
- An operator who has never seen the system deploys server + first five
  devices from the documentation alone, inside an hour.

## After Phase 6

External licensing is gated on the independent security review required by
`SECURITY.md` — it is not a phase, and completing Phase 6 does not waive
it.
