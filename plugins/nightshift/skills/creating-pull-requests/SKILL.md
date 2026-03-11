---
name: creating-pull-requests
description: Use when creating a pull request, submitting work for review, or running gh pr create — produces consistent, well-structured PR descriptions
---

# Creating Pull Requests

## Overview

Generate consistent, well-structured PR descriptions by analyzing the diff and commit history. Write for humans who weren't in the room.

## When to Use

- About to run `gh pr create`
- Submitting work for review
- Need a PR description that explains intent, not just lists files

## Process

### 1. Gather context

```bash
# Determine base branch
BASE=$(git rev-parse --abbrev-ref HEAD@{upstream} 2>/dev/null | sed 's|origin/||' || echo "main")

# Full diff against base
git diff "$BASE"...HEAD --stat
git diff "$BASE"...HEAD

# Commit history on this branch
git log "$BASE"..HEAD --oneline

# Check for related issues (look at branch name, commit messages)
gh issue list --state open --limit 10
```

### 2. Analyze and group changes

Group changes by **logical concern** (not by file):
- What feature/fix is this?
- What supporting changes were needed?
- Were there any refactors or cleanups bundled in?

### 3. Find related artifacts

- Issues: scan branch name and commit messages for `#NNN` or issue keywords
- ADRs: check `docs/adr/` for recently added/modified ADRs in the diff
- Wiki: note if wiki-worthy documentation should be written after merge

### 4. Write the PR description

```markdown
## Summary
<2-3 sentences: what changed and why — lead with the "so what">

## Changes
- <Grouped by logical concern>
- <Not file-by-file — intent, not mechanics>

## Context
<Link to issue (#NNN), ADR (docs/adr/NNNN-*.md), or brief architectural context>
<Omit this section if there's no meaningful context to add>

## Test Plan
- [ ] <Concrete verification steps>
- [ ] <Not just "run tests" — what specifically to check>
```

### 5. Create the PR

**Title must follow [Conventional Commits](https://www.conventionalcommits.org/):**

`<type>(<optional scope>): <short description>`

| Type | When |
|------|------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `docs` | Documentation only |
| `chore` | Build, CI, tooling, dependencies |
| `test` | Adding or fixing tests |
| `style` | Formatting, whitespace (no logic change) |
| `perf` | Performance improvement |

Examples:
- `feat: add JWT-based authentication`
- `fix(auth): handle expired refresh tokens`
- `chore: upgrade dependencies to latest`

**Why this matters:** GitHub uses the PR title as the squash-merge commit message. A conventional commit title means the merge commit is correctly formatted without manual editing.

```bash
gh pr create --title "<type>(<scope>): <description>" --body "$(cat <<'EOF'
<the description from step 4>
EOF
)"
```

Add flags as appropriate:
- `--assignee @me`
- `--label <label>` if relevant labels exist
- `--milestone <milestone>` if one applies
- `--draft` if work is still in progress

## Quality Bar

- **Conventional commit title:** Must start with `type:` or `type(scope):` — no exceptions
- **Self-contained:** A reader should understand the PR without clicking through to issues
- **Intent over mechanics:** "Add user authentication" not "modify 5 files"
- **Embedding-friendly:** Include technology names, pattern names, and domain terms naturally
- **Concise:** If the Summary section is longer than 3 sentences, it's too long

## Common Mistakes

**Listing files instead of intent**
- Problem: "Changed auth.ts, middleware.ts, schema.ts"
- Fix: "Added JWT-based authentication with middleware guard on protected routes"

**Empty test plan**
- Problem: "Run tests"
- Fix: "Verify login returns 200 with valid credentials and 401 without"

**Missing context**
- Problem: PR exists in a vacuum
- Fix: Link the issue, mention the ADR, explain why this approach
