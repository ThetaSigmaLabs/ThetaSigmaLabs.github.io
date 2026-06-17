+++
date = '2026-05-16T23:00:13-03:00'
title = 'Docker_dev_vscode_improvements'
tags = ["docker", "vscode"]
+++

`docker-compose.dev.yml` — added `build: context: .` so Compose can rebuild the image itself with `--build`, no separate `docker build` needed.

.vscode/tasks.json — 5 new tasks added:

| Task | What it does |
| --- | --- |
| Docker: Build Dev Image | Standalone build + prune dangling |
| Docker: Start Dev Stack | Start stack using existing local image |
| Docker: Build and Restart Dev Stack | Primary task — rebuild, restart, prune |
| Docker: Stop Dev Stack | Bring the whole dev stack down |
| Docker: Logs | Stream the-moment logs in a dedicated panel |

**Daily driver**: Terminal → Run Task → Docker: Build and Restart Dev Stack. This is the replacement for your manual docker build command — it rebuilds the image, prunes the dangling old one, and restarts the container in one step.

**Tip**: The Docker VSCode extension (`ms-azuretools.vscode-docker`) adds a visual Docker Explorer panel with compose group management and inline log viewing — worth installing as a complement to these tasks.
