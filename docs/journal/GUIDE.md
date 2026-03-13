## Journal

This project maintains a development journal in `docs/journal/`.
Entries are named `YYYY-MM-DD--NNN-description.md` (e.g., `2026-03-13--004-fix-plugin-installation.md`).

### Commands

| Command | What it does |
|---------|-------------|
| `/journal` | Write a new entry — title is auto-generated from the conversation |
| `/journal <title>` | Write a new entry with an explicit title |
| `/journal search <query>` | Search entries by filename and content |
| `/journal last [N]` | Show the last N entries (default 1) |
| `/journal check` | Find entries relevant to the current conversation |
| `/journal init` | Set up the journal directory and CLAUDE.md @import |
| `/journal help` | Print command reference |

### When to use the journal

- **Before starting work:** Run `/journal check` to surface past decisions relevant to the files or topics you're about to touch.
- **Before architectural decisions:** Run `/journal search <topic>` to see if the topic was discussed before.
- **After completing work:** Run `/journal` to capture what was done and why.

### Examples

```
/journal                          # auto-title from conversation
/journal Switched to monorepo     # explicit title
/journal search auth              # find past auth decisions
/journal last 3                   # review last 3 entries
/journal check                    # what's relevant right now?
```
