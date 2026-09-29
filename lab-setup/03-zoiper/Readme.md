# Zoiper — Installation & Configuration Guide

Zoiper is the GUI softphone used in this lab for manual testing and live demonstration (registration, calls, the echo test) against the Asterisk PBX in [`/asterisk`](../asterisk).

## Installation (Ubuntu / Debian)

Zoiper isn't in the standard apt repositories — install from the official `.deb`:

```bash
wget "<download-link-from-zoiper.com>" -O zoiper.deb
sudo dpkg -i zoiper.deb
sudo apt --fix-broken install   # resolves dependencies dpkg doesn't handle alone
```

## Configuration — manual setup only

Zoiper's automatic setup wizard tries to auto-detect the right protocol/transport for the account you enter (`username@domain`) by probing SIP TLS / TCP / UDP / IAX in sequence. **In practice, against a self-hosted Asterisk on `127.0.0.1`, this auto-detection is unreliable** — it reported `SIP UDP: Not found` even when the server was correctly listening and reachable (confirmed independently with `ss -tulnp` and `tcpdump`). Skip the wizard and configure manually instead:

1. Add a new account, choose **SIP** as the protocol explicitly (not the "automatic" account type)
2. **Username**: the extension number (e.g. `6001`)
3. **Domain / Host**: `127.0.0.1` (or the Asterisk host's IP)
4. **Password**: matches `pjsip.conf` for that extension (e.g. `ChangeMe6001!`)
5. **Transport**: UDP
6. Skip the "Authentication username" / "Outbound proxy" screen — not needed here
7. When prompted to pick an audio device, choose the default autoselect option

Confirm registration succeeded from the Asterisk side:
```bash
docker exec asterisk-pbx asterisk -rx "pjsip show endpoints"
```
The extension should show `Not in use` with a real `Contact`, instead of `Unavailable`.

## Known issues specific to this softphone

See [`troubleshooting.md`](./troubleshooting.md) for two Zoiper-specific problems hit in this lab: a spurious keyring authentication prompt on launch, and no captured audio during the echo test (a VirtualBox setting, not a Zoiper bug).
