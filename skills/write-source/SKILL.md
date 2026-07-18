---
name: write-source
description: Create a new Buny source (Rust-to-WASM novel-site scraper) from a website URL. Use when the user says "add a source for <url>", "write a source for <site>", or similar.
---

Dispatch to the `source-dev` agent with the given site URL and any hints the user gave — desired source id, language, known filters, and optionally a **reference**: a plugin file from another reader ecosystem (e.g. an LNReader `.ts` source), or a plain text file listing known API/endpoint URLs. A reference is a lead for the agent, not something to trust blindly — `source-dev` still verifies everything against the live site. Ask the agent to scaffold, build, lint, and verify the source end to end, autonomously, only stopping to ask the user if it hits one of `source-dev`'s documented blockers (a Cloudflare challenge that needs human input even in a real Chrome tab, login walls, no obtainable icon, ambiguous site structure, disk space, or a site whose data is client-rendered/API-backed with no reference supplied). A Cloudflare wall alone is not a stop condition — `source-dev` falls back to a real Chrome session autonomously to get past headless-fetch failures. Discovering a *hidden API endpoint* is different: `source-dev` will not survey the site's network traffic on its own — it needs either a reference or the user explicitly telling it to go survey the site itself.

Do not re-derive selectors, scaffold files, or run cargo/buny commands yourself here — that logic lives entirely in `source-dev`. This skill exists only for slash-command ergonomics.

Report back what the agent produced (source id, file set, verify status) or what it's blocked on.

Full buny-sources repo context (stack, layout, build/lint/verify, hard rules) beyond what `source-dev` already knows lives at `${CLAUDE_PLUGIN_ROOT}/AGENTS.md` — read it if you need something not already covered by the agent.
