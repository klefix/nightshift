# Design: `nightshift:setup-project` Skill

**Date:** 2026-03-11
**Status:** Draft

## Summary

A new Claude Code skill (`nightshift:setup-project`) that scaffolds GitHub repository infrastructure: Project board, wiki, issue labels, branch protection, and CLAUDE.md project metadata. Designed for one-time-per-repo use, idempotent on re-run.

The existing `/nightshift` skill is renamed to `nightshift:setup-agents` to clarify the separation of concerns.

## Motivation

Nightshift agents depend on GitHub infrastructure (project boards for tracking, labels for triggering, wiki for documentation). Currently this is set up manually. Automating it ensures consistency and captures the project board IDs that agents and skills need.

## Scope

### In scope
- Create or adopt a GitHub Project (v2) with Status and Priority fields
- Add a Board view grouped by Status
- Enable wiki and seed Home.md
- Create `nightshift` issue label
- Optionally protect the `main` branch (require PRs, no force push, no deletion)
- Upsert CLAUDE.md with project board IDs

### Out of scope
- Issue templates
- Milestones
- Additional wiki pages beyond Home.md
- Sprint/iteration fields
- Custom GitHub Actions workflows

## Approach

Pure SKILL.md — Claude uses `gh` CLI commands to perform all operations. No helper scripts, no code artifacts. Consistent with all other nightshift skills.

## Design

### Skill metadata

```yaml
name: setup-project
description: Set up GitHub Project board, wiki, labels, and branch protection for a repository — one-time repo infrastructure scaffolding
```

**Location:** `plugins/nightshift/skills/setup-project/SKILL.md`

### Rename of existing skill

The current `plugins/nightshift/skills/nightshift/` directory is renamed to `plugins/nightshift/skills/setup-agents/`. The skill frontmatter is updated:

```yaml
name: setup-agents
description: Set up Nightshift agents in a repository — scaffolds agent prompts and GitHub Actions workflows for automated code review, security audits, dependency updates, and more
```

No changes to skill content — only the directory name and frontmatter `name` field.

### Procedure

#### Step 1 — Detect context

Gather current state before making changes:

```bash
# Get repo owner and name
gh repo view --json owner,name,hasWikiEnabled

# Check for existing GitHub Projects
gh project list --owner <OWNER> --format json

# Check for nightshift label
gh label list --search "nightshift"

# Check for branch protection on main
gh api repos/<OWNER>/<REPO>/branches/main/protection 2>/dev/null
```

Report findings to the user: "Here's what I found: [project exists / no project], [wiki enabled / disabled], [label exists / missing], [main protected / unprotected]."

#### Step 2 — GitHub Project (create or adopt)

**If no project exists:**

```bash
gh project create --owner <OWNER> --title "<REPO>" --format json
```

**If project exists:** adopt it and check for missing fields/views.

**Configure Status field** (single-select) with options:
- Backlog
- Todo
- In Progress
- Done

**Configure Priority field** (single-select) with options:
- Low
- Medium
- High
- Critical

**Add Board view** grouped by Status field.

All field/view operations use `gh api graphql` with the Projects v2 GraphQL API. The skill instructs Claude to query existing fields first, then only create what's missing (idempotent).

#### Step 3 — Wiki

```bash
# Enable wiki if not already enabled
gh api repos/<OWNER>/<REPO> -X PATCH -f has_wiki=true

# Clone wiki repo to temp directory
git clone https://github.com/<OWNER>/<REPO>.wiki.git /tmp/<REPO>-wiki 2>/dev/null

# If clone fails (no wiki content yet), initialize it
mkdir -p /tmp/<REPO>-wiki && cd /tmp/<REPO>-wiki && git init
git remote add origin https://github.com/<OWNER>/<REPO>.wiki.git
```

**Create Home.md:**

```markdown
# <REPO> Wiki

> Project documentation and guides.

## Quick Links

- [Repository](https://github.com/<OWNER>/<REPO>)
- [Project Board](https://github.com/orgs/<OWNER>/projects/<NUMBER>) or user project URL
- [Issues](https://github.com/<OWNER>/<REPO>/issues)

## Contents

_Pages will be added as the project grows._
```

```bash
cd /tmp/<REPO>-wiki
git add -A
git commit -m "docs: initialize wiki with Home page"
git push -u origin main  # or master, depending on default branch
rm -rf /tmp/<REPO>-wiki
```

#### Step 4 — Issue label

```bash
gh label create "nightshift" \
  --description "Managed by Nightshift agents" \
  --color "1a3a5c" \
  --force  # updates if exists
```

#### Step 5 — Branch protection (interactive)

Ask the user:

> "Want to protect the `main` branch? This would:
> - Prevent force pushes
> - Prevent branch deletion
> - Require pull requests before merging (no direct pushes)
>
> Set this up?"

**If yes**, apply via repository ruleset (preferred over legacy branch protection for GitHub Free compatibility):

```bash
gh api repos/<OWNER>/<REPO>/rulesets -X POST --input - <<'EOF'
{
  "name": "Protect main",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["refs/heads/main"],
      "exclude": []
    }
  },
  "rules": [
    { "type": "deletion" },
    { "type": "non_fast_forward" },
    { "type": "pull_request", "parameters": {
        "required_approving_review_count": 0,
        "dismiss_stale_reviews_on_push": false,
        "require_code_owner_review": false,
        "require_last_push_approval": false,
        "required_review_thread_resolution": false
    }}
  ]
}
EOF
```

Note: `required_approving_review_count: 0` means PRs are required but no approvals needed — least opinionated while still requiring the PR workflow.

**If no**, skip. Print "Skipped branch protection."

#### Step 6 — Update CLAUDE.md

Upsert a `## GitHub Project Board` section in the repo's CLAUDE.md with:

```markdown
## GitHub Project Board

Project #<NUMBER> (owner: <OWNER>). Status field options:

| Status | Option ID |
|--------|-----------|
| Backlog | `<id>` |
| Todo | `<id>` |
| In Progress | `<id>` |
| Done | `<id>` |

Status field ID: `<field_id>`

Priority field options:

| Priority | Option ID |
|----------|-----------|
| Low | `<id>` |
| Medium | `<id>` |
| High | `<id>` |
| Critical | `<id>` |

Priority field ID: `<field_id>`
```

**Upsert behavior:**
- If `## GitHub Project Board` section exists: replace everything from that heading to the next `##` heading (or end of file)
- If section doesn't exist: append at the end of the file

The IDs are retrieved from the GraphQL API after creating/adopting the project fields.

#### Step 7 — Summary

Print a summary of what was done:

```
Setup complete:
  [created/adopted] GitHub Project "<REPO>" (#<NUMBER>)
  [created/verified] Status field (Backlog, Todo, In Progress, Done)
  [created/verified] Priority field (Low, Medium, High, Critical)
  [created/verified] Board view
  [enabled/already enabled] Wiki with Home.md
  [created/already existed] "nightshift" label (#1a3a5c)
  [applied/skipped] Branch protection on main
  [updated] CLAUDE.md with project board IDs

Next: run /nightshift:setup-agents to install automated agents.
```

## Quality Bar

- **Idempotent:** Safe to re-run — detects existing resources, fills gaps, doesn't duplicate
- **Interactive:** Asks before destructive or opinionated actions (branch protection)
- **Transparent:** Reports what it found and what it changed
- **Minimal:** Does exactly what's needed, nothing more

## Changes Summary

| What | Action |
|------|--------|
| `plugins/nightshift/skills/setup-project/SKILL.md` | Create (new skill) |
| `plugins/nightshift/skills/nightshift/` | Rename to `skills/setup-agents/` |
| `plugins/nightshift/skills/nightshift/SKILL.md` | Update frontmatter name to `setup-agents` |
