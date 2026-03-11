---
name: writing-wiki-pages
description: Use when documenting architecture, onboarding guides, tech stack summaries, integration guides, or runbooks — writes to GitHub Wiki
---

# Writing Wiki Pages

## Overview

Create and update GitHub Wiki pages for documentation that doesn't belong in code comments or ADRs. Write for someone onboarding or a PM who needs to understand the system.

## When to Use

**Belongs in wiki:**
- Architecture overviews (how the system fits together)
- Getting started / onboarding guides
- Tech stack summaries (what's used and why)
- Integration guides (how services/modules talk to each other)
- Runbooks (how to deploy, debug, recover)

**Does NOT belong in wiki:**
- Decision rationale → ADR (`docs/adr/`)
- API reference generated from code → code docs
- Changelog-style updates → GitHub Projects

## Process

### 1. Clone the wiki

```bash
# Wiki is a separate git repo
gh repo clone <OWNER>/<REPO>.wiki.git /tmp/<REPO>-wiki

# If wiki doesn't exist yet, this will fail — create first page via:
# gh api repos/<OWNER>/<REPO>/pages (or create via GitHub UI)
```

### 2. Check existing pages

```bash
ls /tmp/<REPO>-wiki/
cat /tmp/<REPO>-wiki/Home.md 2>/dev/null
```

Decide: **update** existing page or **create** new one.
- Same topic exists → update it
- New topic → create and add to `Home.md`

### 3. Write the page

```markdown
# <Page Title>

> <One-sentence summary of what this page covers>

## Overview
<High-level context — what is this, why does it exist>

## <Topic sections>
<Content — use diagrams, tables, code blocks where they aid understanding>
<Be concrete: name the technologies, reference the file paths, show the commands>

## Related
- [ADR: <decision>](../docs/adr/NNNN-*.md)
- [Wiki: <other page>](<Other-Page>)
- Issue #NNN, PR #NNN
```

### 4. Update navigation

**Home.md** (index page):
```markdown
# <Project Name> Wiki

## Contents
- [Architecture](Architecture)
- [Getting Started](Getting-Started)
- [Tech Stack](Tech-Stack)
- ...
```

**_Sidebar.md** (navigation — create if missing):
```markdown
**Navigation**
- [Home](Home)
- [Architecture](Architecture)
- [Getting Started](Getting-Started)
- ...
```

### 5. Commit and push

```bash
cd /tmp/<REPO>-wiki
git add -A
git commit -m "docs: add/update <page-name>"
git push
```

### 6. Clean up

```bash
rm -rf /tmp/<REPO>-wiki
```

## Quality Bar

- **Self-contained:** Reader doesn't need to read code to understand
- **Current:** If you're writing a wiki page, make sure it reflects the actual state
- **Cross-linked:** Reference ADRs, issues, and other wiki pages
- **Embedding-friendly:** Include component names, technology names, and architectural terms
- **Skimmable:** Headings, bullets, tables — not walls of text

## Common Mistakes

**Wiki page that duplicates an ADR**
- Problem: Architecture page explains why SQLite was chosen
- Fix: Reference the ADR, summarize the outcome, not the deliberation

**Stale content**
- Problem: Wiki says "we use Express" but project migrated to Fastify
- Fix: Always check current project state before writing

**No navigation**
- Problem: Pages exist but nobody can find them
- Fix: Always update Home.md and _Sidebar.md
