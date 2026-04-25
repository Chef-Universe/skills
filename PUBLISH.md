# Publishing Chef-Universe/skills

This document is for the Chef Universe maintainer. End users never need to read it.

## Architecture

```
johnGRAMPUS/chefuniverse-web   (private)
└── skills-package/            ← source of truth, edit here
        │
        │  GitHub Action: .github/workflows/sync-skills.yml
        │  triggers on push to main touching skills-package/**
        ▼
Chef-Universe/skills            (public)
└── skills/, README.md, ...    ← what `npx skills add` reads
```

Edit only the source. The public repo is a downstream mirror — direct edits there will be overwritten on next sync.

## One-time setup

### 1. Create a Personal Access Token

GitHub → top-right avatar → **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.

| Field | Value |
|---|---|
| Token name | `chefuniverse-skills-sync` |
| Expiration | 1 year (or "No expiration" if you accept the risk) |
| Repository access | Only select repositories → **`Chef-Universe/skills`** |
| Permissions → Repository → **Contents** | **Read and write** |
| Permissions → Repository → Metadata | Read-only (auto) |

Click **Generate token**, copy the token string (`github_pat_...`). You'll only see it once.

### 2. Add as a secret on chefuniverse-web

`github.com/johnGRAMPUS/chefuniverse-web` → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**.

| Field | Value |
|---|---|
| Name | `SKILLS_REPO_TOKEN` |
| Secret | paste the PAT from step 1 |

### 3. First sync

`github.com/johnGRAMPUS/chefuniverse-web` → **Actions** tab → **Sync skills package → Chef-Universe/skills** workflow → **Run workflow** → branch `main` → **Run workflow**.

Watch the run. If green, open `github.com/Chef-Universe/skills` — it now has the package contents.

### 4. Verify install

From any machine with Node:

```bash
npx skills add Chef-Universe/skills
```

Should report 8 skills installed (read-bazaar, interpret-signals, simulate-buy, buy-ingredient, sell-ingredient, check-portfolio, read-leaderboard, claim-rewards).

## Ongoing edits

After setup, you never touch the public repo manually. Workflow:

1. Edit any file under `skills-package/` in this private repo
2. Commit + push to main (or merge a PR)
3. Action triggers automatically, syncs to public repo
4. End users get the update on their next `npx skills add Chef-Universe/skills`

## Manual sync

If you want to force a sync without changing files (e.g. after a workflow tweak):

`github.com/johnGRAMPUS/chefuniverse-web` → Actions → Sync skills package → **Run workflow**.

## Token rotation

If the PAT expires or leaks:

1. Revoke at github.com → Settings → Developer settings → PAT → Revoke
2. Generate new PAT (same scope as step 1 above)
3. Update `SKILLS_REPO_TOKEN` secret on chefuniverse-web
4. Trigger one manual sync to verify

## Troubleshooting

**Action fails with `403 Forbidden` on push:**
- PAT doesn't have Contents: Read and write on Chef-Universe/skills
- Or PAT was scoped to wrong repository

**Action runs green but public repo unchanged:**
- Means git diff was empty — no actual changes since last sync
- Edit a skills-package/ file or trigger manually

**`npx skills add` says "package not found":**
- Public repo not yet created, or named differently
- Run is on wrong branch (the CLI reads the default branch — `main` in our case)
