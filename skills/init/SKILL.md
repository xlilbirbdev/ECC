---
name: init
description: Initialize a new CLAUDE.md file with codebase documentation. Explores the project, extracts conventions, stack details, and key workflows, then writes a CLAUDE.md that gives future Claude Code sessions immediate context without re-exploration.
origin: claude-code-harness
tools: Read, Write, Bash, Glob, Grep
---

# Init

Initialize a new `CLAUDE.md` file with accurate, project-specific documentation. Explores the codebase to extract the real stack, conventions, and workflows — not a template.

## When to Use

- Project has no `CLAUDE.md` yet
- User asks to "initialize" or "set up" CLAUDE.md
- Existing CLAUDE.md is stale and needs a full rewrite
- Onboarding a new project into Claude Code

**Do not use if:**
- CLAUDE.md already exists and just needs small updates — edit it directly
- The project already has detailed CLAUDE.md content — avoid overwriting good documentation

## What CLAUDE.md Is For

`CLAUDE.md` gives Claude Code immediate context at session start without re-exploring the codebase every time. Good CLAUDE.md content:
- States what the project does and why
- Documents the stack (language, framework, package manager, test runner)
- Lists key commands (how to run, test, build, lint)
- Captures non-obvious conventions (naming, module structure, gotchas)
- Points to where things live

Bad CLAUDE.md content:
- Generic advice Claude already knows (e.g., "write clean code")
- Out-of-date information
- Long explanations of things visible in the code

## Discovery Workflow

### 1. Identify the Stack

```bash
# Language and package manager
ls package.json pyproject.toml Cargo.toml go.mod pom.xml build.gradle 2>/dev/null

# Framework
cat package.json | grep -E '"next|"react|"vue|"express|"fastapi|"django'
cat pyproject.toml | grep -E 'fastapi|django|flask|sqlalchemy'

# Test runner
cat package.json | grep -E '"jest|"vitest|"mocha|"playwright'
```

### 2. Find Key Commands

```bash
# npm scripts
cat package.json | grep -A20 '"scripts"'

# Makefile targets
cat Makefile | grep '^[a-z]'

# Python entry points
cat pyproject.toml | grep '\[tool.taskipy\]\|\[tool.poe\]' -A10
```

### 3. Understand the Directory Structure

```bash
find . -maxdepth 2 -type d | grep -v node_modules | grep -v .git | grep -v __pycache__
```

### 4. Extract Conventions

Look for:
- File naming patterns (kebab-case, camelCase, PascalCase)
- Import style (absolute vs relative, barrel files)
- Test file location and naming
- Environment variable usage (`.env.example`)
- Existing CONTRIBUTING.md or docs/ folder

### 5. Identify Non-Obvious Gotchas

Read recent git history for clues:
```bash
git log --oneline -20
git log --oneline --all | grep -i "fix\|hotfix\|workaround" | head -10
```

## CLAUDE.md Template

```markdown
# CLAUDE.md

## Project Overview

[One paragraph: what this project does and who uses it]

## Stack

- **Language**: [e.g., TypeScript 5, Python 3.11]
- **Framework**: [e.g., Next.js 14, FastAPI]
- **Package manager**: [npm / pnpm / yarn / pip / uv]
- **Test runner**: [Jest / Vitest / pytest / go test]
- **Database**: [if applicable]

## Key Commands

\`\`\`bash
# Install
npm install

# Run dev server
npm run dev

# Run tests
npm test

# Build
npm run build

# Lint
npm run lint
\`\`\`

## Directory Structure

\`\`\`
src/
  components/    # React components
  lib/           # Shared utilities
  api/           # API route handlers
tests/           # Test files (mirror src/ structure)
\`\`\`

## Conventions

- [File naming: kebab-case for files, PascalCase for components]
- [Imports: absolute paths via tsconfig paths aliases]
- [Tests: co-located next to source files as *.test.ts]

## Non-Obvious Notes

- [Any gotchas, workarounds, or environment requirements]
- [External dependencies that need manual setup]
```

## Quality Checks

Before writing CLAUDE.md, verify:
- All commands in "Key Commands" actually work
- Directory structure reflects the real layout (not aspirational)
- No generic advice — every line should be project-specific

After writing, read it back and ask: "Would a developer new to this project find this useful in their first 5 minutes?"

## Related Skills

- `update-config` — Configure `.claude/settings.json` hooks and permissions
- `configure-ecc` — Set up ECC plugin configuration
