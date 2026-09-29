# Lab Setup

1. [Architecture](./01-architecture.md) — the overall lab design, current state, and the two Docker networking decisions behind it

## [`/asterisk`](./02-asterisk)

Everything needed to reproduce the Asterisk PBX yourself — download just this folder:
- `Dockerfile`, `modules.conf`, `pjsip.conf`, `extensions.conf` — the actual config
- `README.md` — installation guide, step by step
- `troubleshooting.md` — two real bugs hit while building this, root-caused in full

## [`/zoiper`](./03-zoiper)
 
Installing and configuring the Zoiper softphone against the Asterisk PBX above:
- `README.md` — installation and manual configuration guide
- `troubleshooting.md` — a spurious keyring prompt, and a VirtualBox microphone setting that silently blocks audio input

## [`/linphone`](./04-linphone)
 
Two Linphone clients tried — Linphone Desktop (GUI, used for real two-party call testing) and `linphonec` (CLI, attempted for scripted calls):
- `README.md` — installation and configuration for both
- `troubleshooting.md` — why `linphonec` was dropped in favor of SIPp for automation, and a port conflict between Linphone Desktop and Asterisk
