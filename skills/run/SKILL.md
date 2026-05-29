---
name: run
description: Launch and drive the project's app to see a change working in practice. Use when asked to run, start, or screenshot the app, confirm a change works in the real app (not just tests), or validate local changes before pushing.
origin: claude-code-harness
tools: Read, Bash, Glob
---

# Run

Launch and drive the project's application to see changes working in practice. Looks for a project-specific launch skill first, then falls back to standard patterns per project type.

## When to Use

- User asks to "run" or "start" the app
- User wants to "see the change working" or "confirm it works"
- User asks to screenshot or interact with the running app
- User wants to validate changes before pushing or opening a PR
- Testing a golden path or edge case in the live application

**Not a substitute for:**
- Automated tests — run those with `npm test`, `pytest`, etc.
- Type checking — run `tsc --noEmit` separately
- Code review — use `code-review` for diff analysis

## Launch Strategy

### 1. Check for a Project Skill

Look for a project-specific run skill:
```bash
ls .claude/skills/run* 2>/dev/null
cat CLAUDE.md | grep -A5 "run\|start\|launch"
```

If found, follow the project's documented launch instructions.

### 2. Detect Project Type and Launch

**Node.js / npm**
```bash
npm run dev        # or start, or serve
npx ts-node src/index.ts
node scripts/main.js
```

**Python**
```bash
python -m uvicorn app.main:app --reload   # FastAPI/Starlette
python app.py
python -m flask run
python manage.py runserver                # Django
```

**CLI tool**
```bash
# Build first if needed
npm run build
# Then run the CLI
node dist/index.js <command>
npx <tool-name> <command>
```

**Browser-based / frontend**
```bash
npm run dev        # Vite, Next.js, CRA
npm start
```

**Go**
```bash
go run main.go
go run ./cmd/server
```

**Rust**
```bash
cargo run
cargo run --bin <name>
```

### 3. Validate the App is Running

After launch, confirm:
- No startup errors in the output
- The expected port is listening: `curl -s http://localhost:3000/health`
- The process didn't immediately exit

### 4. Exercise the Change

Navigate to or invoke the specific feature that was changed:
- For web apps: make an HTTP request or open in browser
- For CLI tools: run the relevant subcommand
- For APIs: call the affected endpoint

### 5. Check for Regressions

After confirming the target change works, spot-check adjacent features:
- Related routes or commands
- Shared utilities the change touches
- Any feature that imports the modified file

## Reporting

After running the app:
- State whether the change works as expected
- Note any errors, warnings, or unexpected output
- List what was tested (golden path + any edge cases)
- If the UI cannot be tested (no display), say so explicitly rather than claiming success

## Common Issues

| Issue | Fix |
|-------|-----|
| Port already in use | `lsof -i :<port>` to find the process, then kill it |
| Missing environment variables | Copy `.env.example` to `.env` and fill in values |
| Dependencies not installed | Run `npm install` or `pip install -r requirements.txt` |
| Build step required | Check `package.json` scripts for a `build` step before `start` |

## Related Skills

- `verify` — Confirm a specific code change works correctly
- `code-review` — Review the diff before running
- `e2e-testing` — Full end-to-end test suite
