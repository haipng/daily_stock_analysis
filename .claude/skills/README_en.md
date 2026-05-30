# Repository Claude Skills

This directory stores repository-level collaboration skills, which are version control assets.

- Single source of truth: Repository root directory `AGENTS.md`
- Compatible entry point: Root directory `CLAUDE.md` (should be a symlink to `AGENTS.md`)
- Skills in this directory must remain consistent with `AGENTS.md`
- `.claude/reviews/` contains local analysis artifacts, not used as rules source

If future compatibility with other agent directories is needed (e.g., `.agents/skills/` or `.github/skills/`), first clarify the single source of truth, then sync via scripts or mirroring, rather than manually maintaining multiple copies of the same content long-term.
