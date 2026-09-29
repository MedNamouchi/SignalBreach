# Linphone — Installation & Configuration Guide

Two different Linphone clients were tried in this lab, for two different reasons — both are documented here.

## Linphone Desktop — GUI client, used for real two-party calls

Registered as extension 6002 in this lab, specifically to test a real call between two distinct extensions (Zoiper as 6001, Linphone Desktop as 6002) rather than the single-phone echo test.

### Installation

```bash
sudo apt install -y linphone-desktop
```

### Configuration — manual SIP account, not a Linphone account

On first launch, Linphone Desktop offers to create or log into a hosted "Linphone account" — **don't use that**. Choose **"Utiliser un compte SIP"** ("Use a SIP account") instead, to point it at the self-hosted Asterisk PBX:

| Field | Value |
|---|---|
| Nom d'utilisateur (Username) | the extension number, e.g. `6002` |
| Domaine SIP (Domain) | `127.0.0.1` (or the Asterisk host's IP) |
| Mot de passe (Password) | matches `pjsip.conf` for that extension, e.g. `ChangeMe6002!` |
| Transport | UDP |

Confirm registration from the Asterisk side:
```bash
docker exec asterisk-pbx asterisk -rx "pjsip show endpoints"
```

See [`troubleshooting.md`](./troubleshooting.md) for a real port-conflict issue this client caused.

## linphone-cli (`linphonec`) — command-line client, attempted for scripted calls

The original plan called for a CLI softphone to generate reproducible, scripted test calls for QoS measurement. Ubuntu packages this as `linphone-cli`:

```bash
sudo apt install -y linphone-cli
```

**Note**: the package name doesn't match the binary name — it installs a command called `linphonec` (no dash), found via:
```bash
dpkg -L linphone-cli | grep bin
```

Inside `linphonec`, an account is added interactively:
```
proxy add
# prompts for: proxy SIP address (sip:127.0.0.1), your identity (sip:6001@127.0.0.1),
# whether to register (yes), expiration (600), and an optional route (leave blank)
```
