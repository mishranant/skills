# Claude Code Skills

A small collection of reusable [Claude Code](https://docs.claude.com/en/docs/claude-code) skills. Drop a skill folder into `~/.claude/skills/` (or a plugin's `skills/` directory) to make it available.

## Skills

### `inside-the-vm-debug`
A structured methodology for debugging a live issue you can't reproduce locally, where the evidence lives on a system you can't touch directly (a VM / host / pod / cluster node). Works in two access modes — interactive shell (an operator pastes command output) or telemetry-only (the agent drives read-only metrics, logs, dashboards, and config-at-deployed-commit). Same spine either way: **Collect → Diagnose → Plan → Guide**, with pre-committed decision-tree "tracks" so each step is bound in advance to its expected results.

### `dejargon`
Rewrites jargon-dense internal text into plain but still-technical engineering language a newcomer can follow — without dumbing down the substance. Extracts acronyms / internal tool & team names, resolves them against your org's glossary, docs, chat, and code search, then emits a de-jargoned rewrite plus a sourced glossary. No analogies, ever — literal mechanism only.

The `dejargon/references/glossary.md` file is a **template**: fill in your own org's stable terms.

## License

MIT
