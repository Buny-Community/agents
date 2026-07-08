---
name: write-source
description: Create a new Buny source (Rust-to-WASM novel-site scraper) from a website URL. Use when the user says "add a source for <url>", "write a source for <site>", or similar.
---

Dispatch to the `source-dev` agent with the given site URL (and any hints the user gave — desired source id, language, known filters). Ask the agent to scaffold, build, lint, and verify the source end to end, autonomously, only stopping to ask the user if it hits one of `source-dev`'s documented blockers (Cloudflare/login walls, no obtainable icon, ambiguous site structure, disk space).

Do not re-derive selectors, scaffold files, or run cargo/buny commands yourself here — that logic lives entirely in `source-dev`. This skill exists only for slash-command ergonomics.

Report back what the agent produced (source id, file set, verify status) or what it's blocked on.
