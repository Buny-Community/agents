---
name: tooling-agent-sync
description: source-dev agent + write-source/test-source skills for scaffolding, fixing, and verifying sources; agent-based design for multi-step, cross-repo work
metadata:
  type: project
---

`agents/source-dev.md` (this repo, `Buny-Community/agents`) writes new sources, fixes broken selectors, and verifies existing ones. `skills/write-source/`, `.../test-source/`, and `.../doctor-source/` are thin slash-command dispatchers with no logic of their own — they hand off to the agent. This repo is packaged as a Claude Code plugin (`.claude-plugin/plugin.json` + `marketplace.json`) and installed into a buny-sources checkout via `claude plugin install source-dev@buny-agents` — it is not copied into buny-sources' own `.claude/`.

Three skills map to three modes of the same agent, not three separate implementations: `write-source` scaffolds a new source, `test-source` verifies an existing one builds/lints/packages (no selector-vs-live-markup diffing, no fixing), `doctor-source` (added 2026-07-17) diagnoses selector drift against the live site and fixes what it finds broken. The distinction between `test-source` and `doctor-source` matters because `buny verify`/`cargo build` prove the wasm compiles, not that a selector still matches anything — this repo has no HTML fixtures, so live-markup diffing is the only thing that actually catches drift.

**Why one agent instead of skills doing the work:** writing a source means WebFetching live HTML and running cargo builds against a different repo — noisy, multi-step, cross-repo work that doesn't fit a lightweight skill. An agent can autonomously navigate this complexity without user interruption. Don't migrate this to a pure-skill design without understanding why it exists as an agent.
