+++
date = '2026-05-15T01:26:12-03:00'
title = 'Working_with_claude_and_vscode'
tags = []
+++

## User scope

 `~/.claude/settings.json` (user scope — applies to **all** projects)

 `<project_folder>/.claude/settings.local.json` (user scope — applies only to the project_folder)

## Why it's there

It's an explicit override you (or `/model`) set at some point — it is not a default. Without that line, Claude Code picks a model based on your plan/account. With it, every session in every project starts on Opus.

`"opus"` is a **family alias**, not a pinned version. It always resolves to the newest Opus available to you — today that's Opus 4.7 (c`laude-opus-4-7`). So you automatically move up when a new Opus ships, without editing this file. (You **pin** a version instead — **see below**.)

## What the values can be

| Value form     | Example                         | Behavior                         |
|----------------|---------------------------------|----------------------------------|
| Family alias   | `"opus"`, `"sonnet"`, `"haiku"` | Always the latest in that family |
| Version prefix | `"opus-4-7"`                    | Pins to that version line        |
| Full model ID  | `"claude-sonnet-4-6"`           | Exact model, no auto-upgrade     |

The three families (current generation, Claude 4.X):

- **Opus** — `claude-opus-4-7` — most capable, what you're on now
- **Sonnet** — `claude-sonnet-4-6` — balanced speed/capability
- **Haiku** — `claude-haiku-4-5-20251001` — fastest, cheapest, lightest
Separately: Fast mode (`/fast`) keeps you on Opus but with faster output — it does not downgrade to a smaller model. It's a different axis from model selection.

## How to manage it

- `/model` — interactive picker, the normal way to switch. Recommended.
- `/config` — general settings UI; model is one of the simple settings it exposes.
- `claude --model sonnet` — per-session override from the CLI, ignores the settings.json value for that run only.
- **Edit** `~/.claude/settings.json` — what's there now; this is the user (global, all-projects) scope.
**Precedence** (later wins): user (`~/.claude/settings.json`) → project (`.claude/settings.json`) → local (`.claude/settings.local.json`) → --model flag → managed/enterprise policy. A .claude/settings.json in a specific repo can override your global Opus default for just that project.

To **remove** the global pin and fall back to the account default, just delete the `"model": "opus"` line (mind the trailing comma on the line above it).
