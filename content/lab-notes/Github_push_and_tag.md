+++
date = '2026-05-16T21:24:05-03:00'
title = 'Github_push_and_tag'
tags = ["git"]
+++

## Option A — Squash all commits into one before pushing

```bash
git checkout --orphan release
git add -A
git commit -m "Beta_1.0.1"
git branch -D main
git branch -m main
git push origin main --force
```

This replaces the entire history with a single clean commit.

## Option B — GitHub Releases (tag-based, history stays)

If the goal is just to publish a versioned release ZIP for users to download (not hide commits), use GitHub Releases:

```bash
git tag v1.0.1
git push origin v1.0.1
```

Then on GitHub: Releases → Draft a new release → pick the tag → publish. GitHub auto-generates a source zip. This is the conventional open-source approach.

---

## Reference

`git checkout --orphan release`
Creates a new branch called release with no history — no parent commits. Existing files are kept in the working tree but the branch starts completely empty.

`git add -A`
Stages all current files (adds everything to the index ready to commit).

`git commit -m "Beta_1.0.1"`
Creates a single fresh commit with all your files and the message Beta_1.0.1. This is now the entire history — one commit, no WIP noise.

`git branch -D main`
Force-deletes the old main branch (which still has all the messy history). The -D is needed because release and main have diverged histories.

`git branch -m main`
Renames the current branch (release) to main. Now main points to your single clean commit.

`git push origin main --force`
Force-pushes the new clean main to GitHub, overwriting the remote's history. --force is required because the histories are incompatible — you're replacing, not extending.
