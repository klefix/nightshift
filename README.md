# Nightshift

Your codebase improves overnight. Nightshift is a Claude Code plugin that sets up AI agents to work on your GitHub repositories via GitHub Actions.

## What It Does

Nightshift provides pre-built agents that run as scheduled GitHub Actions workflows using the official [Claude Code Action](https://github.com/anthropics/claude-code-action). Agents can:

- **Review PRs** — automated code review on every pull request
- **Audit security** — weekly scans for vulnerabilities and OWASP top 10
- **Update dependencies** — check for outdated packages, create update PRs
- **Resolve issues** — pick up labeled issues and create fix PRs
- **Detect dead code** — find unused exports, orphaned files, dead dependencies
- **Audit READMEs** — verify documentation matches actual codebase

## Quick Start

### 1. Install the plugin

```bash
/plugin marketplace add klefix/nightshift
/plugin install nightshift@nightshift
```

### 2. Set up your repo

```bash
cd your-project
claude
> /nightshift:setup-project
```

Claude creates a GitHub Project board, enables the wiki, adds labels, and optionally protects `main`.

### 3. Set up agents

```bash
> /nightshift:setup-agents
```

Claude walks you through picking agents, configuring triggers (PR, cron, label), and scaffolds everything into your repo.

### 4. Add your API key

Go to your repo's **Settings > Secrets and variables > Actions** and add `ANTHROPIC_API_KEY`.

### 5. Done

Agents run automatically on their configured triggers. Customize any agent by editing its markdown file in `.github/nightshift/agents/`.

## What Gets Installed

```
.github/
  nightshift/
    agents/
      code-reviewer.md          # agent prompt (edit to customize)
      security-auditor.md
      ...
  workflows/
    nightshift-code-reviewer.yml    # GitHub Actions workflow
    nightshift-security-auditor.yml
    ...
```

- **Agent files** (`.md`) define what the agent does — edit them to customize behavior
- **Workflow files** (`.yml`) define when the agent runs — triggers, schedule, permissions

## Agents

| Agent | Default Trigger | Description |
|-------|----------------|-------------|
| `code-reviewer` | On PR | Reviews code quality, leaves inline comments |
| `security-auditor` | Weekly (Sun 3am UTC) | Scans for vulnerabilities, creates issues |
| `dependency-updater` | Weekly (Mon 3am UTC) | Checks outdated deps, creates update PRs |
| `issue-resolver` | On `nightshift` label + nightly | Picks up labeled issues, creates fix PRs |
| `dead-code-detector` | Weekly (Sun 3am UTC) | Finds unused code, creates issues |
| `readme-auditor` | Weekly (Sun 3am UTC) | Checks README accuracy, creates issues/PRs |

## Customization

**Change what an agent does:** Edit the markdown file in `.github/nightshift/agents/`. The entire prompt is there — add constraints, change focus areas, adjust reporting style.

**Change when an agent runs:** Edit the `on:` section in `.github/workflows/nightshift-*.yml`. Any GitHub Actions trigger works — cron, PR events, labels, manual dispatch.

**Add/remove agents:** Just ask Claude in your repo: "Add the dead code detector, run it monthly" or "Remove the security auditor". Or run `/nightshift:setup-agents` again.

## Plugin Skills

When installed, Nightshift also provides these Claude Code skills for your workflow:

| Skill | Purpose |
|-------|---------|
| `/nightshift:setup-project` | Repo infrastructure setup (project board, wiki, labels, branch protection) |
| `/nightshift:setup-agents` | Interactive agent setup |
| Creating issues | Structured GitHub issue creation |
| Creating pull requests | Well-formatted PR descriptions |
| Writing wiki pages | GitHub Wiki documentation |
| Writing project updates | GitHub Projects updates |
| Writing ADRs | Architecture Decision Records |
| Updating READMEs | README maintenance |

## How It Works

Nightshift uses the official [`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action) GitHub Action. Each agent is a workflow that:

1. Checks out your repo
2. Runs Claude Code with the agent's prompt
3. Claude reads the prompt file, analyzes the repo, and takes action (comments, issues, PRs)

Everything runs on GitHub's infrastructure — no self-hosting, no Docker, no database.
