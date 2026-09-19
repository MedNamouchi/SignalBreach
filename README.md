# SignalBreach

> SIP/H.323/RTSP signaling lab — attack, measure QoS impact, detect, and remediate.

Breaking VoIP to understand it: a SIP/RTSP signaling lab with live attacks, real-time QoS impact measurement, and blue team detection — built from a SOC analyst's perspective moving toward offensive security.

---

## Table of Contents

- [Context & Objectives](#context--objectives)
- [Architecture](#architecture)
- [Project Phases](#project-phases)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Status](#status)
- [Author](#author)

---

## Context & Objectives

Most coverage of multimedia signaling protocols (SIP, H.323, RTP/RTCP, RTSP) stays theoretical. This project turns it into an end-to-end, measured case study:

1. **QoS, measured, not just described** — jitter, latency, packet loss, scheduling and SLA impact, backed by real captures and metrics rather than textbook numbers.
2. **Full attack/defense lifecycle** — build a real PBX/streaming lab, measure its baseline QoS, break it (DoS, spoofing, eavesdropping), detect the attacks (Suricata), fix the root causes (SRTP/TLS), then replay the attacks to prove the fix works.
3. **Portfolio** — document the offensive-security learning path publicly, building on hands-on Blue Team / Detection Engineering experience (SOC internship at OFIR LTD, a Croatian MSSP).

## Architecture

The lab runs entirely on Docker, on an isolated network. Two Docker networking limitations were identified and solved during design (see [`/lab-setup`](./lab-setup) for details):

- Docker bridge networks behave like a real switch (MAC learning), so passive sniffing from a third container does not see traffic between two other containers by default.
- UDP/RTP traffic crossing Docker's NAT boundary can break SIP media negotiation.

```
network_mode: host
├── asterisk    → SIP PBX (PJSIP, raw config — no FreePBX abstraction)
├── mediamtx    → RTSP server
└── suricata    → IDS, captures the Docker bridge interface directly

isolated bridge network (172.20.0.0/24)
├── linphone-cli × 2–3   → scriptable softphones (automation, reproducible tests)
└── attacker container    → SIPVicious, SIPp, ARP spoofing tools

host machine
└── Zoiper (GUI softphone, live demo)
```

## Project Phases

### Phase 1 — State of the Art
Signaling vs. media separation, real-time constraint, RFC-level coverage of SIP, H.323, RTP, RTCP and RTSP, SIP vs. H.323 comparison, QoS stack placement, and a legacy (SIP/H.323) vs. modern (WebRTC) signaling comparison illustrated with Jitsi Meet / BigBlueButton.

### Phase 2 — Lab & QoS Baseline
- **2.1 Lab deployment** — Dockerized PBX, RTSP server, softphones, IDS.
- **2.2 QoS variation measurement** — reproducible calls (Linphone CLI), `tcpdump` capture, automated RTCP metric extraction (`pyshark`), artificial degradation (`tc netem`), before/after comparison, visualization (matplotlib/plotly).

### Phase 3 — Security's Effect on Quality of Service
- **3.1 Recon & exploitation** — SIPVicious suite (`svmap`, `svwar`, `svcrack`, `svreport`, `svcrash`), then at least two attacks: DoS (INVITE flood), caller ID spoofing, RTP eavesdropping (via ARP spoofing), unauthorized RTSP access.
- **3.2 Attack impact on QoS** — Suricata detection rules, synchronized dashboard (QoS metrics + IDS alerts + attack status).
- **3.3 Remediation & validation** — SRTP + TLS on Asterisk, Fail2ban, replay of the same attacks to prove they now fail and QoS stays stable.

## Tech Stack

`Docker` · `Docker Compose` · `Asterisk (PJSIP)` · `MediaMTX` · `Linphone CLI` · `Zoiper` · `Suricata` · `SIPVicious` · `SIPp` · `Python (pyshark, matplotlib/plotly)` · `tc netem` · `Wireshark` · `Fail2ban`

## Repository Structure

```
.
├── etat-art/           # State of the art, protocol comparisons, RFC notes
├── lab-setup/           # Docker Compose files, Dockerfiles, network design notes
├── qos-baseline/        # Baseline measurement scripts, netem scenarios, graphs
├── recon-exploitation/  # SIPVicious/SIPp scripts, attack scripts, captures
├── detection/           # Suricata rules, dashboard exports
├── remediation/         # TLS/SRTP config, Fail2ban rules, replay results
└── README.md
```

## Status

🚧 Work in progress — Phase 1 (State of the Art) underway.

## Author

**Mohamed Amine Namouchi** — Final-year engineering student, Network Security & Quality, Polytech Dijon. SOC Analyst / Blue Team / Detection Engineer apprentice at Altinea, building toward offensive security.
