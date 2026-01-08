# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ralph is an autonomous AI agent loop that runs [Claude Code](https://claude.ai/code) repeatedly until all PRD items are complete. Each iteration spawns a fresh Claude Code instance with clean context. Memory persists only via git history, `progress.txt`, and `prd.json`.

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/).

## Commands

### Flowchart (React + Vite + TypeScript)

```bash
# Development server
cd flowchart && npm install && npm run dev

# Build
cd flowchart && npm run build

# Lint
cd flowchart && npm run lint

# Type check (runs as part of build)
cd flowchart && tsc -b
```

### Running Ralph

```bash
# Run Ralph with default 10 iterations
./ralph.sh

# Run with custom max iterations
./ralph.sh 20
```

Requires: Claude Code installed and authenticated, `jq` installed, git repository with `prd.json`.

## Architecture

### Core Loop (`ralph.sh`)

The bash script orchestrates the agent loop:
1. Archives previous runs when switching features (different `branchName` in `prd.json`)
2. Initializes/updates `progress.txt` for tracking
3. Pipes `prompt.md` to Claude Code with `-p --dangerously-skip-permissions`
4. Checks output for `<promise>COMPLETE</promise>` signal to exit
5. Loops until completion or max iterations reached

### Memory Model

Each Claude Code iteration is stateless. Context persists through:
- **Git history**: Commits from previous iterations
- **`prd.json`**: Tracks which stories are done (`passes: true/false`)
- **`progress.txt`**: Append-only log with learnings and patterns

### Skills System

Skills in `skills/` are skill definitions (SKILL.md files):
- **`skills/prd/`**: Generates PRDs via clarifying questions, outputs to `tasks/prd-[name].md`
- **`skills/ralph/`**: Converts markdown PRDs to `prd.json` format

### Flowchart (`flowchart/`)

Interactive React Flow visualization for presentations. Built with:
- React 19 + TypeScript
- Vite for build tooling
- @xyflow/react for the diagram
- Deployed to GitHub Pages via `.github/workflows/deploy.yml`

## Key Patterns

### Story Sizing

Stories must be small enough to complete in one context window. Right-sized:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic

Too big (split these):
- "Build the entire dashboard"
- "Add authentication"

### Story Ordering

Dependencies must be resolved before dependent stories:
1. Schema/database changes first
2. Backend logic second
3. UI components that use backend third
4. Aggregation/dashboard views last

### Acceptance Criteria

Must be verifiable, not vague. Always include:
- "Typecheck passes" for every story
- "Verify in browser" for UI stories

### AGENTS.md Updates

After completing work, update relevant `AGENTS.md` files with discovered patterns, gotchas, and conventions for future iterations.

### Stop Condition

When all stories have `passes: true`, output `<promise>COMPLETE</promise>` to signal completion.
