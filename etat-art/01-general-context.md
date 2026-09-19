# General Context — Signaling vs. Media

Every multimedia communication over IP (a phone call, a video stream) splits into two strictly separate roles:

| Role | Definition | Protocols |
|---|---|---|
| **Signaling** | Establishing, controlling and terminating the session: "I'm calling you", negotiating parameters (codec, port), ringing, answering, hanging up | SIP, H.323, RTSP |
| **Media (real-time transport)** | Carrying the actual audio/video stream once the session is established, under strict timing constraints | RTP (measured by RTCP) |

**Analogy**: dialing a number, waiting for it to ring, and the callee picking up is signaling. The conversation itself is media.

**Key concept: real time.** A late audio packet is as good as a lost one — unlike a file transfer, where waiting is an option. This constraint is the reason the entire QoS toolbox (jitter, latency, packet loss, scheduling) exists.

## Why separate signaling and media into different protocols?

### Modularity
RTP is an independent, reusable transport building block. Any signaling protocol — SIP, RTSP, H.323 — can plug RTP in behind it to carry media. Both a SIP-based phone call and an RTSP-controlled camera stream end up using the exact same RTP transport. This is comparable to IP serving as the common layer under both TCP and UDP: nobody has to reinvent real-time transport for every new signaling protocol.

### Independent security
Signaling (SIPS/TLS) and media (SRTP) can be encrypted independently of each other. Media ports are negotiated dynamically for each session, so compromising the signaling channel (e.g. spoofing a SIP INVITE) does not automatically expose the media content — a point directly relevant to the eavesdropping and spoofing attacks in this project's exploitation phase.

### Differentiated QoS
SIP signaling traffic is low-volume and can tolerate some latency. RTP media traffic is highly jitter-sensitive and needs to be prioritized on the network (DSCP marking, scheduling). Separating the flows lets each be governed by different QoS policies.

### Scalability
A signaling proxy can handle thousands of sessions without ever touching the media stream, which often flows directly, peer-to-peer, between the two endpoints — offloading the central infrastructure.

---
[← Back to index](./README.md) · [Next: Protocol Overview →](./02-protocols-overview.md)
