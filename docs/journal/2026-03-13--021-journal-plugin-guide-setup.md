# 021 — Journal plugin GUIDE.md setup

**Date:** 2026-03-13
**Scope:** Project tooling, CLAUDE.md cleanup

## Summary

Removed the inline journal section from `CLAUDE.md` and replaced it with the plugin-driven `@docs/journal/GUIDE.md` import. Created `docs/journal/GUIDE.md` with the standard journal plugin template.

## What changed

| File | Change |
|------|--------|
| `CLAUDE.md` | Removed the "## Journal" section (lines 138–144) that contained inline journal instructions |
| `CLAUDE.md` | Added `@docs/journal/GUIDE.md` import at the end of the file |
| `docs/journal/GUIDE.md` | Created with standard plugin template — commands, usage guidance, examples |

## Context

The previous journal setup embedded instructions directly in `CLAUDE.md`. The dev-journal plugin provides a `GUIDE.md` template that is imported via `@docs/journal/GUIDE.md`, keeping `CLAUDE.md` cleaner and letting the plugin manage its own documentation. This completes the journal init process started in entry 020.
