# Branching Strategy — geekswagg/openclaw

## Overview

```
openclaw/openclaw (upstream)
       │
       │  fetch + merge (periodic)
       ▼
   main ──────────────────────── mirrors upstream (never commit custom work here)
       │
       │  merge (upstream flows down)
       ▼
   develop ───────────────────── integration branch (all custom work lands here)
       │        ▲
       │        │ merge feature branches
       │        │
       │    feature/*  ────────── individual features
       │
       │  promote (tag + merge when validated)
       ▼
   release ───────────────────── what BOTH machines run (stable)
       │
   upstream-pr/* ─────────────── clean branches off main for PRs to openclaw/openclaw
```

## Branches

### `main` — Upstream Mirror
- **Rule:** Only updated via `git fetch upstream && git merge upstream/main`
- **Never** commit custom work here
- Base for `upstream-pr/*` branches

### `develop` — Integration
- All custom features merge here first
- Periodically merges `main` to pick up upstream changes
- This is where we innovate freely
- Deploy to **alfred-ar** (Windows) first as canary

### `feature/*` — Individual Features
- Branch from `develop`, merge back to `develop`
- Examples: `feature/native-oidc-auth`, `feature/whatsapp-allowlist-default`

### `release` — Stable Deployment
- Both machines (alfred-ar + alfred-prime) track this branch
- Only gets code that's been validated on `develop`
- Tagged with versions: `alfred-v2026.3.15`, etc.

### `upstream-pr/*` — Contributing Back
- Branch from `main` (clean upstream base)
- Cherry-pick or rewrite the feature cleanly
- Submit PR to `openclaw/openclaw`
- Once merged upstream, flows back: upstream → `main` → `develop` → `release`

## Workflows

### Adding a Custom Feature
```bash
git checkout develop
git checkout -b feature/my-feature
# ... make changes ...
git commit -m "feat: my feature"
git push origin feature/my-feature
# merge to develop (PR or direct)
git checkout develop
git merge feature/my-feature
git push origin develop
```

### Syncing Upstream Changes
```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

git checkout develop
git merge main
# resolve any conflicts
git push origin develop
```

### Promoting to Release
```bash
# After validating develop on alfred-ar
git checkout release
git merge develop
git tag alfred-v2026.3.15
git push origin release --tags
```

### Deploying on a Machine
```bash
cd ~/openclaw  # or C:\dev\openclaw
git checkout release
git pull origin release
pnpm install
openclaw gateway restart
```

### Contributing Upstream
```bash
git checkout main
git checkout -b upstream-pr/my-feature
# cherry-pick or rewrite cleanly from develop
git cherry-pick <commit>
git push origin upstream-pr/my-feature
# create PR to openclaw/openclaw from this branch
```

## Machine Differences
Machine-specific settings belong in **config** (`~/.openclaw/openclaw.json`), not in code branches.
Both machines run identical code from the `release` branch.

## Remotes
```
origin   → geekswagg/openclaw (our fork)
upstream → openclaw/openclaw  (upstream)
```
