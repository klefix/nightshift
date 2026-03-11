# Nightshift — Project Instructions

## What This Is

Nightshift is a Claude Code plugin that scaffolds AI agents (GitHub Actions workflows + prompts) into repositories. Agents run Claude Code via the official `anthropics/claude-code-action` GitHub Action.

## Repo Structure

```
agents/          # Agent prompt library (source markdown files)
workflows/       # GitHub Actions workflow templates
skills/          # Claude Code plugin skills
  nightshift/    # /nightshift interactive setup skill
  creating-issues/
  creating-pull-requests/
  writing-wiki-pages/
  writing-project-updates/
  writing-adrs/
  updating-readmes/
docs/plans/      # Design documents
```

## Key Concepts

- **Agents** are markdown files that define what Claude should do (review code, audit security, etc.)
- **Workflows** are GitHub Actions YAML files that define when agents run (cron, PR, label, manual)
- **The `/nightshift` skill** scaffolds agents + workflows into a target repo interactively
- Agent prompts point Claude to the `.md` file in the target repo: `Read and follow the agent instructions in .github/nightshift/agents/<name>.md`

## Conventions

- Agent markdown files go in `agents/` at repo root (source of truth)
- When installed in a target repo, agents go in `.github/nightshift/agents/`
- Workflow templates go in `workflows/` at repo root
- When installed, workflows go in `.github/workflows/nightshift-*.yml`
- All workflows include `workflow_dispatch` for manual triggering
- Skills follow the `SKILL.md` convention in `skills/<name>/SKILL.md`

## GitHub Project Board

Project #1 (owner: klefix). Status field options:

| Status | Option ID |
|--------|-----------|
| Backlog | `d2711ddc` |
| Todo | `c20daeea` |
| In Progress | `6385cc7c` |
| Done | `993491db` |

Status field ID: `PVTSSF_lAHOAC0sG84BQ_oyzg-9CiQ`

## Workflow — Nightshift Skills

Use these skills proactively at the right workflow moments:

### When starting work on an issue

1. Transition the issue to **In Progress** on the project board
2. Create a feature branch (`klefix/<descriptive-name>`)

### During implementation

- **If diverging from issue spec:** use `nightshift:creating-issues` to comment explaining why
- **If discovering tech debt/issues:** use `nightshift:creating-issues` to comment (ask before creating follow-up issues)
- **If making architectural decisions:** use `nightshift:writing-adrs`

### When completing work

1. Use `nightshift:creating-pull-requests` for PRs — use `Closes #N` to auto-close issues
2. After merging: use `nightshift:writing-project-updates`
3. If structure changed: use `nightshift:updating-readmes`
4. If documentation needed: consider `nightshift:writing-wiki-pages`
