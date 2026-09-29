# Troubleshooting Log — Zoiper

## Issue 1 — spurious keyring authentication prompt on launch

**Symptom**: launching Zoiper on the VM triggered a GNOME popup: *"Authentification nécessaire — le mot de passe que vous utilisez pour ouvrir une session sur cet ordinateur ne correspond plus à celui de votre trousseau de connexion"* (asking for the login keyring's password).

**Cause**: unrelated to Zoiper itself. The VM's user password had earlier been reset via a GRUB single-user-mode recovery (`init=/bin/bash` + `passwd`) after being forgotten. That procedure changes the login password but does **not** touch the separate, already-encrypted GNOME keyring — so the keyring stays locked with the old password, and any application that touches it (Zoiper included, to store saved credentials) prompts for a password that no longer exists.

**Fix used**: click "Annuler" (Cancel) on the prompt — Zoiper continues to function normally without keyring access, storing its session credentials another way.

**Permanent fix, if the prompt becomes disruptive**: reset the keyring entirely (loses any previously saved passwords elsewhere, harmless on a lab VM):
```bash
rm -rf ~/.local/share/keyrings
```
A fresh keyring, synced to the current login password, is created automatically the next time one is needed.

## Issue 2 — echo test plays the greeting but never echoes the microphone

**Symptom**: dialing extension `600` played the `demo-echotest` greeting correctly, but nothing was ever echoed back — regardless of which microphone device was selected inside Zoiper.

**Cause**: VirtualBox disables audio input ("Enable Audio Input") for a VM by default. The host machine's physical microphone is never forwarded to the VM until this is explicitly enabled — independent of any in-VM software configuration. This affects every application running inside the VM, not just Zoiper.

**Fix**: on the host (not inside the VM), VirtualBox Manager → the VM's Settings → **Audio** tab → enable **"Enable Audio Input"** (and confirm "Enable Audio Output" is also on). Requires a full VM restart (not just an application restart) for the virtual audio device to reinitialize.

**Verification**: after the restart, the echo test correctly played back the captured microphone audio, with a slight delay (expected for a purely local loopback test).
