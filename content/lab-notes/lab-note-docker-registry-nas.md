+++
date = '2026-06-04T00:00:00-03:00'
title = 'Lab Note: Docker Registry on Synology NAS with Browse UI'
Status = 'Confirmed Working'
tags = ["Docker", "Synology", "Jenkins", "CI/CD", "Registry", "The Moment"]
project = 'The Moment'
+++

## Overview

A private Docker registry running on the Synology NAS at `<nas-ip>`, used by the Jenkins
CI/CD pipeline to store built images and available for pulling to any LAN machine.
Includes the joxit/docker-registry-ui browse interface.

Compose file lives at `/volume1/docker/registry/docker-compose.yml` on the NAS.

---

## Why This Is Useful

**For CI/CD:** Jenkins builds The Moment, pushes the image here, then pulls it on the ARM64
production node (Odroid N2+) to deploy. No GitHub or internet dependency in the
build/deploy loop — the pipeline runs entirely on the LAN.

**For personal use:** Any machine on the home network can pull a known-good image directly.
If The Moment needs to move to a different machine (e.g. a new Odroid, a spare PC, or a
second NAS), a single `docker pull <nas-ip>:5050/the-moment:latest` gets the exact same
build that Jenkins produced and tested — no rebuilding from source, no internet required.

**Versioned history:** Every Jenkins build pushes a tagged image. Rolling back to a previous
version is `docker pull <nas-ip>:5050/the-moment:<build_number>`. The browse UI at port 5051
shows all available tags at a glance so you can pick the right one.

---

## Compose File

```yaml
services:
  registry:
    image: registry:2
    container_name: registry
    restart: always
    ports:
      - "5050:5000"
    environment:
      - REGISTRY_HTTP_HEADERS_Access-Control-Allow-Origin=['*']
      - REGISTRY_HTTP_HEADERS_Access-Control-Allow-Methods=['HEAD','GET','OPTIONS','DELETE']
      - REGISTRY_HTTP_HEADERS_Access-Control-Allow-Headers=['Authorization','Accept','Cache-Control']
      - REGISTRY_HTTP_HEADERS_Access-Control-Expose-Headers=['Docker-Content-Digest']
      - REGISTRY_HTTP_HEADERS_Access-Control-Allow-Credentials=[true]
      - REGISTRY_STORAGE_DELETE_ENABLED=true
    volumes:
      - ./data:/var/lib/registry

  ui:
    image: joxit/docker-registry-ui:latest
    ports:
      - "5051:80"
    environment:
      - SINGLE_REGISTRY=true
      - NGINX_PROXY_PASS_URL=http://registry:5000
      - REGISTRY_TITLE=NAS Registry
      - DELETE_IMAGES=true
    depends_on:
      - registry
```

- Registry API: `http://<nas-ip>:5050`
- Browse UI: `http://<nas-ip>:5051`

---

## Key Fix: NGINX_PROXY_PASS_URL vs REGISTRY_URL

The UI has two modes for reaching the registry:

| Variable | How it works | Result |
| --- | --- | --- |
| `REGISTRY_URL` | Browser JS calls registry directly | CORS error — browser blocks cross-origin requests |
| `NGINX_PROXY_PASS_URL` | nginx in the UI container proxies to registry | No CORS — browser only talks to the UI on port 5051 |

**Always use `NGINX_PROXY_PASS_URL`** for a same-host setup. `REGISTRY_URL` requires
configuring CORS headers on the registry itself and is unnecessary complexity.

The value must use the Docker service name (`registry`), not the NAS IP, so the
container-to-container call stays on the internal Docker network:

```text
NGINX_PROXY_PASS_URL=http://registry:5000   ✓
NGINX_PROXY_PASS_URL=http://<nas-ip>:5050  ✗  (may not route correctly inside Docker)
```

---

## Quick API Browse (no UI needed)

Standard registry API endpoints work directly in a browser:

```text
http://<nas-ip>:5050/v2/_catalog              # list all repos
http://<nas-ip>:5050/v2/the-moment/tags/list  # list tags for an image
```

---

## Pulling Images on Another LAN Machine

The registry is HTTP (not HTTPS). Docker treats it as insecure by default and will refuse
to pull without explicit permission. One-time setup per machine:

### Linux

Edit or create `/etc/docker/daemon.json`:

```json
{
  "insecure-registries": ["<nas-ip>:5050"]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

### macOS (Docker Desktop)

Open Docker Desktop → Settings → Docker Engine tab. Add the key to the JSON config:

```json
{
  "insecure-registries": ["<nas-ip>:5050"]
}
```

Click **Apply & Restart**.

### Windows (Docker Desktop)

Same as macOS: Docker Desktop → Settings → Docker Engine, add the same JSON key, then
**Apply & Restart**.

### Windows (Docker Engine without Desktop)

Edit `C:\ProgramData\docker\config\daemon.json` (create it if missing):

```json
{
  "insecure-registries": ["<nas-ip>:5050"]
}
```

Restart the Docker service:

```text
net stop docker && net start docker
```

---

### Pulling and using the image

Once the insecure registry is configured, pull and run normally:

```bash
docker pull <nas-ip>:5050/the-moment:latest
```

In a `docker-compose.yml` on the target machine, replace the image line:

```yaml
services:
  the-moment:
    image: <nas-ip>:5050/the-moment:latest
    # ... rest of compose config unchanged ...
```

Then `docker compose up -d` — Docker pulls from the NAS registry instead of GHCR.

---

## Deleting Images via the UI

The UI delete button (trashcan) calls the registry `DELETE /v2/<name>/manifests/<digest>` API.
Two things must both be true for it to work:

**1. Delete must be enabled on the registry** — it is off by default.

```yaml
- REGISTRY_STORAGE_DELETE_ENABLED=true
```

**Gotcha:** the value must be the bare word `true`, not `[true]`. The bracket notation is
only for list-type headers (Allow-Origin, Allow-Methods, etc.). Using `[true]` passes the
literal string `[true]` to the registry, which it does not recognise as truthy — delete
silently stays disabled and the UI returns "Operation not supported".

**2. CORS headers must expose `Docker-Content-Digest`** — the UI reads the digest from the
response header to build the delete URL. Without `Access-Control-Expose-Headers` including
`Docker-Content-Digest`, the browser blocks the header and the UI has nothing to delete.

Both are included in the compose file above.

### Garbage collection after deletion

Deleting a manifest removes the reference but the blobs (layers) remain on disk until
garbage collection runs:

```bash
docker exec registry registry garbage-collect /etc/docker/registry/config.yml
```

Run this after bulk deletes to reclaim space. The registry must be running when you run it
(garbage collection is online-safe in registry:2).

---

## Jenkins Integration

Jenkins pushes to this registry after each successful build. The image tag follows the
build number: `<nas-ip>:5050/the-moment:<BUILD_NUMBER>` and `latest`.

The Jenkins controller and all agents must also have `<nas-ip>:5050` in their
insecure-registries list, or Docker push/pull steps will fail with a TLS error.

### Multi-arch images: `--platform` is required on both build agents

The pipeline builds `linux/arm64` on the ARM64 agent and `linux/amd64` on the Windows
agent, then combines them into a multi-arch manifest with `docker buildx imagetools create`.

The ARM64 build **must** include `--platform linux/arm64` explicitly:

```sh
docker build --platform linux/arm64 --target production \
  -t <nas-ip>:5050/the-moment:<TAG>-arm64 .
```

Without `--platform`, Docker builds for the native host but does not record the platform
metadata reliably enough for `buildx imagetools create` to identify it as `linux/arm64`.
The manifest ends up showing only `linux/amd64`. Adding the flag fixes it.
