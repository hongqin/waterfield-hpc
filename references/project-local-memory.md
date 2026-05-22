# Project-Local Memory Management

Project-level memory lives in the project root `memory/` directory (e.g., `~/2026-06-05-R01-viralGPT/memory/`), separate from Claude's auto-memory at `~/.claude/projects/*/memory/`.

## Purpose

Project-local memory stores durable project facts (budget numbers, personnel, constraints, aim structure) that persist across Claude sessions AND are version-controlled with git. Auto-memory is session-scoped and not committed to the repo.

## Files

| File | Content |
|---|---|
| `MEMORY.md` | Index of all memory files (one line each) |
| `project_viralGPT_R01.md` | Authoritative project metadata: title, PI, aims, consultant rates, budget envelope, PAR constraints |
| `data_inventory.md` | Post-reorg data paths |
| `session_*.md` | Session logs with completed work |
| `agent_*.md` | Agent handoff records |
| Other `*.md` | Topic-specific notes |

## Update Workflow

1. **Read the existing file** before editing — never overwrite blindly.
2. **Edit in place** — update the existing memory file rather than creating a new one for the same topic.
3. **Keep MEMORY.md in sync** — if you add or remove a file, update the index.
4. **Use frontmatter** on new files:
   ```markdown
   ---
   name: descriptive name
   description: one-line summary for relevance matching
   type: project | reference | user | feedback
   ---
   ```
5. **Commit memory changes** with the related code changes, not separately.

## When to Update

- Budget numbers change (rates, totals, personnel)
- Aim structure or scope changes
- New constraints discovered (PAR rules, ODURF policy)
- HPC job outcomes that affect next steps (handoff memos)
- Data reorganization or new data sources

## When NOT to Update

- Ephemeral task state (use Claude tasks instead)
- Code patterns derivable from reading the code
- Git history (use `git log`)
- Anything already in CLAUDE.md
