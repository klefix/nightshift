# Nightshift v2 — Pivot Design

Date: 2026-03-11

## Context

The original Nightshift was a self-hosted Docker app: Next.js dashboard, SQLite DB, node-cron scheduler, Claude Agent SDK executor. Too much infrastructure for the core value prop.

Key realizations:
- GitHub Actions already provides sandboxed runners, cron scheduling, and secrets management
- `anthropics/claude-code-action` is an official GitHub Action that runs Claude Code
- GitHub Projects provides the cross-repo oversight originally planned for the dashboard
- Claude Code skills/plugins can handle setup and configuration interactively

## Decision

Nightshift becomes a **Claude Code plugin** that scaffolds GitHub Actions workflows to run AI agents on your repos overnight.

## What Nightshift Is

A Claude Code plugin you install once. It provides:

1. **Pre-built agent prompts** (markdown files) for common tasks
2. **GitHub Actions workflow templates** that run agents on triggers
3. **A `/nightshift` skill** for interactive setup in any repo
4. **GitHub workflow skills** for issues, PRs, wiki, project updates, ADRs, READMEs

## Agent Library

| Agent | Default Trigger | What It Does |
|-------|----------------|-------------|
| code-reviewer | On PR | Reviews code quality, leaves inline comments |
| security-auditor | Weekly cron | Scans for vulnerabilities, creates issues |
| dependency-updater | Weekly cron | Checks outdated deps, creates update PRs |
| issue-resolver | On label `nightshift` | Picks up labeled issues, creates PRs |
| dead-code-detector | Weekly cron | Finds unused exports/code, creates issues |
| readme-auditor | Weekly cron | Checks if docs match code, creates issues/PRs |

All agents are trigger-flexible: any agent can run on any trigger (cron, PR, label, manual dispatch). The user configures this during setup.

## Architecture

### Execution: GitHub Actions + Official Action

Each agent runs as a GitHub Actions workflow using `anthropics/claude-code-action@v1`:

```yaml
name: "Nightshift: Security Auditor"
on:
  schedule:
    - cron: '0 3 * * 0'
  workflow_dispatch: {}

permissions:
  contents: read
  issues: write
  pull-requests: write
  id-token: write

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: anthropics/claude-code-action@v1
        with:
          prompt: |
            Read and follow the agent instructions in .github/nightshift/agents/security-auditor.md
            Repository: ${{ github.repository }}
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

The prompt points Claude to the markdown file in the repo. Customization = editing the markdown. The workflow file rarely needs to change.

### Trigger Types

- **Cron**: `schedule` with cron expression (nightly, weekly, monthly)
- **PR**: `pull_request` types `[opened, synchronize]`
- **Label**: `issues` types `[labeled]` with `if:` condition
- **Manual**: `workflow_dispatch` (always included)

### Customization

Agent prompts are plain markdown files in `.github/nightshift/agents/`. Edit them directly — no config layers, no indirection.

### Dashboard: GitHub Itself

- **Issues** for findings and tasks
- **PRs** for code changes
- **GitHub Projects** for cross-repo oversight and tracking
- **Actions tab** for run history and logs

## Repo Structure (nightshift repo)

```
agents/                          # agent prompt library (source of truth)
  code-reviewer.md
  security-auditor.md
  dependency-updater.md
  issue-resolver.md
  dead-code-detector.md
  readme-auditor.md
workflows/                       # workflow templates
  nightshift-code-reviewer.yml
  nightshift-security-auditor.yml
  nightshift-dependency-updater.yml
  nightshift-issue-resolver.yml
  nightshift-dead-code-detector.yml
  nightshift-readme-auditor.yml
skills/                          # Claude Code plugin skills
  nightshift/                    # /nightshift interactive setup
  creating-issues/
  creating-pull-requests/
  writing-wiki-pages/
  writing-project-updates/
  writing-adrs/
  updating-readmes/
README.md
CLAUDE.md
```

## Installed Structure (target repo)

```
.github/
  nightshift/
    agents/
      code-reviewer.md
      security-auditor.md
      ...
  workflows/
    nightshift-code-reviewer.yml
    nightshift-security-auditor.yml
    ...
```

## Setup Experience

1. `claude install klefix/seville` — installs the plugin
2. `/nightshift` in any repo — interactive setup:
   - Pick agents from the library
   - Choose triggers and schedules per agent
   - Scaffolds `.github/nightshift/agents/` and `.github/workflows/`
   - Reminds to add `ANTHROPIC_API_KEY` to repo secrets
3. After setup, natural language for changes:
   - "Add the dead code detector, run monthly"
   - "Make the code reviewer check accessibility too"
   - "Disable the security auditor"

## What We Drop

- Next.js app, all UI code
- Database (SQLite, Drizzle ORM)
- Docker setup
- Orchestrator/scheduler (node-cron)
- Agent SDK direct usage
- API routes
- Authentication system

## What We Keep

- The 6 GitHub workflow skills (issues, PRs, wiki, etc.)
- CLAUDE.md workflow guidance
- The core proposition: **Nightshift improves your codebase overnight**
