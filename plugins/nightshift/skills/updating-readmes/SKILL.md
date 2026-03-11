---
name: updating-readmes
description: Use when project structure, setup instructions, or capabilities change — updates README files at any level (root, packages, sub-modules) and detects stale content
---

# Updating READMEs

## Overview

Create and update README files at any level in a repo — root, packages, sub-apps, modules. Detect staleness and preserve custom sections. Write for someone cloning the repo for the first time.

## When to Use

- Project structure changed (new directories, removed modules)
- Setup instructions changed (new env vars, different build steps)
- Capabilities changed (new features, removed endpoints)
- New module added without a README
- Existing README references things that no longer exist

## Process

### 1. Discover existing READMEs

```bash
# Find all READMEs in the repo
fd README.md

# Check for staleness — does the README reference things that don't exist?
# Compare README mentions of directories/files against actual structure
```

### 2. Determine scope

- **Root README** — project overview, quickstart, architecture summary, links to sub-READMEs
- **Sub-README** (e.g., `packages/api/README.md`) — scoped to that module only

### 3. Write or update

**Root README template:**
```markdown
# <Project Name>

<One-paragraph description: what this is and why it exists>

## Quick Start

<Minimal steps to get running locally — copy-pasteable commands>

## Architecture

<High-level overview — link to wiki for detailed docs>

## Project Structure

<Key directories explained — only top-level, not exhaustive>

## Contributing

<How to work on this — link to wiki onboarding guide if it exists>
```

**Sub-module README template:**
```markdown
# <Module Name>

<What this module does and its role in the system>

## Usage

<How to use/import/call this module>

## Configuration

<Relevant env vars, config files — omit if none>
```

### 4. Preserve custom sections

When updating an existing README:
- Read the current content first
- Identify sections the user added manually (anything not matching the templates above)
- Preserve those sections — update the templated sections, leave custom ones intact
- If unsure whether a section is custom, keep it

### 5. Suggest new READMEs

When discovering a module without a README:
- Ask the user before creating one
- Don't auto-generate READMEs for every directory

## Quality Bar

- **Copy-pasteable:** Quick Start commands should work when pasted directly
- **Current:** Every path, command, and env var mentioned should actually exist
- **Scoped:** Sub-module READMEs don't repeat root-level information
- **Embedding-friendly:** Include technology names, framework versions, and key concepts
- **Concise:** A README that reads like a novel has failed

## Common Mistakes

**Stale Quick Start**
- Problem: README says `npm start` but project uses `pnpm dev`
- Fix: Always verify commands against `package.json` or equivalent

**README for everything**
- Problem: Auto-generated README in every directory
- Fix: Only create READMEs where there's meaningful content to add

**Overwriting custom content**
- Problem: User had a "Deployment" section, update removed it
- Fix: Read first, preserve custom sections, update only templated ones

**Root README duplicating sub-READMEs**
- Problem: Root README explains how the API module works in detail
- Fix: Root gives overview, links to `packages/api/README.md` for details
