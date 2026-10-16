# Xenomit Tooling Guidelines

Shared AI agent instructions and automation for Xenomit repositories.

## Source of truth

This repository is the **canonical source** for root `AGENTS.md` and `CLAUDE.md` used across the Xenomit workspace. Edit those files here first; downstream repos receive copies via the sync workflow.

`targets.tsv` lists each `owner/repo` and branch to update (tab-separated). The guidelines repo itself is not listed.

## Sync workflow

On every push to **`main`** (including merges), and on manual **workflow_dispatch**, the [Sync agent guidelines](.github/workflows/sync-agent-guidelines.yml) workflow:

1. Reads `targets.tsv`
2. Copies `AGENTS.md` and `CLAUDE.md` to each target repository root
3. Commits only when content changed, with message suffix **`[skip ci]`**
4. Pushes to the branch named in `targets.tsv` (for example `main`, or `master` for `ykaydogdu/ruffle`)

Target pushes use a dedicated credential—not the default `GITHUB_TOKEN`—so cross-repo writes are explicit and scoped.

### Required secret: `GUIDELINES_SYNC_TOKEN`

A repository **owner** must create a **fine-grained personal access token** with minimum scope:

| Permission | Access |
|------------|--------|
| Contents | Read and write on every repository listed in `targets.tsv` |
| Metadata | Read on those repositories |

Add the token as an Actions secret named **`GUIDELINES_SYNC_TOKEN`** on this repository. The workflow sets it as `GH_TOKEN` and runs `gh auth setup-git` for authenticated pushes.

**Branch protection:** each target branch must allow this token’s identity to push (or bypass rules where your policy requires it). Do not weaken protections in this repo to compensate; fix permissions on the targets or the PAT scopes instead.

Do not commit the token or store it outside GitHub Actions secrets.
