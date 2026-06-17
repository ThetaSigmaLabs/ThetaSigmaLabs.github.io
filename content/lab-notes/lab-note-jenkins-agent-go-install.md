+++
date = '2026-06-02T00:00:00-03:00'
title = 'Lab Note: Jenkins Agent Setup — Go, MinGW, and NSSM on Windows'
Status = 'Confirmed Working'
tags = ["Jenkins", "Go", "Windows", "NSSM", "CI/CD", "The Moment"]
project = 'The Moment'
+++

## Overview

Notes for setting up Jenkins build agents across all three lab nodes (Oroid N2+/arm64, Beelink/Windows, MiniMac/macOS).
Covers Go installation per platform, MinGW gcc on Windows for CGO builds, and running the
Jenkins agent as a persistent Windows service via NSSM.

---

## Installing Go

### Windows — beelink

```bash
winget install -e --id GoLang.Go
```

Open a **new** cmd prompt after install (PATH won't update in the existing window).

```bash
go version
```

Expect: `go version go1.24.x windows/amd64`

Check the module cache paths are writable. This matters if the Jenkins agent runs as a service
account rather than the interactive user:

```bash
go env GOPATH
go env GOCACHE
```

Both should point inside the service account's profile. If not, set `GOPATH` and `GOCACHE`
explicitly in the Jenkins node environment variables.

---

### macOS — miniMac

```bash
brew install go
```

Or download the `.pkg` installer from `https://go.dev/dl/` and run it.

```bash
go version
```

---

### Linux ARM64 — ODroid N2+

Download the ARM64 tarball from `https://go.dev/dl/` (filename: `go1.24.x.linux-arm64.tar.gz`).

```bash
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.24.x.linux-arm64.tar.gz
```

Add to PATH permanently:

```bash
echo 'export PATH=/usr/local/go/bin:$PATH' | sudo tee /etc/profile.d/go.sh
sudo chmod +x /etc/profile.d/go.sh
source /etc/profile.d/go.sh
go version
```

**Jenkins caveat:** The Jenkins agent process does not source `/etc/profile.d/`. The Go binary
path must be hardcoded in the Jenkinsfile `environment` block:

```groovy
environment {
    PATH = '/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
}
```

---

## Installing MinGW-w64 gcc on Windows — beelink

Required for `mattn/go-sqlite3` which uses `CGO_ENABLED=1` and needs a C compiler.
Pure-Go SQLite alternatives (e.g. `modernc.org/sqlite`) would avoid this, but The Moment
uses `mattn/go-sqlite3`.

**Option A — Chocolatey (simplest):**

```bash
choco install mingw
```

**Option B — MSYS2:**

```bash
winget install -e --id MSYS2.MSYS2
```

Then in the MSYS2 MinGW64 shell:

```bash
pacman -S mingw-w64-x86_64-gcc
```

Add `C:\msys64\mingw64\bin` to the **system** PATH (not user PATH — the Jenkins service
account needs it).

**Verify:**

```bash
gcc --version
```

Expect: `gcc (x86_64-posix-seh-rev..., Built by MinGW-W64...) 13.x`

---

## Running the Jenkins Agent as a Windows Service — NSSM

NSSM (Non-Sucking Service Manager) wraps any executable as a proper Windows service with
restart-on-failure and log capture. Much cleaner than Task Scheduler for a Jenkins agent.

### 1. Install Java 21 (Temurin)

Download the Windows x64 MSI from `https://adoptium.net/temurin/releases/?version=21`

Run the installer. On the optional features screen, tick:
- **Add to PATH**
- **Set JAVA_HOME variable**

Verify:

```bash
java -version
```

Expect: `openjdk version "21.x.x" ... Temurin-21`

---

### 2. Install NSSM

Download from `https://nssm.cc/download` — get the latest release zip.

Extract to `C:\tools\nssm`. Add `C:\tools\nssm\win64` to the **system** PATH.

Verify:

```bash
nssm version
```

---

### 3. Download agent.jar

In Jenkins UI: **Manage Jenkins → Nodes → beelink → Agent** (or the node's status page).
The page shows a launch command containing the secret and a download link for `agent.jar`.

Save to: `C:\jenkins-agent\agent.jar`

Also create the log directory:

```bash
mkdir C:\jenkins-agent\logs
```

---

### 4. Create the service

```bash
nssm install JenkinsAgent
```

This opens a GUI. Fill in the **Application** tab:

| Field | Value |
|---|---|
| Path | `C:\Program Files\Eclipse Adoptium\jdk-21.x.x.x-hotspot\bin\java.exe` |
| Startup directory | `C:\jenkins-agent` |
| Arguments | `-jar agent.jar -url http://<jenkins-ip>:8085 -name <agent-name> -secret <SECRET> -workDir C:\jenkins-agent` |

Switch to the **I/O** tab:

| Field | Value |
|---|---|
| Stdout | `C:\jenkins-agent\logs\stdout.log` |
| Stderr | `C:\jenkins-agent\logs\stderr.log` |

Click **Install service**.

---

### 5. Start and verify

```bash
nssm start JenkinsAgent
```

Check Jenkins UI: **Manage Jenkins → Nodes → beelink** — the node should show as online
within a few seconds.

Check the logs if it doesn't connect:

```bash
type C:\jenkins-agent\logs\stderr.log
```

---

### NSSM quick-reference

```bash
nssm status  JenkinsAgent        # Running / Stopped / paused
nssm restart JenkinsAgent        # Restart the service
nssm stop    JenkinsAgent        # Stop without removing
nssm edit    JenkinsAgent        # Re-open the GUI to change settings
nssm remove  JenkinsAgent confirm  # Permanently remove the service
```

To update the agent secret (e.g. after re-registering the node in Jenkins):

```bash
nssm edit JenkinsAgent
```

Change the Arguments field, save, then `nssm restart JenkinsAgent`.
