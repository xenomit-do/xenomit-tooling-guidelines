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
4. Pushes to the branch named in `targets.tsv` (for example `main`)

Target pushes use a dedicated credential—not the default `GITHUB_TOKEN`—so cross-repo writes are explicit and scoped. The workflow picks the token from the target repository’s **resource owner** (`owner` in `owner/repo`).

### Required secrets

Targets under the **`xenomit-do`** org use a fine-grained personal access token stored as the Actions secret **`XENOMIT_DO_SYNC_TOKEN`** on **this** repository. Create the token under an account that can push to those repositories, and limit it to the `xenomit-do` repos listed in `targets.tsv`.

Minimum permissions on the repositories that token will push to:

| Permission | Access |
|------------|--------|
| Contents | Read and write |
| Metadata | Read |

Add **`XENOMIT_DO_SYNC_TOKEN`** as a repository secret on `xenomit-do/xenomit-tooling-guidelines`. Before any authenticated push, the workflow exports that secret as `GH_TOKEN` and runs `gh auth setup-git`. If `targets.tsv` includes a repository under another resource owner, the workflow fails until a token mapping is added in `.github/workflows/sync-agent-guidelines.yml`. Targets with no file changes are synced without using a token.

**Branch protection:** each target branch must allow that token’s identity to push (or bypass rules where your policy requires it). Do not weaken protections in this repo to compensate; fix permissions on the targets or the PAT scopes instead.

Do not commit tokens or store them outside GitHub Actions secrets.
