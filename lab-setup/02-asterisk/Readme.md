# Asterisk PBX — Config & Installation Guide

Everything needed to reproduce this lab's Asterisk PBX. Download just this folder and follow the steps below — no need for anything else in the repository.

## What's in here

| File | Purpose |
|---|---|
| `Dockerfile` | Builds the image: Ubuntu 24.04 base, Asterisk 20 (LTS) installed from the `universe` repo, the 3 config files below copied in |
| `modules.conf` | Disables the legacy `chan_sip` module (in favor of modern PJSIP) and the voicemail modules (unused, kept logs clean) |
| `pjsip.conf` | Defines 3 SIP extensions (6001, 6002, 6003) — transport, authentication, and address-of-record (AOR) for each |
| `extensions.conf` | The dialplan: routes calls between extensions, plus a self-testing echo extension (`600`) |
| `troubleshooting.md` | Two real bugs hit while building this, root-caused in full |

## Prerequisites

- Docker Engine + Docker Compose plugin
- A Linux host (or VM) — `network_mode: host` below is Linux-specific
- `sudo` access

## 1. Build the image

```bash
docker build -t signalbreach-asterisk .
```

## 2. Run the container

```bash
docker run -d --network host --name asterisk-pbx signalbreach-asterisk
```

**Why `--network host`**: Docker's default bridge network applies NAT to outgoing traffic, which rewrites UDP source ports — this breaks RTP media negotiation for real-time voice traffic. Running Asterisk directly on the host's network stack avoids this entirely. The tradeoff: Asterisk binds straight to the host's `0.0.0.0:5060`, so make sure nothing else is already using that port (see `troubleshooting.md` for a real case of this happening after a VM reboot).

## 3. Verify it's running correctly

```bash
docker logs asterisk-pbx
```

Look for `Asterisk Ready.` at the end. Then check the 3 extensions loaded correctly:

```bash
docker exec asterisk-pbx asterisk -rx "pjsip show endpoints"
```

Expected: `Objects found: 3`, each showing `Unavailable` (nothing has registered yet — expected at this point) with the `Aor` name matching the endpoint exactly (this matters — see `troubleshooting.md`).

Check the dialplan loaded too:

```bash
docker exec asterisk-pbx asterisk -rx "dialplan show internal"
```

Expected: extensions `600`, `6001`, `6002`, `6003` listed, each sourced from `extensions.conf`.

## 4. Register a softphone

Any SIP softphone (Zoiper, Linphone, MicroSIP...) can register using:

| Field | Value |
|---|---|
| Domain / Server | `127.0.0.1` (or the host's IP, if connecting from another machine) |
| Username | `6001`, `6002`, or `6003` |
| Password | `ChangeMe<extension>!` (e.g. `ChangeMe6001!`) — see the note below |
| Transport | UDP |

Confirm registration succeeded:

```bash
docker exec asterisk-pbx asterisk -rx "pjsip show endpoints"
```

The registered extension should now show `Not in use` with a real `Contact` listed, instead of `Unavailable`.

## 5. Test the echo extension

Dial **`600`** from any registered softphone. You'll hear an instructional greeting, then everything your microphone sends is echoed back to you — a self-contained way to validate registration, dialplan routing, and bidirectional RTP media in a single call, without needing a second phone.

To test a real call between two people, register a second softphone as a different extension and dial one from the other (e.g. dial `6002` from the phone registered as `6001`).

## A note on the weak passwords

The passwords in `pjsip.conf` (`ChangeMe6001!`, etc.) are intentionally weak — a deliberate, documented vulnerability used in this project's later reconnaissance/exploitation phase. Don't reuse this `pjsip.conf` as-is outside of an isolated lab environment.
