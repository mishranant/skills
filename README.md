# Claude Code Skills

A small collection of reusable [Claude Code](https://docs.claude.com/en/docs/claude-code) skills. Drop a skill folder into `~/.claude/skills/` (or a plugin's `skills/` directory) to make it available.

## Skills

### `inside-the-vm-debug`
A structured methodology for debugging a live issue you can't reproduce locally, where the evidence lives on a system you can't touch directly (a VM / host / pod / cluster node). Works in two access modes — interactive shell (an operator pastes command output) or telemetry-only (the agent drives read-only metrics, logs, dashboards, and config-at-deployed-commit). Same spine either way: **Collect → Diagnose → Plan → Guide**, with pre-committed decision-tree "tracks" so each step is bound in advance to its expected results.

### `dejargon`
Rewrites jargon-dense internal text into plain but still-technical engineering language a newcomer can follow — without dumbing down the substance. Extracts acronyms / internal tool & team names, resolves them against your org's glossary, docs, chat, and code search, then emits a de-jargoned rewrite with every term glossed inline, by function, at its first load-bearing use. Uses real teaching structure — problem-ladders, worked examples, question-beats — under a strict provenance rule: every sentence is source, a cited lookup, or visibly-marked scaffolding, and scaffolding may never supply a fact the source lacks. Comparisons may only borrow from technology the reader already knows (Kubernetes, Terraform, git); no household analogies, no ELI5. Missing motivation is routed rather than guessed: sourced "why" in prose, standard-pattern "why" in a quarantined industry-context block flagged for confirmation, load-bearing unknowns as targeted hypotheses for a reviewer. Organization-specific motivation is never supplied.

The `dejargon/references/glossary.md` file is a **template**: fill in your own org's stable terms.

## License

MIT
