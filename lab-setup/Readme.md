# Lab Setup

1. [Architecture](./01-architecture.md) — the overall lab design, current state, and the two Docker networking decisions behind it

## [`/asterisk`](./02-asterisk)

Everything needed to reproduce the Asterisk PBX yourself — download just this folder:
- `Dockerfile`, `modules.conf`, `pjsip.conf`, `extensions.conf` — the actual config
- `README.md` — installation guide, step by step
- `troubleshooting.md` — two real bugs hit while building this, root-caused in full
