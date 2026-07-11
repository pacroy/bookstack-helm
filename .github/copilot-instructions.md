# Copilot Instructions for gh-common-workflows

## Repository Purpose

This is a centralized GitHub Actions workflows repository that serves as a **source for syncing common workflows** to multiple target repositories. The goal is to maintain workflow configurations in one place and automatically propagate them to all dependent repositories.

## Architecture

### Workflow Structure

**Public Workflows** (distributed to target repos):
- `linter.yml` - Calls reusable `wf_linter.yml`
- `mdlink.yml` - Calls reusable `wf_mdlink.yml`

**Reusable Workflows** (internal reference, excluded from sync):
- `wf_linter.yml` - Implements Super-linter for code linting
- `wf_mdlink.yml` - Implements Markdown link checking

**Utility Workflows** (internal sync helpers, excluded from sync):
- `sync.yml` - Syncs `.github/` folder to target repositories via rsync
- `_sync_secrets.yml` - Syncs GitHub repository secrets across multiple repositories

### Sync Process

The sync system works by:
1. **Source repo** (this repository) contains all workflow configurations
2. **Target repos** include a copy of `sync.yml` to pull changes
3. `sync.yml` uses rsync to copy `.github/` contents while **excluding**:
   - Workflows starting with `_` (underscore)
   - Workflows starting with `wf_` (internal reusable)
   - Old `markdown-link-check.yml` and related directories
4. Syncs are triggered on PR to `main` or manual `workflow_dispatch`
5. Changes are auto-committed to target repos using git bot account

## Key Conventions

### Workflow Naming
- **Public workflows**: No prefix (e.g., `linter.yml`)
- **Reusable (internal)**: `wf_` prefix (e.g., `wf_linter.yml`)
- **Utility/Admin**: `_` prefix (e.g., `_sync_secrets.yml`)

This naming scheme ensures only public workflows are synced to target repositories.

### Environment Variables
- `SOURCE_REPO`: Full repository path (e.g., `pacroy/gh-common-workflows`)
- `SOURCE_REF`: Git reference for source (e.g., `v1`) - allows version pinning in target repos
- `REPO_LIST_REGEX`: When `true`, treat repository patterns as regex expressions
- Secret patterns use regex for flexible matching (e.g., `^SYNC_PAT$`)

### File Exclusion in rsync
Patterns are defined in `sync.yml` under the "Sync files" step:
```bash
--exclude="workflows/_*.yml" --exclude="workflows/wf_*.yml"
rm -f "target/${folder}/workflows/_*.yml"
rm -f "target/${folder}/workflows/wf_*.yml"
```

Always update both the rsync `--exclude` flags AND the explicit `rm` commands when adding new internal workflows.

## Testing Workflows

### Dry-run Workflows
Several workflows support `workflow_dispatch` with `dry_run` input:
- `_sync_secrets.yml` - Use `dry_run: true` to preview changes without applying them
- Test on a specific target repo regex pattern before rolling out

### Linting and Link Checking
These are automatically triggered on:
- Push to `main`
- Pull requests to `main`
- Manual `workflow_dispatch`

To test locally:
- **Linting**: Super-linter is configured in `wf_linter.yml` with `VALIDATE_ALL_CODEBASE: true`
- **Markdown links**: Configured via `.github/mdlink/mlc_config.json`

## GitHub Token & Permissions

When setting up sync in target repositories, use a Personal Access Token (`SYNC_PAT`) with:

**Classic token:**
- `repo` scope (full control of private repositories)
- `workflow` scope (update GitHub Action workflows)

**Fine-grained token:**
- Repository > Contents: Read and write
- Repository > Metadata: Read-only
- Repository > Secrets: Read and write
- Repository > Workflows: Read and write

## Configuration Files

- `.github/mdlink/mlc_config.json` - Markdown link checker configuration (retries, timeouts, headers)
- `.claude/settings.local.json` - Permissions for Claude sessions in this repo

## Making Changes

1. Create a branch from `main`
2. Update workflows (both public and internal)
3. Push and create a PR to `main`
4. Workflows auto-run on PR:
   - Linter checks code
   - Markdown link checker validates documentation
   - Sync workflow shows a preview of what will sync to target repos
5. Merge PR when ready
6. To release updates to target repos:
   - Create a git tag (e.g., `v1`) or update `SOURCE_REF` in target repos
   - Target repos will pull latest changes on their next sync trigger

## Key Dependencies

- **actions/checkout**: v7.0.0
- **actions/github-script**: v9.0.0
- **super-linter/super-linter**: v8.7.0
- **jpoehnelt/secrets-sync-action**: v1.10.0
- **gaurav-nelson/github-action-markdown-link-check**: 1.0.17
