# Lab Architecture

## Overview

The lab is a Dockerized VoIP environment. As it stands today, every component (Asterisk in its container, and both softphones directly on the VM) communicates over the **loopback interface (127.0.0.1)** — no bridge network or containerized softphones are in place yet. This section describes the current state precisely, and what's planned next.

```
Ubuntu VM (VirtualBox)
│
├── Docker container: asterisk        → SIP PBX (Asterisk 20, PJSIP)
│                                        network_mode: host
│                                        listens on 0.0.0.0:5060 (reachable via 127.0.0.1)
│
├── Zoiper (native app on the VM)      → registered as extension 6001
├── Linphone Desktop (native app on the VM) → registered as extension 6002
│
└── All SIP/RTP/RTCP traffic between the above → 127.0.0.1 (loopback)

Planned, not yet built:
├── mediamtx        → RTSP server
├── suricata        → IDS, network_mode: host
├── linphone-cli × 2–3 (containerized, on an isolated bridge network 172.20.0.0/24)
└── attacker container (SIPVicious, SIPp, ARP spoofing tools)
```

## Current state: everything through loopback

Both softphones used so far — **Zoiper** and **Linphone Desktop** — run as native applications directly on the Ubuntu VM itself, not inside containers and not on a separate machine. Asterisk runs in a Docker container with `network_mode: host`, which means it shares the VM's own network stack rather than having an isolated IP. The practical consequence: every SIP registration, INVITE, and RTP/RTCP packet captured so far shows `127.0.0.1 → 127.0.0.1` — a single machine talking to itself over loopback, with Asterisk as the PBX in the middle.

This is why `tc netem` applied to the `lo` interface degrades the *entire* call path (signaling and media both) — there is currently only one interface in play.

## Components

**Asterisk** — the SIP PBX. Self-built image (no FreePBX) so every configuration file stays plain text and version-controlled. Built, configured with 3 extensions, and validated end-to-end (see [Dockerfile & Base Image](./02-dockerfile-and-image.md), [PJSIP Configuration](./03-pjsip-configuration.md), [Dialplan](./04-dialplan.md)).

**Zoiper** — GUI softphone running natively on the VM, registered as extension 6001. Used for manual testing and live demonstration (registration, calls, the echo test).

**Linphone Desktop** — GUI softphone running natively on the VM, registered as extension 6002. Added specifically to test a real call between two distinct extensions (rather than the single-phone echo test), since `linphonec` (Linphone's CLI client) turned out not to send any traffic at all — see [Troubleshooting Log](./05-troubleshooting.md).

**MediaMTX** *(planned)* — the RTSP server that will stand in for an IP camera / video stream, for the unauthorized-access scenario in the exploitation phase.

**Suricata** *(planned)* — the IDS responsible for detecting SIP floods and SIPVicious scans. Needs full traffic visibility, which is what drives the `network_mode: host` decision described below — relevant once the lab moves off pure loopback.

**Linphone-cli containers / attacker container** *(planned)* — scriptable softphones and offensive tooling (SIPVicious, SIPp, ARP spoofing) for the reconnaissance and exploitation phase, intended to live on an isolated bridge network once that phase begins.

**Packet capture & analysis** — `tcpdump`/`tshark` and a set of Python scripts (RTCP metric extraction, static plotting, a live 4-panel QoS monitor) have been the core tools for the QoS baseline work — see the `/qos-baseline` folder.

## Network design decisions (for when the lab grows beyond loopback)

Two Docker networking constraints were identified before planning the next components, and they will shape the architecture once MediaMTX, Suricata, and containerized attack tooling are added.

### Constraint 1 — a Docker bridge network behaves like a real switch

A default Docker bridge network learns MAC addresses and only forwards unicast traffic to the right port — it does not behave like a hub. Consequence: an IDS container in "promiscuous mode" on its own interface would never see traffic exchanged between two *other* containers by default.

This will affect two parts of the project directly once built:
- **Suricata** will need full visibility into SIP/RTP traffic between the PBX and softphones.
- **RTP eavesdropping** (exploitation phase) won't be able to rely on simple passive sniffing on a switched network — a real ARP spoofing attack will be used to redirect traffic through the attacker first. Arguably a more realistic red-team technique anyway.

**Planned resolution**: Suricata will run with `network_mode: host`, capturing the host's Docker bridge interface directly.

### Constraint 2 — Docker NAT and RTP/UDP

Documented both by MediaMTX and the Asterisk community: when a client outside the Docker network reaches a server inside Docker through published ports, Docker's NAT can rewrite UDP source ports — breaking RTP media negotiation.

**Resolution already applied to Asterisk, and planned for MediaMTX**: `network_mode: host`, listening directly on the host's IP, bypassing the bridge's NAT entirely. This is already why Asterisk behaves correctly today.

**Verification**: `sudo ss -tulnp | grep 5060` on the host (outside any container) confirms Asterisk listens directly on `0.0.0.0:5060` — concrete proof the host networking mode works as intended.

---
[← Back to index](./README.md) · [Next: Dockerfile & Image →](./02-dockerfile-and-image.md)
