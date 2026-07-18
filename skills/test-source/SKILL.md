---
name: test-source
description: Verify an existing Buny source still builds, lints clean, and passes buny verify. Use when the user says "test source <id>", "check if <site> still works", or after editing an existing source under sources/*.
---

Dispatch to the `source-dev` agent with the given `sources/<id>` path, framed as verify-only: skip scaffolding, go straight to `cargo clippy` (zero warnings), `buny package && buny verify`, and a live exercise of the three core methods (search, novel update, chapter content).

Do not reimplement any of the build/lint/verify steps here — that logic lives entirely in `source-dev`. This skill exists only for slash-command ergonomics.

Report back pass/fail per gate, and if it fails, what `source-dev` diagnosed as the cause (e.g. a selector broken by a site redesign).

Full buny-sources repo context (stack, layout, build/lint/verify, hard rules) beyond what `source-dev` already knows lives at `${CLAUDE_PLUGIN_ROOT}/AGENTS.md` — read it if you need something not already covered by the agent.
