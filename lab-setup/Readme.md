# Lab Setup

Building the Dockerized Asterisk PBX from scratch, including every real bug hit along the way. Read in order.

1. [Architecture](./01-architecture.md) — network_mode host decision, and the two Docker networking constraints that shaped it
2. [Dockerfile & Base Image](./02-dockerfile-and-image.md) — why Ubuntu over Debian, and the final Dockerfile
3. [PJSIP Configuration](./03-pjsip-configuration.md) — modules.conf, endpoints, auth, AORs, the deliberately weak passwords
4. [Dialplan](./04-dialplan.md) — extensions.conf and the echo-test extension
5. [Troubleshooting Log](./05-troubleshooting.md) — the AOR naming bug and the Dockerfile path typo, root-caused in full
6. [Softphone Setup & Final Validation](./06-softphone-setup-and-validation.md) — Zoiper, Linphone CLI, the VirtualBox microphone fix, and the successful end-to-end test
