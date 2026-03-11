# Marketplace Setup Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the nightshift repo into a valid Claude Code plugin marketplace so it can be installed via `/plugin marketplace add`.

**Architecture:** The repo becomes both the marketplace (`.claude-plugin/marketplace.json` at root) and contains a single plugin (`plugins/nightshift/` with its own `.claude-plugin/plugin.json`). All skills, agents, and workflows move into the plugin directory.

**Tech Stack:** Claude Code plugin system, JSON manifests, git

**Spec:** `docs/superpowers/specs/2026-03-11-marketplace-setup-design.md`

---

## Chunk 1: Create marketplace and plugin manifests

### Task 1: Create marketplace manifest

**Files:**
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create `.claude-plugin/` directory and `marketplace.json`**

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

- [ ] **Step 2: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: add marketplace manifest"
```

### Task 2: Create plugin manifest

**Files:**
- Create: `plugins/nightshift/.claude-plugin/plugin.json`

- [ ] **Step 1: Create directory and `plugin.json`**

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

- [ ] **Step 2: Commit**

```bash
git add plugins/nightshift/.claude-plugin/plugin.json
git commit -m "feat: add plugin manifest"
```

## Chunk 2: Move files into plugin directory

### Task 3: Move agents into plugin

**Files:**
- Move: `agents/*.md` → `plugins/nightshift/agents/`

- [ ] **Step 1: Move all agent files**

```bash
mkdir -p plugins/nightshift/agents
git mv agents/code-reviewer.md plugins/nightshift/agents/
git mv agents/dead-code-detector.md plugins/nightshift/agents/
git mv agents/dependency-updater.md plugins/nightshift/agents/
git mv agents/issue-resolver.md plugins/nightshift/agents/
git mv agents/readme-auditor.md plugins/nightshift/agents/
git mv agents/security-auditor.md plugins/nightshift/agents/
rm -rf agents
```

- [ ] **Step 2: Commit**

```bash
git add -A
git commit -m "refactor: move agents into plugin directory"
```

### Task 4: Move workflows into plugin

**Files:**
- Move: `workflows/*.yml` → `plugins/nightshift/workflows/`

- [ ] **Step 1: Move all workflow files**

```bash
mkdir -p plugins/nightshift/workflows
git mv workflows/nightshift-code-reviewer.yml plugins/nightshift/workflows/
git mv workflows/nightshift-dead-code-detector.yml plugins/nightshift/workflows/
git mv workflows/nightshift-dependency-updater.yml plugins/nightshift/workflows/
git mv workflows/nightshift-issue-resolver.yml plugins/nightshift/workflows/
git mv workflows/nightshift-readme-auditor.yml plugins/nightshift/workflows/
git mv workflows/nightshift-security-auditor.yml plugins/nightshift/workflows/
rm -rf workflows
```

- [ ] **Step 2: Commit**

```bash
git add -A
git commit -m "refactor: move workflows into plugin directory"
```

### Task 5: Move skills into plugin

**Files:**
- Move: `skills/` → `plugins/nightshift/skills/`

- [ ] **Step 1: Move all skill directories**

```bash
mkdir -p plugins/nightshift/skills
git mv skills/nightshift plugins/nightshift/skills/
git mv skills/creating-issues plugins/nightshift/skills/
git mv skills/creating-pull-requests plugins/nightshift/skills/
git mv skills/updating-readmes plugins/nightshift/skills/
git mv skills/writing-adrs plugins/nightshift/skills/
git mv skills/writing-project-updates plugins/nightshift/skills/
git mv skills/writing-wiki-pages plugins/nightshift/skills/
rm -rf skills
```

- [ ] **Step 2: Commit**

```bash
git add -A
git commit -m "refactor: move skills into plugin directory"
```

## Chunk 3: Update the nightshift setup skill

### Task 6: Update `/nightshift` SKILL.md for plugin paths

**Files:**
- Modify: `plugins/nightshift/skills/nightshift/SKILL.md`

- [ ] **Step 1: Update agent file reference**

Replace the current line 67:
```
For agent markdown files: copy them from the Nightshift plugin's `agents/` directory. The source files are at the root of the nightshift plugin repo under `agents/`.
```

With:
```
For agent markdown files: read each selected agent's source file from this plugin's `agents/` directory using bash:

```bash
cat "$CLAUDE_PLUGIN_ROOT/agents/<agent-name>.md"
```

Copy the content into the target repo at `.github/nightshift/agents/<agent-name>.md`.

If `$CLAUDE_PLUGIN_ROOT` is not set, read the files from the `agents/` directory relative to this skill file (two levels up: `../../agents/`).
```

- [ ] **Step 2: Add settings scaffolding step**

After the "### 5. Remind about secrets" section (line 104-109), add a new section:

```markdown
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
```

- [ ] **Step 3: Renumber the commit section from 6 to 7**

Update "### 6. Commit" to "### 7. Commit" and update the commit message to include settings.json if it was added.

- [ ] **Step 4: Commit**

```bash
git add plugins/nightshift/skills/nightshift/SKILL.md
git commit -m "feat: update nightshift skill for plugin paths and CI settings"
```

## Chunk 4: Update documentation

### Task 7: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Update repo structure section**

Replace the full repo structure section (lines 7-21, from `## Repo Structure` through the closing triple backticks) with:

```markdown
## Repo Structure

```
.claude-plugin/          # Marketplace manifest
plugins/nightshift/      # The nightshift plugin
  .claude-plugin/        # Plugin manifest
  agents/                # Agent prompt library (source markdown files)
  workflows/             # GitHub Actions workflow templates
  skills/                # Claude Code plugin skills
    nightshift/          # /nightshift interactive setup skill
    creating-issues/
    creating-pull-requests/
    writing-wiki-pages/
    writing-project-updates/
    writing-adrs/
    updating-readmes/
docs/                    # Design documents (not part of plugin)
```
```

- [ ] **Step 2: Update conventions section**

Replace lines 30-37 with:

```markdown
## Conventions

- Agent markdown files go in `plugins/nightshift/agents/` (source of truth)
- When installed in a target repo, agents go in `.github/nightshift/agents/`
- Workflow templates go in `plugins/nightshift/workflows/`
- When installed, workflows go in `.github/workflows/nightshift-*.yml`
- All workflows include `workflow_dispatch` for manual triggering
- Skills follow the `SKILL.md` convention in `plugins/nightshift/skills/<name>/SKILL.md`
```

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md for marketplace structure"
```

### Task 8: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Update install command**

Replace line 21:
```
claude install klefix/nightshift-new
```

With:
```
/plugin marketplace add klefix/nightshift
/plugin install nightshift@nightshift
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: update README install commands for marketplace"
```

### Task 9: Update .gitignore

**Files:**
- Modify: `.gitignore`

- [ ] **Step 1: Add docs/superpowers to .gitignore**

The `docs/superpowers/` directory contains spec and plan files generated by the superpowers skills during development. These are working documents, not distributed content. Add to `.gitignore`:

```
docs/superpowers/
```

- [ ] **Step 2: Commit**

```bash
git add .gitignore
git commit -m "chore: ignore superpowers working docs"
```

## Chunk 5: Validate

### Task 10: Validate marketplace structure

- [ ] **Step 1: Run plugin validation**

```bash
claude plugin validate .
```

Expected: No errors. Warnings are acceptable (e.g., "no marketplace description" is fine since we have one).

- [ ] **Step 2: Verify directory structure is correct**

```bash
ls .claude-plugin/marketplace.json
ls plugins/nightshift/.claude-plugin/plugin.json
ls plugins/nightshift/agents/
ls plugins/nightshift/workflows/
ls plugins/nightshift/skills/
```

Expected: All files present, no leftover `agents/`, `workflows/`, or `skills/` directories at root.
