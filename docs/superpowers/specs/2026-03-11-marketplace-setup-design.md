# Marketplace Setup Design

## Problem

The nightshift repo contains skills, agents, and workflows but lacks the Claude Code marketplace structure (`.claude-plugin/marketplace.json`, `plugin.json`). It can't be installed via `/plugin marketplace add`.

## Decision

Single plugin inside a marketplace. The repo is both the marketplace catalog and the plugin source.

## Structure

```
.claude-plugin/
  marketplace.json              # marketplace catalog

plugins/
  nightshift/
    .claude-plugin/
      plugin.json               # plugin manifest
    skills/                     # moved from root
      nightshift/SKILL.md
      creating-issues/SKILL.md
      creating-pull-requests/SKILL.md
      writing-wiki-pages/SKILL.md
      writing-project-updates/SKILL.md
      writing-adrs/SKILL.md
      updating-readmes/SKILL.md
    agents/                     # moved from root
      code-reviewer.md
      security-auditor.md
      dependency-updater.md
      issue-resolver.md
      dead-code-detector.md
      readme-auditor.md
    workflows/                  # moved from root
      nightshift-code-reviewer.yml
      nightshift-security-auditor.yml
      nightshift-dependency-updater.yml
      nightshift-issue-resolver.yml
      nightshift-dead-code-detector.yml
      nightshift-readme-auditor.yml

docs/                           # stays at root (not plugin content)
CLAUDE.md                       # stays at root
README.md                       # stays at root
```

## Marketplace catalog (`.claude-plugin/marketplace.json`)

```json
{
  "name": "nightshift",
  "owner": {
    "name": "klefix"
  },
  "metadata": {
    "description": "AI agents that work your GitHub repo overnight"
  },
  "plugins": [
    {
      "name": "nightshift",
      "source": "./plugins/nightshift",
      "description": "AI agents that work your GitHub repo overnight — code review, security audits, dependency updates, and more",
      "category": "automation",
      "tags": ["github-actions", "code-review", "security", "agents"]
    }
  ]
}
```

## Plugin manifest (`plugins/nightshift/.claude-plugin/plugin.json`)

```json
{
  "name": "nightshift",
  "description": "AI agents that work your GitHub repo overnight — code review, security audits, dependency updates, and more",
  "version": "1.0.0",
  "author": {
    "name": "klefix"
  },
  "repository": "https://github.com/klefix/nightshift"
}
```

## Skill updates

### `/nightshift` setup skill

The setup skill must read agent template files from within the installed plugin. Since plugins are cached, the skill needs to reference files relative to its own location.

**Agent file access:** Update the SKILL.md to instruct Claude to read agent files via bash using `$CLAUDE_PLUGIN_ROOT/agents/<name>.md`. This env var is set by Claude Code for installed plugins. If it turns out not to be available during skill execution, fall back to embedding agent descriptions inline in the SKILL.md.

**Settings scaffolding:** Add a new step to scaffold `.claude/settings.json` in the target repo (merge into existing if present):

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

### Other skills (no changes needed)

The 6 other skills (`creating-issues`, `creating-pull-requests`, `writing-wiki-pages`, `writing-project-updates`, `writing-adrs`, `updating-readmes`) are self-contained prompt instructions that operate on the target repo via `gh` CLI and git. They do not reference plugin file paths and need no updates beyond being moved into the plugin directory.

## Scope exclusions

- `docs/` stays at root — it's repo documentation for nightshift development, not plugin content. No skill references it.
- `CLAUDE.md` stays at root — it's development instructions for this repo, not distributed to end users.

## Validation

After restructuring, run `claude plugin validate .` or `/plugin validate .` to verify the marketplace and plugin manifests are correct.

## Installation flow

```
/plugin marketplace add klefix/nightshift
/plugin install nightshift@nightshift
/nightshift  # interactive setup in target repo
```

## CLAUDE.md updates

Update repo structure section and conventions to reflect new paths:
- Source agents: `plugins/nightshift/agents/`
- Source workflows: `plugins/nightshift/workflows/`
- Source skills: `plugins/nightshift/skills/`
