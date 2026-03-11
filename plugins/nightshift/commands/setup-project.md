---
description: Set up GitHub Project board, wiki, labels, and branch protection for a repository — one-time repo infrastructure scaffolding
---

# Setup Project

## Overview

Scaffold GitHub repository infrastructure for use with Nightshift agents: a Project board with Status and Priority fields, wiki with a Home page, issue labels, optional branch protection, and CLAUDE.md metadata. Idempotent — safe to re-run on repos that already have partial setup.

## When to Use

- First-time setup of a repository before installing Nightshift agents
- Re-running to fill in missing infrastructure (e.g. wiki was skipped, label is missing)
- Adopting an existing GitHub Project and recording its IDs in CLAUDE.md

## Process

### 1. Detect context

Gather current state before making changes:

```bash
# Get repo owner and name
gh repo view --json owner,name,hasWikiEnabled

# Check for existing GitHub Projects
gh project list --owner <OWNER> --format json

# Check for nightshift label
gh label list --search "nightshift"

# Check for branch protection on main
gh api repos/<OWNER>/<REPO>/rulesets --jq '.[] | select(.name == "Protect main")' 2>/dev/null
```

Report findings to the user: "Here's what I found: [project exists / no project], [wiki enabled / disabled], [label exists / missing], [main protected / unprotected]."

### 2. GitHub Project (create or adopt)

**If no project exists:**

```bash
gh project create --owner <OWNER> --title "<REPO>" --format json
```

**If multiple projects exist:** ask the user which one to use.

**If one project exists:** adopt it — record its number and check for missing fields/views.

**Check existing fields:**

```bash
gh project field-list <PROJECT_NUMBER> --owner <OWNER> --format json
```

Compare returned fields against the required set. Only create fields that are missing.

**Status field** (single-select): Backlog, Todo, In Progress, Done

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

**Priority field** (single-select): Low, Medium, High, Critical

Use the same mutation pattern with colors: GRAY, BLUE, ORANGE, RED.

**Board view:** Check existing views first via `gh project view <NUMBER> --owner <OWNER> --format json` — only create if no Board view exists. Group by Status field.

**Retrieve field and option IDs** for use in step 6:

```bash
gh project field-list <PROJECT_NUMBER> --owner <OWNER> --format json
```

Parse the JSON output to extract field IDs and option IDs for Status and Priority fields.

### 3. Wiki

```bash
# Enable wiki if not already enabled
gh api repos/<OWNER>/<REPO> -X PATCH -f has_wiki=true
```

**Clone wiki repo** using `gh` for automatic auth:

```bash
gh repo clone <OWNER>/<REPO>.wiki /tmp/<REPO>-wiki
```

If clone fails (wiki has no content yet — common for newly enabled wikis):

```bash
mkdir -p /tmp/<REPO>-wiki
cd /tmp/<REPO>-wiki
git init
git checkout -b main
gh auth setup-git
git remote add origin https://github.com/<OWNER>/<REPO>.wiki.git
```

If `Home.md` already exists in the wiki, skip and inform the user: "Wiki Home.md already exists — skipping." Do not overwrite user-edited content.

If `Home.md` does not exist, create it. Determine the project board URL based on owner type — check via `gh api users/<OWNER> --jq '.type'` (returns "Organization" or "User"):
- Organization: `https://github.com/orgs/<OWNER>/projects/<NUMBER>`
- User account: `https://github.com/users/<OWNER>/projects/<NUMBER>`

```markdown
# <REPO> Wiki

> Project documentation and guides.

## Quick Links

- [Repository](https://github.com/<OWNER>/<REPO>)
- [Project Board](<URL based on owner type>)
- [Issues](https://github.com/<OWNER>/<REPO>/issues)

## Contents

_Pages will be added as the project grows._
```

Commit and push:

```bash
cd /tmp/<REPO>-wiki
git add -A
git commit -m "docs: initialize wiki with Home page"
git push -u origin main
```

Clean up only on successful push:

```bash
rm -rf /tmp/<REPO>-wiki
```

If push fails, inform the user and leave `/tmp/<REPO>-wiki` intact for manual recovery.

### 4. Issue label

```bash
gh label create "nightshift" \
  --description "Managed by Nightshift agents" \
  --color "1a3a5c" \
  --force
```

The `--force` flag updates the label if it already exists.

### 5. Branch protection (interactive)

Ask the user:

> "Want to protect the `main` branch? This would:
> - Prevent force pushes
> - Prevent branch deletion
> - Require pull requests before merging (no direct pushes)
>
> Set this up?"

**If yes**, first check for an existing ruleset:

```bash
gh api repos/<OWNER>/<REPO>/rulesets --jq '.[] | select(.name == "Protect main")'
```

If it already exists, inform the user: "Branch protection ruleset 'Protect main' already exists — skipping."

If it does not exist, create it:

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

Uses repository rulesets (not legacy branch protection) for GitHub Free compatibility. `required_approving_review_count: 0` means PRs are required but no approvals needed.

**If no**, skip. Print "Skipped branch protection."

### 6. Update CLAUDE.md

Upsert a `## GitHub Project Board` section in the repo's CLAUDE.md with the project number, field IDs, and option IDs collected in step 2:

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

### 7. Summary

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

Next: run nightshift:setup-agents to install automated agents.
```

## Quality Bar

- **Idempotent:** Safe to re-run — detects existing resources, fills gaps, doesn't duplicate
- **Interactive:** Asks before destructive or opinionated actions (branch protection)
- **Transparent:** Reports what it found and what it changed
- **Minimal:** Does exactly what's needed, nothing more

## Common Mistakes

**Creating duplicate project fields**
- Problem: Running the skill twice creates two "Status" fields
- Fix: Always check `gh project field-list` before creating — only add missing fields

**Overwriting an edited wiki Home page**
- Problem: User customized Home.md, skill replaces it with the template
- Fix: If Home.md exists, skip and inform — never overwrite

**Using legacy branch protection instead of rulesets**
- Problem: Legacy branch protection requires a paid plan for private repos
- Fix: Always use repository rulesets (`/rulesets` endpoint), which work on GitHub Free
