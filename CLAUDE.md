# claude-buddy

A coding companion for Claude Code — animated buddy in the status line, driven by an MCP server.

## Stack

- **Runtime**: [bun](https://bun.sh) for all TypeScript (server + CLI)
- **MCP server**: `server/index.ts` — serves the `/buddy` slash commands and manages buddy state
- **Status line**: `statusline/combined-status.sh` — bash script injecting stats + buddy art into Claude Code's status line
- **Hooks**: `hooks/*.sh` — react to tool use, file edits, name mentions, mood

## Commands

```bash
bun test              # run test suite (56 tests, ~65 ms)
bun run typecheck     # tsc --noEmit
bun run server        # start MCP server manually
bun run install-buddy # install / reinstall into current Claude Code profile
bun run upgrade       # upgrade in place
bun run uninstall     # remove from Claude Code profile
bun run doctor        # diagnose install issues
bun run show          # print buddy card to terminal
bun run test-statusline  # render statusline for visual review
```

## Project layout

```
server/        TypeScript MCP server — state, art, engine, reactions, achievements
cli/           CLI tools — install, upgrade, doctor, show, etc.
statusline/    combined-status.sh + buddy-status.sh (bash, keep buddy-status.sh clean for upstream)
hooks/         PostToolUse / UserPromptSubmit / Stop hook scripts
scripts/       gen-emoji-widths.ts, snapshot-statusline.sh
```

## Status line JSON (Claude Code → combined-status.sh)

Claude Code sends this JSON on stdin every `refreshInterval` seconds:

```json
{
  "context_window": { "used_percentage": 28 },
  "model": { "id": "claude-sonnet-4-6", "display_name": "Sonnet 4.6" },
  "rate_limits": {
    "five_hour": { "used_percentage": 63, "resets_at": 1778602800 },
    "seven_day":  { "used_percentage": 11, "resets_at": 1779181200 }
  },
  "cost": { "total_cost_usd": 0.89 },
  "workspace": { "current_dir": "/home/user" }
}
```

`rate_limits` is only present for Claude Max subscribers. The combined script gracefully omits those bars when absent.

## Key files to know

| File | Purpose |
|------|---------|
| `server/engine.ts` | Deterministic buddy generation from user ID hash |
| `server/state.ts` | Read/write `buddy-state/status.json` (pre-renders all animation frames) |
| `server/art.ts` | ASCII art frames per species, hat overlays, blink logic |
| `statusline/combined-status.sh` | Merges ctx/5h/7d/model bars into buddy art output |
| `statusline/buddy-status.sh` | Pure buddy art renderer — keep untouched for upstream PRs |
| `hooks/buddy-comment.sh` | Extracts `<!-- buddy: ... -->` from Stop hook, writes to status |

## State directory

Lives at `$BUDDY_STATE_DIR` (default: `~/.claude/buddy-state/`).  
Key file: `status.json` — written by MCP server, read by statusline bash script.

## Tests

Co-located with source: `server/*.test.ts`. Run with `bun test`.  
Snapshot tests for statusline rendering: `scripts/snapshot-statusline.sh`.
