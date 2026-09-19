# SIP vs. H.323

| | H.323 | SIP |
|---|---|---|
| Standards body | ITU-T (traditional telecom) | IETF (Internet) |
| Year | 1996 | 1999 (RFC 3261) |
| Message format | Binary (ASN.1 / PER) | Plain text, HTTP/email-style |
| Cultural origin | Telecom world (ISDN heritage) | Internet world |
| Media negotiation | Separate protocol (H.245), several exchanges | Embedded SDP in the first INVITE, one round-trip |

## Why SIP ended up dominating IP networks

The text format is simpler to implement, extend and debug — a SIP packet is directly readable in Wireshark, no decoding tool required. The H.323 binary format is more compact and faster to parse on the machine side, an advantage that mattered in 1996 over low-bandwidth links, but a less decisive factor once IP networks became the norm.

## Structural reason H.323 is heavier to establish a call

SIP bundles signaling and media negotiation into a single protocol + embedded SDP — one round-trip. H.323 splits the same job across several separate sub-protocols (H.225, H.245, RAS, Q.931 — see [Protocol Overview](./02-protocols-overview.md)), each requiring its own exchange. This is the concrete, structural reason H.323 call setup is slower and heavier than SIP's, not just an assertion.

---
[← Previous](./03-qos-stack-placement.md) · [Back to index](./README.md) · [Next: Jitsi Meet vs. BigBlueButton →](./05-jitsi-vs-bigbluebutton.md)
