# feat: Migrate Ralph from Amp CLI to Claude Code CLI

## Overview

Adapt Ralph to work with Claude Code CLI instead of Amp CLI. This is a focused migration: change CLI invocation, remove Amp-specific features, and update documentation.

## Acceptance Criteria

- [ ] `ralph.sh` invokes Claude Code instead of Amp
- [ ] Ralph completes a 2-story PRD end-to-end with Claude Code
- [ ] All "Amp" references replaced with "Claude Code" in documentation
- [ ] Amp-only features (thread URLs, read_thread) removed

## Feature Losses (Acknowledged)

**Thread URL tracking:** Amp exposed `$AMP_CURRENT_THREAD_ID` for linking progress logs to conversation history. Claude Code doesn't have an equivalent. Progress logs will no longer include conversation links. Git history and `progress.txt` content remain the primary context for future iterations.

**Auto-handoff config:** Amp's `amp.experimental.autoHandoff` setting is removed. Claude Code handles context limits differently (automatically, without configuration).

---

## Essential Changes

### 1. `ralph.sh` - CLI Invocation (THE critical change)

In the main loop where Amp is invoked:

```bash
# Before
OUTPUT=$(cat "$SCRIPT_DIR/prompt.md" | amp --dangerously-allow-all 2>&1 | tee /dev/stderr) || true

# After
OUTPUT=$(cat "$SCRIPT_DIR/prompt.md" | claude -p --dangerously-skip-permissions 2>&1 | tee /dev/stderr) || true
```

### 2. `prompt.md` - Remove Dead References

**Progress Report Format section:**
- Remove the `Thread: https://ampcode.com/threads/$AMP_CURRENT_THREAD_ID` line
- Remove the sentence about using `read_thread` tool to reference previous work

**Browser Testing section:**
```markdown
# Before
1. Load the `dev-browser` skill
2. Navigate to the relevant page
...

# After
1. Use browser tools to navigate to the relevant page
2. Take a snapshot or screenshot to verify the UI changes
3. Interact with the page to confirm functionality works as expected
```

### 3. Documentation - Find and Replace

**All documentation files (README.md, AGENTS.md, CLAUDE.md):**
- Replace "Amp" with "Claude Code" throughout
- Replace `https://ampcode.com` links with `https://claude.ai/code`
- Replace `https://ampcode.com/manual` with `https://docs.anthropic.com/en/docs/claude-code`

**README.md specific:**
- Update prerequisites: "Claude Code CLI installed and authenticated"
- Remove the auto-handoff config section entirely (the `amp.experimental.autoHandoff` JSON block)
- Update "dev-browser skill" references to "browser tools"

### 4. `flowchart/` - Update Titles (MISSING FROM ORIGINAL PLAN)

**flowchart/index.html:**
```html
# Before
<title>How Ralph Works with Amp</title>

# After
<title>How Ralph Works</title>
```

**flowchart/src/App.tsx:**
- Change node label "Amp picks a story" to "Claude Code picks a story"
- Change heading "How Ralph Works with Amp" to "How Ralph Works"

### 5. Example Files - Update Acceptance Criteria

**prd.json.example and skills files:**
- Replace "Verify in browser using dev-browser skill" with "Verify in browser"

---

## Deferred (Out of Scope)

**Skills directory migration:** The plan originally suggested migrating `~/.config/amp/skills/` to `~/.claude/commands/`. However, Amp's SKILL.md format may not be compatible with Claude Code's custom commands format. This needs investigation before committing to a migration path.

For now: Keep skills in `skills/` directory within the project. Users can copy to their preferred location.

---

## Key Mappings

| Amp | Claude Code |
|-----|-------------|
| `amp --dangerously-allow-all` | `claude -p --dangerously-skip-permissions` |
| `$AMP_CURRENT_THREAD_ID` | removed (no equivalent) |
| `read_thread` tool | removed (no equivalent) |
| `dev-browser` skill | "browser tools" (MCP-based) |
| `~/.config/amp/skills/` | deferred - needs investigation |
| `amp.experimental.autoHandoff` | removed (handled automatically) |

---

## Files Changed

1. `ralph.sh` - CLI invocation
2. `prompt.md` - Thread references, browser instructions
3. `README.md` - Documentation, remove auto-handoff section
4. `AGENTS.md` - Documentation
5. `CLAUDE.md` - Documentation
6. `flowchart/index.html` - Title
7. `flowchart/src/App.tsx` - Title and node labels
8. `prd.json.example` - Acceptance criteria
9. `skills/prd/SKILL.md` - Browser references
10. `skills/ralph/SKILL.md` - Amp references, browser references

---

## Testing

**Before implementing:** Verify CLI works with piped input:
```bash
echo "Say hello" | claude -p --dangerously-skip-permissions
```

**After implementing:** Run Ralph end-to-end:
1. Create a simple 2-story `prd.json` (one backend, one with browser verification)
2. Run `./ralph.sh 5`
3. Confirm both stories complete with `passes: true`
4. Verify progress.txt has entries for both iterations
5. Verify git commits were created
