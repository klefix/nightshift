---
description: Set up Nightshift agents in a repository — scaffolds agent prompts and GitHub Actions workflows for automated code review, security audits, dependency updates, and more
---

# Setup Agents

## Overview

Interactively set up Nightshift agents in the current repository. Nightshift runs Claude Code agents on your codebase via GitHub Actions — reviewing PRs, auditing security, updating dependencies, resolving issues, and more.

## When to Use

- First-time setup of Nightshift in a repo
- Adding a new agent to an existing Nightshift setup
- Removing or reconfiguring agents

## Available Agents

| Agent | Description | Suggested Trigger |
|-------|-------------|-------------------|
| `code-reviewer` | Reviews PRs for bugs, security, quality, performance | On PR |
| `security-auditor` | Scans for OWASP top 10, CVEs, hardcoded secrets | Weekly cron |
| `dependency-updater` | Checks outdated deps, creates update PRs | Weekly cron |
| `issue-resolver` | Picks up `nightshift`-labeled issues, creates fix PRs | On label + nightly cron |
| `dead-code-detector` | Finds unused exports, orphaned files, dead deps | Weekly cron |
| `readme-auditor` | Checks README accuracy against actual codebase | Weekly cron |

## Setup Process

### 1. Check existing setup

```bash
ls .github/nightshift/agents/ 2>/dev/null
ls .github/workflows/nightshift-* 2>/dev/null
```

If agents already exist, show what's installed and ask what to change.

### 2. Ask which agents to install

Present the agent table above. Ask the user to pick which agents they want. Default recommendation: start with `code-reviewer` and `issue-resolver`.

### 3. Configure triggers for each agent

For each selected agent, ask about triggers. Present the suggested trigger as the default:

- **On PR**: `pull_request` types `[opened, synchronize, ready_for_review, reopened]`
- **On label**: `issues` types `[labeled]` — ask which label (default: `nightshift`)
- **Cron**: ask for frequency — nightly (`0 3 * * *`), weekly (`0 3 * * 0`), monthly (`0 3 1 * *`)
- **Manual only**: just `workflow_dispatch`

All workflows always include `workflow_dispatch` for manual triggering.

### 4. Scaffold files

Create the directory structure:
```
.github/
  nightshift/
    agents/
      <selected-agent>.md    # copy from nightshift plugin's agents/ directory
  workflows/
    nightshift-<agent>.yml   # generate from template with chosen triggers
```

For agent markdown files: read each selected agent's source file from this plugin's `agents/` directory using bash:

```bash
cat "$CLAUDE_PLUGIN_ROOT/agents/<agent-name>.md"
```

Copy the content into the target repo at `.github/nightshift/agents/<agent-name>.md`.

If `$CLAUDE_PLUGIN_ROOT` is not set, read the files from the `agents/` directory relative to this skill file (two levels up: `../../agents/`).

For workflow files: generate them using the template pattern below, substituting the agent name and triggers.

### Workflow Template

```yaml
name: "Nightshift: <Agent Display Name>"

on:
  <triggers based on user choice>
  workflow_dispatch: {}

permissions:
  contents: <read or write depending on agent>
  pull-requests: write
  issues: write
  id-token: write

jobs:
  run:
    <if condition for label triggers>
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Read and follow the agent instructions in .github/nightshift/agents/<agent>.md

            Repository: ${{ github.repository }}
            <PR Number if PR-triggered: ${{ github.event.pull_request.number }}>
```

Agents that create PRs or push commits need `contents: write`. Read-only agents use `contents: read`.

### 5. Remind about secrets

After scaffolding, remind the user:

> Don't forget to add `ANTHROPIC_API_KEY` to your repository secrets:
> **Settings > Secrets and variables > Actions > New repository secret**

### 6. Configure plugin for CI agents

Scaffold `.claude/settings.json` in the target repo so GitHub Actions agents can use nightshift skills:

```bash
mkdir -p .claude
```

If `.claude/settings.json` exists, merge into it. Otherwise create:

```json
{
  "extraKnownMarketplaces": {
    "nightshift": {
      "source": { "source": "github", "repo": "klefix/nightshift" }
    }
  },
  "enabledPlugins": {
    "nightshift@nightshift": true
  }
}
```

Ask the user if they want to add this configuration. It enables the nightshift skills (creating issues, PRs, wiki pages, etc.) for agents running in GitHub Actions.

### 7. Commit

Stage and commit all new files with message: `feat: add nightshift agents (<list of agents>)`

## After Setup

The user can customize agents by editing the markdown files directly. No need to re-run `/nightshift` — just tell Claude what to change in natural language.
