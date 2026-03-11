---
name: creating-issues
description: Use when creating, updating, or commenting on GitHub issues — covers bugs, features, tasks, divergence notes, and discovery observations
---

# Creating Issues

## Overview

Create, update, and comment on GitHub issues with enough context for someone (human or agent) to pick them up without asking questions. Write for a PM reading this on Monday morning.

## When to Use

- Creating a new bug report, feature request, or task
- Updating an existing issue with new context
- Commenting on an issue to explain divergence from plan
- Documenting discoveries (brittle code, flaky tests, tech debt) found during work
- Closing an issue with meaningful context

## Process

### Creating Issues

#### 1. Determine issue type

Detect from context or ask. Three templates:

**Bug:**
```markdown
## Problem
<What's broken — observed vs expected behavior>

## Steps to Reproduce
1. <Concrete, numbered steps>

## Environment
<Stack version, OS, relevant config>

## Possible Cause
<Only if you have actual insight — omit otherwise>
```

**Feature:**
```markdown
## Goal
<What should be possible after this is done>

## Context
<Why this matters, what triggered the request>

## Acceptance Criteria
- [ ] <Concrete, verifiable criteria>

## Notes
<Constraints, related decisions, links to ADRs>
```

**Task:**
```markdown
## What
<Concrete deliverable>

## Why
<Context — not just "because we need it">

## Done When
- [ ] <Checklist>
```

#### 2. Create the issue

```bash
gh issue create \
  --title "<concise title>" \
  --body "$(cat <<'EOF'
<body from template above>
EOF
)" \
  --label "<bug|enhancement|task>" \
  --assignee "@me"
```

### Commenting on Issues

Three categories — only comment when it adds value. If implementation matched the issue exactly, a silent close via PR linkage is fine.

#### Divergence — implementation took a different path

```bash
gh issue comment <NUMBER> --body "$(cat <<'EOF'
## Resolution
<What was done and how it differs from the original plan>

## Why the divergence
<What was infeasible and what was chosen instead>

## Related
- PR #XX
- ADR docs/adr/NNNN-*.md (if applicable)
EOF
)"
```

#### Discovery — found something noteworthy during the work

```bash
gh issue comment <NUMBER> --body "$(cat <<'EOF'
## Observations
- <What was found — e.g., "Integration tests for payment module are flaky due to shared DB state">
- <Severity/impact — e.g., "Caused 2 test retries during this PR">

## Suggested Follow-up
- <Concrete next step — e.g., "Isolate test DB per suite">
- Label suggestion: `tech-debt`, `flaky-test`, `needs-refactor`
EOF
)"
```

When a discovery is significant enough (will keep causing problems), create a follow-up issue — but ask the user for permission first.

#### Closure context — when "done" isn't enough

```bash
gh issue comment <NUMBER> --body "$(cat <<'EOF'
## What actually happened
<Summary of the implementation, linking to PR>

## Notable decisions
<Anything future readers should know>
EOF
)"
```

### Updating Issues

```bash
# Add or change labels
gh issue edit <NUMBER> --add-label "tech-debt"

# Update title or body
gh issue edit <NUMBER> --title "new title"
gh issue edit <NUMBER> --body "updated body"
```

## Quality Bar

- **Self-contained:** Reader understands the issue without digging through code
- **Actionable:** Someone can start working on it immediately
- **Embedding-friendly:** Include technology names, error messages, component names
- **No noise:** Don't comment just to comment — only when it adds value

## Common Mistakes

**Vague bug reports**
- Problem: "Login is broken"
- Fix: "Login returns 500 when email contains '+' character — expected 200 with session cookie"

**Missing acceptance criteria**
- Problem: "Add dark mode"
- Fix: Concrete checkboxes someone can verify

**Commenting for the sake of it**
- Problem: "Fixed in PR #47" on every closed issue
- Fix: Only comment when there's divergence, discovery, or non-obvious context
