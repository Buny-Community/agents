---
name: tooling-agent-sync
description: source-dev agent + write-source/test-source skills for scaffolding, fixing, and verifying sources; agent-based design for multi-step, cross-repo work
metadata:
  type: project
---

`buny-sources/.claude/agents/source-dev.md` writes new sources, fixes broken selectors, and verifies existing ones. `buny-sources/.claude/skills/write-source/` and `.../test-source/` are thin slash-command dispatchers with no logic of their own — they hand off to the agent.

**Why one agent instead of skills doing the work:** writing a source means WebFetching live HTML and running cargo builds against a different repo — noisy, multi-step, cross-repo work that doesn't fit a lightweight skill. An agent can autonomously navigate this complexity without user interruption. Don't migrate this to a pure-skill design without understanding why it exists as an agent.
