# Troubleshooting Log

Two real bugs hit while bringing this PBX up, documented in full — including the false leads. The debugging process matters as much as the fix, and a clean "it just worked" narrative would misrepresent how PJSIP configuration actually behaves.

## Bug 1 — `AOR '' not found for endpoint`

**Symptom**: two independent SIP clients (Zoiper and Linphone's CLI client) both send a REGISTER that authenticates correctly (no repeated 401), yet Asterisk consistently replies `404 Not Found`, logging:
```
WARNING: res_pjsip_registrar.c:1166 find_registrar_aor: AOR '' not found for endpoint '6001'
```

**Ruled out, methodically, before finding the real cause**:
- `pjsip.conf` content verified character by character — correct
- Hidden/Windows line-ending characters checked with `cat -A` — none found
- The file actually loaded inside the running container verified directly (`docker exec ... cat`) — correct
- `ufw` firewall status checked — inactive
- Two unrelated SIP clients producing the exact same failure ruled out a client-specific bug

**Root cause**, confirmed against matching reports on the official Asterisk community forum: both clients send a REGISTER with a Request-URI of `sip:127.0.0.1;transport=UDP` — no username, which is actually RFC 3261-compliant (a REGISTER Request-URI should be domain-only). Asterisk's AOR resolution for an incoming REGISTER does **not** go through the endpoint's `aors=` reference to translate a name — it looks directly for an AOR object whose **name** matches the identified username (`6001`) exactly, regardless of what the endpoint's `aors=` field says. Naming the AOR `6001-aor` (instead of `6001`) silently broke this resolution, even though the reference `aors=6001-aor` was syntactically valid and `pjsip show endpoint` displayed everything as expected.

**Fix**: renamed the AOR sections to match the endpoint name exactly (`[6001]` with `type=aor`, instead of `[6001-aor]`) — Asterisk allows multiple sections sharing the same bracketed name in one file, as long as their `type=` differs (endpoint / auth / aor). The `auth` object (`6001-auth`, with its suffix) was never affected by this constraint — only the AOR is. This is why the shipped `pjsip.conf` in this folder uses `[6001]` twice (once as `type=endpoint`, once as `type=aor`).

**Verification**: a `tcpdump -i lo -n port 5060` capture showing a `200 OK` response to REGISTER, followed by `pjsip show endpoints` confirming the endpoint state moved from `Unavailable` to `Not in use`, with a real `Contact` listed.

## Bug 2 — a Dockerfile typo silently loading the wrong dialplan

**Symptom**: after fixing Bug 1 and successfully registering an extension, dialing extension `600` (the echo test) returned `404 Not Found` on the INVITE. `asterisk -rx "dialplan show internal"` returned: `There is no existence of 'internal' context` — the context didn't exist at all.

**Cause**: an earlier version of the Dockerfile had `COPY extensions.conf /etc/asterik/extensions.conf` — a typo, "asterik" instead of "asterisk". The file was copied to a path with no effect, and Asterisk silently fell back to loading its own bundled sample `extensions.conf` (with contexts like `[demo]`, `[default]`) — with no error raised anywhere in the build or boot logs.

**Fix**: corrected to `COPY extensions.conf /etc/asterisk/extensions.conf` (the version shipped in this folder), followed by a full rebuild and container redeploy.

**Verification**: `dialplan show internal` listed all extensions correctly, sourced from `extensions.conf`.

**Takeaway**: a `COPY` path typo in Docker fails silently at the application layer — the build succeeds, the container starts, and the application falls back to its own defaults without complaint. The useful habit is verifying what the application actually loaded (`dialplan show <context>`), rather than assuming a successful `COPY` at build time means the intended file is in use.

## Bonus — a port conflict after a VM reboot

**Symptom**: after rebooting the VM, a previously-working softphone registration started failing with `405 Method not allowed`, then `408 Request Timeout`.

**Cause**: another local SIP client (Linphone Desktop, running natively alongside the container) had claimed UDP port 5060 for itself before the Asterisk container restarted, so registrations were hitting the wrong process entirely.

**Fix**: changed the competing client's local SIP port, then restarted the Asterisk container so it could reclaim port 5060.

**Takeaway**: `sudo ss -tulnp | grep 5060` is the fastest way to confirm *which* process is actually listening on Asterisk's port before assuming a configuration problem.
