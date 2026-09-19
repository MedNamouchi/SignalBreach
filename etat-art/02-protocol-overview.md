# Protocol Overview — SIP, H.323, RTP, RTCP, RTSP

## SIP — Session Initiation Protocol (RFC 3261)

A text-based signaling protocol, standardized by the IETF in 1999, styled after HTTP and SMTP. It establishes, modifies and terminates sessions between two or more participants.

**Architecture** (see [SIP vs. H.323](./04-sip-vs-h323.md) for the full breakdown):
- **User Agent (UA)** — the endpoint (softphone)
- **Registrar** — the directory: each UA sends a `REGISTER` message announcing "I'm extension X, reachable at this address"
- **Proxy** — relays `INVITE` messages between UAs; the caller never contacts the callee directly for signaling

Example INVITE message:
```
INVITE sip:6002@192.168.1.10 SIP/2.0
Via: SIP/2.0/UDP 192.168.1.5:5060
From: "Amine" <sip:6001@192.168.1.10>
To: <sip:6002@192.168.1.10>
Call-ID: a84b4c76e66710@pc33.local
CSeq: 1 INVITE
Contact: <sip:6001@192.168.1.5>
Content-Type: application/sdp
```

Media negotiation is carried inside the SIP message body via **SDP** (RFC 4566) — see below.

## H.323 (ITU-T, 1996)

Not a single protocol but a stack of sub-protocols, standardized by the ITU-T — the traditional telecom standards body (same lineage as ISDN):

- **H.225** — call setup/teardown, includes a **Q.931** sub-part inherited from ISDN
- **H.245** — separate negotiation of media capabilities (codecs)
- **RAS (Registration, Admission, Status)** — terminal registration with a **Gatekeeper** (the H.323 equivalent of a SIP Registrar)

Messages are binary-encoded (ASN.1 / PER), unlike SIP's plain text. Establishing a call requires several separate protocol exchanges (H.225, then H.245), making it heavier than SIP's single INVITE + embedded SDP.

## RTP — Real-time Transport Protocol (RFC 3550)

Carries the actual audio/video stream, always over **UDP**, never TCP.

**Why UDP**: TCP guarantees delivery and ordering but blocks delivery to the application until a lost packet is retransmitted — for a real-time stream, this creates a growing silence/freeze with every loss. UDP guarantees nothing, but never blocks: a lost packet is simply skipped, producing a brief glitch instead of a full stall.

**Key header fields**:
- **Sequence number** (16-bit, incremented per packet) — detects **packet loss** (e.g. receiving 1, 2, 3, 5, 6 reveals packet 4 is missing)
- **Timestamp** — marks the emission time of each packet, used to detect **jitter**

## RTCP — RTP Control Protocol (RFC 3550)

Monitors transmission quality using the RTP header fields above.

**Jitter**: the *variation* in inter-packet arrival delay — not the delay itself (latency), but its irregularity. RTCP computes it as a running average of the difference between the theoretical inter-packet spacing (from timestamps) and the actual measured spacing at arrival.

**Metric chain feeding into QoS**:
1. Sequence number → detects loss
2. Timestamp → detects jitter
3. Loss + jitter + RTT (measured separately by RTCP) → feed into the **MOS** (Mean Opinion Score, 1–5), the industry-standard perceived-quality metric — see [QoS Stack Placement](./03-qos-stack-placement.md).

## RTSP — Real-Time Streaming Protocol (RFC 2326 / RFC 7826)

Unlike SIP (a bidirectional call between two people), RTSP controls the playback of an already-existing media stream (an IP camera, a VOD server) — like a remote control.

```
OPTIONS   → which methods the server supports
DESCRIBE  → describe the available stream (response is SDP — same format as SIP)
SETUP     → negotiate transport (RTP port, protocol) — never the video content itself
PLAY      → start the RTP stream
PAUSE     → pause it
TEARDOWN  → close the session
```

RTSP typically runs over **TCP** (port 554) for its control messages, unlike SIP's default UDP — control commands (PLAY, PAUSE) are infrequent and need reliable delivery. The actual video stream still runs over RTP/UDP for the same real-time reasons as any other media flow.

**Unidirectional**: unlike SIP where both UAs exchange RTP in both directions, RTSP media flows only one way — server to client.

## SDP — Session Description Protocol (RFC 4566)

Not a standalone protocol — a description format carried inside SIP (in the INVITE body) or RTSP (in the DESCRIBE response). It describes:
- Supported audio/video codecs (G.711, G.729, Opus...)
- The RTP port to listen on
- The IP address to use for media

```
m=audio 5004 RTP/AVP 0
a=rtpmap:0 PCMU/8000
```

SDP being reused identically by both SIP and RTSP is direct evidence that it's an independent, reusable building block — same principle as RTP itself.

---
[← Previous](./01-general-context.md) · [Back to index](./README.md) · [Next: QoS Stack Placement →](./03-qos-stack-placement.md)
