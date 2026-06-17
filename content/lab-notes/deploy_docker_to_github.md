+++
date = '2026-05-16T22:19:21-03:00'
draft = true
title = 'Deploy_docker_to_github'
tags = []
+++

## Complete Testing & Deployment Guide

### Phase 1 — Local Docker build test (Mac)

`cd ~/code/the-moment`

#### Build the image locally

`docker build -t the-moment:local .`

#### Spin up the full stack using the dev override

`THE_MOMENT_PORT=5002 docker compose -f` `docker-compose.yml -f docker-compose.dev.yml up -d`

#### Watch logs

`docker compose logs -f the-moment`

#### Verify health

`curl http://localhost:5002/api/status`

#### Tear down when done

`docker compose down`

#### Test the golden path

* add a printer,
* load spools,
* check NFC tab,
* generate a tag .bin.

## Phase 2 — Push to GitHub & trigger image build

### Commit the fixes

```zsh
git add docker-compose.yml docker-compose.dev.yml .github/workflows/docker-build.yml README.md CONTRIBUTING.md
git commit -m "fix: correct GHCR image name to ThetaSigmaLabs, fix latest tag on release, rename dev override"
```

### Push main

`git push origin main`

### Tag to trigger the Docker build + release workflows

`git tag v1.0.1`
`git push origin v1.0.1`

Monitor the build at [https://github.com/ThetaSigmaLabs/the-moment/actions](https://github.com/ThetaSigmaLabs/the-moment/actions).

Wait for both workflows to go green:

Docker Build and Push → publishes ghcr.io/thetasigmalabs/the-moment:v1.0.1 and ghcr.io/thetasigmalabs/the-moment:latest
Release → creates GitHub release with binaries
Phase 3 — End-user pull test (simulate Odroid)
Do this on any machine that isn't your dev Mac (or a clean directory with no local image):

mkdir /tmp/the-moment-test && cd /tmp/the-moment-test

### Download ONLY the docker-compose.yml — this is what an end user does

`curl -O https://raw.githubusercontent.com/ThetaSigmaLabs/the-moment/main/docker-compose.yml`

### Pull and start — must NOT have the-moment:local cached

`docker compose pull`
`docker compose up -d`

### Check health

`docker compose ps`
`curl http://localhost:5000/api/status`

### Tear down

`docker compose down -v`

## Phase 4 — Odroid N2+ deploy

SSH to the Odroid and run:

### Install Docker if not present

```zsh
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

### re-login

### Deploy

```zsh
mkdir ~/the-moment && cd ~/the-moment
curl -O https://raw.githubusercontent.com/ThetaSigmaLabs/the-moment/main/docker-compose.yml
```

## Set timezone in docker-compose.yml

## Edit TZ: GMT0

```zsh
docker compose pull
docker compose up -d
```

## Verify

```zsh
docker compose ps
curl http://localhost:5000/api/status
```

Access at [http://\<odroid-ip\>:5000](http://<odroid-ip>:5000).

Future updates on Odroid:

```zsh
cd ~/the-moment
docker compose pull && docker compose up -d
```

## Checklist summary

* [ ] `docker build` succeeds locally with no errors
* [ ] `docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d` runs the local image at port 5002
* [ ] `/api/status` returns 200 with correct version
* [ ] NFC tab loads, spool list populates from Spoolman
* [ ] Tag v`1.0.1` pushed → GitHub Actions green
* [ ] `ghcr.io/thetasigmalabs/the-moment:latest` visible in GitHub Packages
* [ ] Clean-directory `docker compose pull && up` works with no local image cached
* [ ] Same test passes on Odroid arm64
