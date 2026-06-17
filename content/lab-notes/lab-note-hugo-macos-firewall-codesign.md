+++
date = '2026-06-10T14:30:00-03:00'
title = 'Lab Note: Signing Hugo so macOS Firewall Allows LAN Access to the Dev Server'
Status = 'Confirmed Working'
tags = ["Hugo", "macOS", "Firewall", "codesign", "Networking"]
project = 'ThetaSigma Site'
hardware = 'Mac (miniMac)'
+++

## Overview

The Hugo dev server worked on `localhost` but was unreachable on the Mac's LAN IP
(`<mac-ip>`), even with the correct `--bind` flag. Root cause: the macOS application
firewall identifies apps by **code signature**, and Homebrew's `hugo` binary is shipped
completely unsigned — so even an explicit firewall allow rule could not attach to it.
Fix: ad-hoc sign the binary.

---

## Symptom

```bash
hugo server -D --bind <mac-ip> --baseURL http://<mac-ip>
```

- `http://localhost:1313` → works
- `http://<mac-ip>:1313` → `curl: (52) Empty reply from server`

TCP connects, then the connection dies with no HTTP response. Loopback traffic never
crosses the application firewall, which is why `localhost` always worked.

---

## Diagnosis chain

1. Confirm Hugo is actually listening on the LAN IP:

   ```bash
   lsof -nP -iTCP:1313 -sTCP:LISTEN
   # hugo ... TCP <mac-ip>:1313 (LISTEN)
   ```

2. Confirm the firewall is on:

   ```bash
   /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
   # Firewall is enabled. (State = 1)
   ```

3. Check the binary's signature — this is the key step:

   ```bash
   codesign -dv /usr/local/bin/hugo
   # /usr/local/bin/hugo: code object is not signed at all
   ```

4. Adding a firewall allow rule (`socketfilterfw --add` / `--unblockapp`) reported
   `Incoming connection to /usr/local/bin/hugo is permitted` — but connections were
   still dropped. The rule exists; it just can't bind to an unsigned executable.

---

## Fix

Ad-hoc sign the real binary (the `/usr/local/bin` path is a symlink into the Cellar):

```bash
codesign -s - --force /usr/local/Cellar/hugo/0.161.1/bin/hugo
```

Verify the signature took:

```bash
codesign -dv /usr/local/bin/hugo
# Identifier=hugo-<hash>
```

Restart the dev server (the firewall verdict is made when the process starts listening):

```bash
hugo server -D --bind <mac-ip> --baseURL http://<mac-ip>
```

Verify from the LAN:

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://<mac-ip>:1313/
# 200
```

---

## Caveats

- **`brew upgrade hugo` undoes this.** A new version means a new unsigned binary at a
  new Cellar path — re-run the `codesign` command (and re-check the firewall rule)
  after every upgrade.
- A firewall daemon in a stale state can drop traffic for *all* local services after
  rule changes. If unrelated ports stop responding on the LAN IP, restart it:

  ```bash
  sudo pkill socketfilterfw   # launchd respawns it
  ```

- `--bind 0.0.0.0` exposes the server on every interface; prefer binding the specific
  LAN IP for dev work.
