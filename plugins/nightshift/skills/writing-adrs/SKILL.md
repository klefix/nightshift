---
name: writing-adrs
description: Use when making architectural decisions — choosing between approaches, adopting a technology, establishing a pattern, or changing a previous decision
---

# Writing ADRs

## Overview

Write Architecture Decision Records when choosing between approaches. ADRs capture the why behind decisions so future readers (humans or agents) understand the context. Based on Michael Nygard's lightweight template.

## When to Use

**Needs an ADR:**
- Choosing a database, framework, or library
- Deciding on a pattern (monorepo vs polyrepo, REST vs GraphQL)
- Establishing conventions (error handling strategy, folder structure)
- Changing an existing decision (superseding a previous ADR)

**Does NOT need an ADR:**
- Implementation details (how a function works)
- Bug fixes
- Cosmetic choices (formatting, linting rules)

## Process

### 1. Determine the ADR number

```bash
# Create directory if needed
mkdir -p docs/adr

# Find highest existing number
ls docs/adr/ | grep -oP '^\d+' | sort -n | tail -1
# Next number = highest + 1, or 0001 if none exist
```

### 2. Write the ADR

File: `docs/adr/NNNN-<slug>.md`

```markdown
# NNNN. <Decision Title>

Date: YYYY-MM-DD

## Status

Accepted

## Context

<What's the situation? What forces are at play? What problem are we solving?>

## Options Considered

1. **<Option A>** — <one-line summary>
   - Pros: <what's good>
   - Cons: <what's bad>

2. **<Option B>** — <one-line summary>
   - Pros: <what's good>
   - Cons: <what's bad>

3. **<Option C>** — <if applicable>
   - Pros: <what's good>
   - Cons: <what's bad>

## Decision

<What was chosen and why. Be specific about the reasoning.>

## Consequences

- <What becomes easier>
- <What becomes harder or more complex>
- <What assumptions need to hold for this decision to remain valid>
```

### 3. If superseding an old ADR

Update the old ADR's status:

```bash
# In the old ADR, change:
# ## Status
# Accepted
# to:
# ## Status
# Superseded by [NNNN](./NNNN-new-decision.md)
```

### 4. Commit alongside code

```bash
git add docs/adr/NNNN-<slug>.md
# Include in the same commit as the code change, or commit separately if the decision is standalone
git commit -m "docs: add ADR NNNN — <decision title>"
```

## Quality Bar

- **Short:** If it takes more than 5 minutes to read, it's too long
- **Honest about trade-offs:** Every decision has downsides — name them
- **Self-contained:** Reader understands the decision without chasing links
- **Embedding-friendly:** Name the technologies, patterns, and alternatives explicitly
- **Timestamped:** Always include the date

## Common Mistakes

**ADR without options**
- Problem: "We chose PostgreSQL" — but why not SQLite or MySQL?
- Fix: Always list what was considered, even if the choice was obvious

**Missing consequences**
- Problem: Only lists what becomes easier
- Fix: Honestly state what becomes harder or what assumptions must hold

**ADR for trivial decisions**
- Problem: "ADR 0047: Use prettier for formatting"
- Fix: Only write ADRs for decisions with real architectural impact

**Stale status**
- Problem: Old ADR still says "Accepted" but was replaced
- Fix: Always update the superseded ADR's status
