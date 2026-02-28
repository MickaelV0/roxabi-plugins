# claude-memory-audit

Claude Code plugin that drains auto-memory to zero. Every memory entry gets resolved — not just deleted, but moved to its proper permanent home.

## Philosophy

Claude Code's auto-memory (`MEMORY.md`) is an **inbox, not a knowledge base**. Entries accumulate as agents discover patterns, hit bugs, and learn project quirks. Left unchecked, memory becomes stale, redundant, and bloated.

This plugin treats every memory entry as a temporary holding zone. Each audit resolves every entry to one of four outcomes:

| Resolution | What it means |
|-----------|---------------|
| **Fix** | The entry exists because something is broken. Fix the root cause (code, config, docs), then delete the entry. |
| **Promote** | Durable insight that belongs in permanent docs. Move to CLAUDE.md, AGENTS.md, or docs/, then delete the entry. |
| **Relocate** | Domain-specific knowledge sitting in global memory. Move to the narrowest scope (subfolder CLAUDE.md, agent def, skill def), then delete. |
| **Delete** | Ephemeral (issue numbers, PR refs, worktree paths), stale (references files that no longer exist), or redundant (already in permanent docs). Just delete. |

**Target: memory = 0 after every audit.**

## Recurrence Detection

The plugin maintains an append-only audit log (`.claude/memory-audit-log.md`). When the same type of entry keeps reappearing across audits, it flags it:

- **2nd occurrence** — the previous fix didn't stick. Investigate why (wrong target? agents not reading it? docs unclear?).
- **3rd+ occurrence** — systemic gap. The permanent home is broken. Create an issue to fix the root cause.

This creates a feedback loop: recurring memory entries expose gaps in your documentation, agent prompts, or processes.

## Auto-Discovery

The plugin auto-detects your project structure — no configuration needed:

- Scans for `CLAUDE.md` files (root + subfolders)
- Finds `AGENTS.md` if it exists
- Discovers `.claude/agents/*.md` and `.claude/skills/*/SKILL.md`
- Locates documentation directories (`docs/`, `doc/`, `documentation/`)
- Scans both project memory (`~/.claude/projects/`) and agent memory (`.claude/agent-memory/`)

## Install

```bash
claude plugin add /path/to/claude-memory-audit
```

Or add to your project's `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "memory-audit@your-marketplace": true
  }
}
```

## Usage

```
/memory-audit           # Full audit
/memory-audit --dry-run # Preview without changes
```

Or use natural language triggers:
- "audit memory"
- "clean memory"
- "prune memory"

## What Gets Scanned

| Source | Path | Description |
|--------|------|-------------|
| Project memory | `~/.claude/projects/<project>/memory/MEMORY.md` | Main memory file (first 200 lines injected every session) |
| Topic files | `~/.claude/projects/<project>/memory/*.md` | Additional memory files loaded on demand |
| Agent memory | `.claude/agent-memory/*/MEMORY.md` | Per-agent memory (frontend-dev, backend-dev, etc.) |

## Audit Flow

```
Scan sources (δ + α)
       ↓
  Parse into entries (ε)
       ↓
  Check audit log for recurrences
       ↓
  Auto-discover placement targets (Π)
       ↓
  Classify each ε → Fix / Promote / Relocate / Delete
       ↓
  Present resolution plan → user approves
       ↓
  Execute resolutions
       ↓
  Append to audit log
       ↓
  Verify zero
```

## License

MIT
