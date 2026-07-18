---
name: doctor-source
description: Diagnose whether an existing Buny source's selectors still match the live site, and fix any that have drifted (site redesign, dead selectors). Use when the user says "doctor source <id>", "check <site> selectors", "is <source> still working", or reports a source returning empty/wrong results.
---

Dispatch to the `source-dev` agent in **doctor mode** with the given `sources/<id>` path: extract every selector from the source's code, fetch the equivalent live page per selector group (search/novel/chapter — same Cloudflare fallback rules as scaffolding), and check each selector actually matches something sane in the current markup rather than just confirming the wasm builds.

This is different from `test-source`: `test-source`/`buny verify` only prove the source compiles and the manifest/icon are valid — they don't diff selectors against live markup, since this repo has no HTML fixtures. `doctor-source` is the one that actually re-checks selectors, and it fixes what it finds broken (a targeted patch to the drifted selector, not a rewrite), then re-runs the full definition of done (`cargo clippy`, `buny package && buny verify`, live exercise of the three core methods).

Do not reimplement any of the diagnosis, fix, or verify logic here — that lives entirely in `source-dev`'s doctor mode. This skill exists only for slash-command ergonomics.

Report back the per-selector table `source-dev` produces (method, old selector, live status, action taken), whether anything was fixed, and if it hit source-dev's ambiguous-drift stop-and-ask blocker.

Full buny-sources repo context (stack, layout, build/lint/verify, hard rules) beyond what `source-dev` already knows lives at `${CLAUDE_PLUGIN_ROOT}/AGENTS.md` — read it if you need something not already covered by the agent.
