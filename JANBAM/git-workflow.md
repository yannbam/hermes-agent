# Git Workflow & Branch Organization

This fork of [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) is owned by **janbam** (jan). The autonomous hermes agent runs under the **janbambot** GitHub account as a collaborator with restricted permissions.

---

## Branches

| Branch | Protected | Who pushes | Purpose |
|--------|-----------|------------|---------|
| `main` | ✅ Yes (janbam only, PR approval required) | janbam | Curated mainline. Jan's feature work merges here. |
| `hermes-dev` | ❌ No (force-push safe) | janbambot | Hermes autonomous development sandbox. Disposable. |
| `upstream-release` | ❌ No | janbam | Read-only mirror of upstream's latest release tag. |

**Invariant**: `main` is never polluted by autonomous work. `hermes-dev` can be reset to `main` at any time without collateral damage.

---

## Tags

### Upstream release tags (untouched)
Upstream uses dated version tags: `v2026.4.23`, `v2026.4.16`, etc. These are **never created or modified** by this fork.

### Fork dev-stable tags (separate namespace)
To mark tested dev commits for the stable runtime, we use a separate versioning scheme:

```
dev-stable-v0.1.0
dev-stable-v0.2.0
...
```

These tags are:
- Created on commits in the `hermes-dev` branch that hermes has verified as stable
- Completely independent from upstream version numbers
- Used by the stable clone to `git checkout` the exact commit to run

**Version bump convention**: minor (dev-stable-v0.X.0) for promotion. Patch (dev-stable-v0.X.Y) if a hotfix lands before the next planned promotion.

---

## Three Clones

```
/home/jan/src/hermes-agent/                 jan       main         Jan's dev workspace (codex/pi, PRM)
/home/hermes/workspace/hermes-dev/           hermes    hermes-dev   Hermes autonomous sandbox
~/.local/hermes-dev-stable/                 hermes    (detached)   Hermes runtime — checked out to a dev-stable-v* tag
```

### Clone 1 — Jan's dev workspace
- **Path**: `/home/jan/src/hermes-agent/` (already exists)
- **Branch**: `main`
- **Install**: Jan's editable venv at `~/src/hermes-agent/venv/`
- **Workflow**: Standard PRM (feature branch → PR to main → merge → pull back)
- Jan does **not** depend on a stable hermes install. The editable install tracks whatever `main` HEAD is.

### Clone 2 — Hermes dev sandbox
- **Path**: `/home/hermes/workspace/hermes-dev/`
- **Branch**: `hermes-dev`
- **Install**: Hermes' editable venv at `/home/hermes/venv-dev/` → `hermes-dev` executable
- **Workflow**: Feature branches + PRs into `hermes-dev` are the default. Direct commits to `hermes-dev` are allowed only for trivial changes (typos, formatting, small fixes).
- **Access**: janbambot pushes here. jan can read/fix via group permissions.
- **Disposable**: If hermes fucks up `hermes-dev`, checkout the last good commit and branch from there:
  ```bash
  cd /home/hermes/workspace/hermes-dev
  git fetch origin main
  git checkout origin/main           # or any known-good commit
  git branch -f hermes-dev           # move the branch pointer here
  git checkout hermes-dev
  git push origin +hermes-dev
  ```

### Clone 3 — Hermes stable runtime
- **Path**: `~/.local/hermes-dev-stable/`
- **Branch**: None (detached HEAD or checked out to a `dev-stable-v*` tag)
- **Install**: Hermes' editable venv at `/home/hermes/venv-stable/` → `hermes` executable
- **No commits, no merges, no edits happen here.** The only operation is `git checkout <newer-dev-stable-tag>`.
- `.pyc` / `__pycache__` is gitignored and ignored in spirit — write noise doesn't matter.

> **⚠️ Unresolved**: restarting the stable hermes gateway after promotion is not yet automated. For now, manual restart is required. Automating this is deferred.

---

## GitHub Accounts & Permissions

| Account | Role | Can push to |
|---------|------|-------------|
| [janbam](https://github.com/janbam) | Owner | `main` (protected), `hermes-dev`, tags |
| [janbambot](https://github.com/janbambot) | Collaborator | `hermes-dev`, `dev-stable-v*` tags |

- `main` branch protection: PRs require janbam approval, janbam merges
- `hermes-dev` branch: janbambot can force-push (sandbox semantics)
- Hermes clones the repo via `git@github-bot.com` — an SSH alias using the janbambot key

---

## Promotion Path: Dev → Stable

Hermes works on `hermes-dev`, tests, then promotes to stable without touching `main`:

```
1. Hermes dev clone (on hermes-dev branch):
   $ hack hack hack
   $ git add ...
   $ git commit -m "feat: ..."
   $ # test locally via hermes-dev, pytest, smoke tests
   $ git push origin hermes-dev

2. Tag the tested commit:
   $ git tag -a dev-stable-v0.1.0 -m "hermes: promote tested dev commit"
   $ git push origin dev-stable-v0.1.0

3. Hermes stable clone:
   $ git fetch origin --tags
   $ git checkout dev-stable-v0.1.0
   $ # restart hermes gateway / CLI to pick up changes
```

If hermes promotes a bad commit by mistake: re-tag an earlier commit and check it out in stable. No history rewrite needed.

---

## Jan Incorporating Hermes' Work Into Main

When jan wants to pull hermes' dev contributions into main:

```bash
cd /home/jan/src/hermes-agent
git checkout main
git merge origin/hermes-dev          # or cherry-pick specific commits
# resolve conflicts, review
git push origin main
```

Hermes never touches `main`. The merge direction is always `hermes-dev → main` (by jan's hand).

---

## Upstream Sync

When NousResearch tags a new release:

```bash
cd /home/jan/src/hermes-agent
git fetch upstream --tags

# Update the upstream-release mirror
LATEST=$(git tag --sort=-v:refname --list 'v*' | head -1)
git branch -f upstream-release "$LATEST"
git push origin upstream-release --force-with-lease

# Merge into main
git checkout main
git merge upstream-release
# resolve conflicts, rebuild, test

# Optionally: reset hermes-dev branch to updated main
git checkout hermes-dev
git merge main
git push origin hermes-dev

git push origin main
```

---

## Hermes Executables

The hermes user has two entry points:

| Command | Installed from | Source clone | Purpose |
|---------|---------------|--------------|---------|
| `hermes` | `/home/hermes/venv-stable/` | `~/.local/hermes-dev-stable/` | Stable runtime (gateway, daily CLI) |
| `hermes-dev` | `/home/hermes/venv-dev/` | `/home/hermes/workspace/hermes-dev/` | Development & testing |

Both are editable installs. The stable hermes can spawn `hermes-dev` for testing without affecting its own runtime.

---

## File Permissions

```
/home/jan/src/hermes-agent/                   jan:jan    775
/home/hermes/workspace/hermes-dev/             hermes:jan 775   (hermes writes, jan reads/fixes)
~/.local/hermes-dev-stable/                   hermes:jan 755   (hermes reads, jan writes)
```

The stable clone at 755 prevents hermes from accidentally writing source files there. `__pycache__/` writes are harmless (gitignored) but the tighter permissions are belt-and-suspenders.

---

## Disaster Recovery

| Scenario | Recovery |
|----------|----------|
| Hermes corrupts `hermes-dev` branch | `git checkout <good-commit> && git branch -f hermes-dev && git checkout hermes-dev && git push origin +hermes-dev` |
| Hermes promotes bad `dev-stable-v*` tag | `git checkout <previous-good-tag>` in stable clone |
| Hermes somehow corrupts stable clone | Delete and re-clone at the correct tag |
| Jan's `main` has merge conflict with upstream | Resolve manually, push (janbam only) |
| Upstream rebases / force-pushes | Unlikely (upstream is NousResearch, stable tags) — manually re-sync |
