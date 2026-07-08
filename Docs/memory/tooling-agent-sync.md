---
name: tooling-agent-sync
description: source-dev agent + write-source/test-source skills scaffold, fix, and verify sources; mirrors Reader's own agent-over-skills ruling; requires manual sync with Reader's copy
metadata:
  type: project
---

`buny-sources/.claude/agents/source-dev.md` writes new sources, fixes broken selectors, and verifies existing ones. `buny-sources/.claude/skills/write-source/` and `.../test-source/` are thin slash-command dispatchers with no logic of their own — they hand off to the agent.

**Why one agent instead of skills doing the work:** this mirrors a ruling Reader's own project memory already recorded (`Reader/Reader/Docs/memory/skills-vs-agents-rule.md`, 2026-07-03/07): they originally planned two skills (`/test-source`, `/write-source`) and shipped one agent instead, because writing a source means WebFetching live HTML and running cargo builds against a different repo — noisy, multi-step, cross-repo work that doesn't fit a lightweight skill. Don't resurrect a two-skill design that reimplements the agent's steps.

**Why this repo has its own copy instead of just using Reader's:** an equivalent `source-dev` agent already existed at `Reader/.claude/agents/source-dev.md`, but agents are scoped per-repo — invoking Reader's from a buny-sources session isn't possible. buny-sources' copy was authored 2026-07-07, adapted from Reader's (not copied verbatim): fixed a stale method name (`get_novel_details` → the real `get_novel_update`), added awareness of the `madtheme` shared-theme crate (introduced after Reader's agent was originally written — see [[repo-gotchas]]), and replaced a build-only "done" check with the actual two-gate CI contract (clippy zero-warnings + `buny verify`).

**Sync obligation:** there is no automated sync between the two copies. When `source-dev.md` changes in one repo for a reason that isn't repo-specific (a trait signature fix, a new pattern, a corrected CI gate), apply the same fix to the other's copy by hand. Reader's copy carries a one-line pointer back to this note.
