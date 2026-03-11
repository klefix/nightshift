---
name: writing-project-updates
description: Use after completing meaningful work — features merged, architectural decisions made, blockers resolved, or significant discoveries — to write project updates in GitHub Projects
---

# Writing Project Updates

## Overview

Write project updates to GitHub Projects after meaningful milestones. Decide autonomously what's "worthy" of an update. Write for a PM or stakeholder checking in.

## When to Use

**Worthy of an update:**
- A feature was completed (PR merged)
- A significant architectural decision was made
- A blocker was hit or resolved
- Multiple related issues were closed in a batch
- A discovery changed understanding of the project's state

**NOT worthy:**
- Routine dependency bumps
- Single typo fixes
- Config tweaks with no user-facing impact
- Work that's still in progress (wait until it's done)

## Process

### 1. Gather context

```bash
# Recent merged PRs
gh pr list --state merged --limit 10

# Recently closed issues
gh issue list --state closed --limit 10

# Recent commits
git log --oneline -20

# Check for new ADRs
ls docs/adr/ 2>/dev/null
```

### 2. Identify the update-worthy event

Batch related work into one update. Don't write one update per PR — group by theme:
- "Authentication system is now complete" (covers 3 PRs and 5 issues)
- "Migrated from REST to GraphQL for user endpoints" (covers 1 ADR and 2 PRs)

### 3. Find the project

```bash
# List projects for the repo owner
gh project list --owner <OWNER>

# Get project number
gh project view <NUMBER> --owner <OWNER>
```

### 4. Write and create the update

```bash
gh project item-create <PROJECT_NUMBER> --owner <OWNER> \
  --title "<Date> — <One-line summary>" \
  --body "$(cat <<'EOF'
**What happened:**
- <Concrete changes, linked to PRs/issues>

**Why it matters:**
- <Impact — what's now possible, what's unblocked>

**What's next:**
- <Natural follow-ups, if any>

**Open questions:**
- <Unresolved decisions or risks — omit if none>
EOF
)"
```

## Quality Bar

- **Batched:** One update per milestone, not per PR
- **Linked:** Reference PRs, issues, ADRs by number
- **Forward-looking:** Include "what's next" so readers know the trajectory
- **Embedding-friendly:** Name technologies, components, and decisions explicitly

## Common Mistakes

**One update per PR**
- Problem: Noisy, fragmented, hard to follow
- Fix: Batch related work under one thematic update

**Status without context**
- Problem: "Completed PR #47"
- Fix: "Added JWT authentication — users can now log in with email/password"

**Missing links**
- Problem: Update mentions work without linking to it
- Fix: Always reference PR/issue numbers
