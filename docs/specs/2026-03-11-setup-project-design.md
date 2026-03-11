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

The H1 heading inside the skill body is updated from `# Nightshift Setup` to `# Setup Agents`. No other content changes.

**Ordering:** The rename must land before or together with the new `setup-project` skill, since Step 7 of `setup-project` references `nightshift:setup-agents`.

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

**If multiple projects exist:** ask the user which one to use. If only one exists, adopt it automatically.

**If one project exists:** adopt it — record its number and check for missing fields/views.

**Checking existing fields** on the adopted project:

```bash
gh project field-list <PROJECT_NUMBER> --owner <OWNER> --format json
```

Compare returned fields against the required set. Only create fields that are missing.

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

**Creating a single-select field** (example for Status):

```bash
gh api graphql -f query='
  mutation {
    createProjectV2Field(input: {
      projectId: "<PROJECT_NODE_ID>"
      dataType: SINGLE_SELECT
      name: "Status"
      singleSelectOptions: [
        {name: "Backlog", color: GRAY}
        {name: "Todo", color: BLUE}
        {name: "In Progress", color: YELLOW}
        {name: "Done", color: GREEN}
      ]
    }) {
      projectV2Field { ... on ProjectV2SingleSelectField { id options { id name } } }
    }
  }
'
```

Use the same pattern for the Priority field (colors: GRAY, BLUE, ORANGE, RED).

**Add Board view** grouped by Status field. Check existing views first via `gh project view <NUMBER> --owner <OWNER> --format json` — only create if no Board view exists.

**Retrieving field and option IDs** for CLAUDE.md (Step 6):

```bash
gh project field-list <PROJECT_NUMBER> --owner <OWNER> --format json
```

Parse the JSON output to extract field IDs and option IDs for Status and Priority fields.

#### Step 3 — Wiki

```bash
# Enable wiki if not already enabled
gh api repos/<OWNER>/<REPO> -X PATCH -f has_wiki=true
```

**Clone wiki repo** using `gh` for automatic auth:

```bash
gh repo clone <OWNER>/<REPO>.wiki /tmp/<REPO>-wiki
```

**If clone fails** (wiki has no content yet — common for newly enabled wikis):

```bash
mkdir -p /tmp/<REPO>-wiki
cd /tmp/<REPO>-wiki
git init
git checkout -b main
git remote add origin https://github.com/<OWNER>/<REPO>.wiki.git
```

**Idempotency:** If `Home.md` already exists in the wiki, skip this step and inform the user: "Wiki Home.md already exists — skipping." Do not overwrite user-edited content.

**If Home.md does not exist**, create it. Determine the project board URL based on owner type:
- Organization: `https://github.com/orgs/<OWNER>/projects/<NUMBER>`
- User account: `https://github.com/users/<OWNER>/projects/<NUMBER>`

Check owner type via `gh api users/<OWNER> --jq '.type'` (returns "Organization" or "User").

```markdown
# <REPO> Wiki

> Project documentation and guides.

## Quick Links

- [Repository](https://github.com/<OWNER>/<REPO>)
- [Project Board](<appropriate URL based on owner type>)
- [Issues](https://github.com/<OWNER>/<REPO>/issues)

## Contents

_Pages will be added as the project grows._
```

**Commit and push:**

```bash
cd /tmp/<REPO>-wiki
git add -A
git commit -m "docs: initialize wiki with Home page"
git push -u origin main
```

**Clean up** only on successful push:

```bash
rm -rf /tmp/<REPO>-wiki
```

If push fails, inform the user and leave `/tmp/<REPO>-wiki` intact for manual recovery.

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

**If yes**, first check for an existing "Protect main" ruleset:

```bash
gh api repos/<OWNER>/<REPO>/rulesets --jq '.[] | select(.name == "Protect main")'
```

If it already exists, inform the user: "Branch protection ruleset 'Protect main' already exists — skipping." Do not create a duplicate.

If it does not exist, apply via repository ruleset (preferred over legacy branch protection for GitHub Free compatibility):

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
- If section doesn't exist: append at the end of the file, ensuring a blank line separator before the new heading

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
| `plugins/nightshift/skills/nightshift/SKILL.md` | Update frontmatter name to `setup-agents`, H1 to `# Setup Agents` |
