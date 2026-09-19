# QoS Stack Placement

Quality of Service (QoS) mechanisms guarantee that a stream arrives with acceptable quality , low latency, low jitter, low loss. QoS is not confined to the network layer (routers, packet scheduling); it depends on information coming from the application layer above. This is where the signaling and media protocols fit in.

## Three stages

```
1. Signaling (SIP, H.323, RTSP)
        ↓ negotiates codec, ports, priority — before the stream even starts
2. Media transport (RTP)
        ↓ carries the stream, header includes sequence number + timestamp
3. Measurement / feedback (RTCP)
        ↓ computes jitter, loss, RTT, MOS in real time
Network (global QoS: scheduling, prioritization, SLA)
```

1. **Signaling** (SIP, H.323, RTSP) prepares the ground: it negotiates the parameters that will impact QoS (codec choice, ports, priority) before any media flows.
2. **Media transport** (RTP) carries the actual stream, with header fields (sequence number, timestamp) essential to QoS measurement.
3. **Measurement / feedback** (RTCP) computes the real QoS metrics in real time — jitter, loss, RTT, MOS — and reports them, feeding into network-level prioritization and SLA enforcement.

## One-line summary

> SIP/H.323/RTSP prepare the ground for QoS to be possible; RTP transports the stream under real-time constraints; RTCP measures whether the promised QoS is actually being delivered.

---
[← Previous](./02-protocols-overview.md) · [Back to index](./README.md) · [Next: SIP vs. H.323 →](./04-sip-vs-h323.md)
