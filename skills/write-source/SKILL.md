---
name: write-source
description: Create a new Buny source (Rust-to-WASM novel-site scraper) from a website URL, optionally with a reference/helper file such as an LNReader plugin. Use when the user says "add a source for <url>", "write a source for <site>", or similar.
---

Dispatch to the `source-dev` agent with the given site URL and any hints the user gave — desired source id, language, known filters, and optionally a **reference** (helper file): a plugin from another reader ecosystem (usually an LNReader `.ts` plugin from `https://github.com/lnreader/lnreader-plugins/tree/master/plugins`), or a text file of known API/endpoint URLs. Pass a reference through as a URL or path; it's a lead, and `source-dev` verifies everything against the live site.

The reference is optional. Without one, `source-dev` first looks for a matching plugin in lnreader-plugins, and if there isn't one it surveys the site itself (including the Chrome network survey for client-rendered/API-backed sites). A missing reference is never a reason to stop.

Ask the agent to scaffold, build, lint, package, verify, and live-test the source end to end, autonomously, only stopping for `source-dev`'s documented blockers: a Cloudflare challenge needing human input even in a real Chrome tab, login walls, no obtainable icon, ambiguous site structure, the live test runner being unable to reach the network (firewall), or disk space.

If the current session can do the work directly (it has the Chrome tools and shell access the agent would use), it may follow `agents/source-dev.md` inline instead of spawning — the agent file is the procedure either way.

After the source passes, run a short retro before reporting. List what this site taught you that the procedure didn't cover: a scaffold defect, a probe that would have saved time, a test-runner or app divergence, a CMS family worth naming. Fold each item into `agents/source-dev.md` (procedure) or `Docs/memory/repo-gotchas.md` (dated facts), and propose any update to the buny-sources working notes. If the user asked for review, show them the diff rather than committing it. Skip anything already covered.

Report back what was produced (source id, file set, clippy/verify/live-test status, discrepancies between the reference and the live site) or what it's blocked on.

Full buny-sources repo context (stack, layout, build/lint/verify, hard rules) beyond what `source-dev` already knows lives at `${CLAUDE_PLUGIN_ROOT}/AGENTS.md`.
