# Setup Project Skill Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `nightshift:setup-project` skill that scaffolds GitHub Project, wiki, labels, and branch protection; rename the existing `/nightshift` skill to `nightshift:setup-agents`.

**Architecture:** Two independent tasks — a directory rename + frontmatter edit for the existing skill, and a new SKILL.md file for the setup-project skill. Pure markdown, no code artifacts.

**Tech Stack:** Markdown (SKILL.md files), `gh` CLI commands documented within the skill.

**Spec:** `docs/specs/2026-03-11-setup-project-design.md`

---

## Chunk 1: Rename and Create Skills

### Task 1: Rename `nightshift` skill to `setup-agents`

**Files:**
- Rename: `plugins/nightshift/skills/nightshift/` → `plugins/nightshift/skills/setup-agents/`
- Modify: `plugins/nightshift/skills/setup-agents/SKILL.md` (frontmatter + H1)

- [ ] **Step 1: Rename the directory**

```bash
git mv plugins/nightshift/skills/nightshift plugins/nightshift/skills/setup-agents
```

- [ ] **Step 2: Update frontmatter name field**

In `plugins/nightshift/skills/setup-agents/SKILL.md`, change:

```yaml
name: nightshift
```

to:

```yaml
name: setup-agents
```

- [ ] **Step 3: Update H1 heading**

In `plugins/nightshift/skills/setup-agents/SKILL.md`, change:

```markdown
# Nightshift Setup
```

to:

```markdown
# Setup Agents
```

- [ ] **Step 4: Verify no other content changed**

Read `plugins/nightshift/skills/setup-agents/SKILL.md` and confirm:
- Frontmatter `name` is `setup-agents`
- Frontmatter `description` matches spec: `Set up Nightshift agents in a repository — scaffolds agent prompts and GitHub Actions workflows for automated code review, security audits, dependency updates, and more` (the current description already matches — no change needed, but verify)
- H1 heading is `# Setup Agents`
- All other procedure content remains identical

- [ ] **Step 5: Commit**

```bash
git add plugins/nightshift/skills/
git commit -m "refactor: rename nightshift skill to setup-agents"
```

---

### Task 2: Create `setup-project` SKILL.md

**Files:**
- Create: `plugins/nightshift/skills/setup-project/SKILL.md`

- [ ] **Step 1: Create the skill directory**

```bash
mkdir -p plugins/nightshift/skills/setup-project
```

- [ ] **Step 2: Write the SKILL.md**

Create `plugins/nightshift/skills/setup-project/SKILL.md` with the full skill content. The file must include:

**Frontmatter:**
```yaml
---
name: setup-project
description: Set up GitHub Project board, wiki, labels, and branch protection for a repository — one-time repo infrastructure scaffolding
---
```

**Sections (in order):**
1. `# Setup Project` — H1 heading
2. `## Overview` — one-paragraph summary of what the skill does
3. `## When to Use` — first-time repo setup, re-running to fill gaps
4. `## Process` — the 7-step procedure from the spec:
   - Step 1: Detect context (`gh repo view`, `gh project list`, `gh label list`, `gh api` for branch protection)
   - Step 2: GitHub Project (create or adopt, Status field, Priority field, Board view, with GraphQL mutation examples and idempotent field checking)
   - Step 3: Wiki (enable via `gh api`, clone via `gh repo clone`, fallback init with `git checkout -b main` + `gh auth setup-git`, Home.md creation with owner-type-aware project URL, idempotent skip if Home.md exists, cleanup only on success)
   - Step 4: Issue label (`gh label create "nightshift" --color "1a3a5c" --force`)
   - Step 5: Branch protection (interactive — ask user, check for existing "Protect main" ruleset before creating, full ruleset JSON payload)
   - Step 6: Update CLAUDE.md (upsert `## GitHub Project Board` section with Status + Priority field/option IDs, blank line before append)
   - Step 7: Summary (print what was created/adopted/skipped, suggest `nightshift:setup-agents` next)
5. `## Quality Bar` — Idempotent, Interactive, Transparent, Minimal
6. `## Common Mistakes` — at least 2-3 common mistakes following the pattern of other skills

**Style reference:** Match the tone, structure, and formatting of existing skills. Use `plugins/nightshift/skills/writing-wiki-pages/SKILL.md` as the primary style reference (flat process structure matches `setup-project`). Key patterns:
- Commands use ```bash code blocks with `gh` CLI
- Templates use ```markdown code blocks
- Each process step uses an H3 numbered heading: `### 1. Step name`
- Idempotency notes are inline, not in a separate section

**Important:** The spec uses `gh repo clone <OWNER>/<REPO>.wiki` (no `.git` suffix). The style reference `writing-wiki-pages` uses `.wiki.git`. Follow the spec form (`.wiki` without `.git`).

**Content source:** All procedure details, commands, and templates come directly from the spec at `docs/specs/2026-03-11-setup-project-design.md`. Do not invent new steps or change the procedure.

- [ ] **Step 3: Verify the skill matches the spec**

Read back the created SKILL.md and verify:
- All 7 steps from the spec are present
- GraphQL mutation example is included for field creation
- Wiki clone uses `gh repo clone` (not raw HTTPS)
- Wiki fallback includes `git checkout -b main` and `gh auth setup-git`
- Wiki has idempotency check for existing Home.md
- Branch protection has idempotency check for existing ruleset
- CLAUDE.md upsert specifies blank line separator
- Label color is `1a3a5c`
- Owner type check for project board URL is included
- Board view has idempotency check (existing views checked via `gh project view` before creating)

- [ ] **Step 4: Commit**

```bash
git add plugins/nightshift/skills/setup-project/SKILL.md
git commit -m "feat: add setup-project skill for repo infrastructure scaffolding"
```

---

### Task 3: Update README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Read the current README**

Read `README.md` and find references to the old `/nightshift` skill name. Key locations to update:
- The skill table row (lists `/nightshift` as a command)
- The Quick Start code block (shows `> /nightshift` as a command invocation)
- The command invocation changes from `/nightshift` to `nightshift:setup-agents`

- [ ] **Step 2: Update skill references**

- Add `setup-project` to the skills list
- Update the old `nightshift` reference to `setup-agents`
- Add a brief description of what `setup-project` does

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: update README with setup-project skill and setup-agents rename"
```
