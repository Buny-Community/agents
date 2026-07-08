---
name: repo-gotchas
description: Non-obvious traps in this repo and its sibling repos found while building the source-authoring tooling — stale reference example, empty stub source dirs, disk space, madtheme's origin
metadata:
  type: project
---

**`buny-rs/examples/example-source` is stale — don't copy it.** As of 2026-07-07 it calls `get_novel_update` without the `page: i32` parameter that's now part of the `Source` trait, and references a `NovelWithChapter` type that no longer exists anywhere in `buny-rs/crates/lib`. `git log` on `buny-rs` shows the trait file was updated in later commits than the example. Use `buny-rs/crates/cli/src/supporting/templates/source-lib.rs.template` as the scaffold instead — it's what `buny init` actually generates and stays current with the trait by construction.

**`sources/en.novelarchive` and `sources/en.novelbin` are not real examples.** Both are untracked/gitignored leftovers — only `Cargo.lock` and generated `public/` build artifacts exist on disk, no `Cargo.toml` or `src/` is committed to git. Don't read them expecting a working source; don't scaffold new sources by copying them.

**`madtheme` postdates the original source-dev agent.** `sources/en.novelbuddy` was rewritten to depend on the shared `templates/madtheme` crate in commit `1a6c3b2` ("fix novelbuddy (new structure)") — before that, every source was hand-written custom. When the `madtheme` pattern was introduced, the `source-dev` agent was updated to document it. If madtheme gets extended to cover more CMS patterns, or a second shared theme crate appears, update the "pick a pattern" section in `source-dev.md` (see [[tooling-agent-sync]]).

**Disk is chronically near-full on this machine.** If a `cargo build` inside `buny-rs`/`buny-sources` hits `ENOSPC`, `cargo clean` inside those repos is safe and expected; don't touch anything outside them to free space.

**Two identical repo checkouts exist for `buny-rs` and `buny-sources`** (`the local checkout of...` and `the local checkout of...`) — confirmed byte-identical as of 2026-07-07. Agents/tooling reference both paths interchangeably; if they ever diverge, that's a signal something was edited in one checkout and not synced to the other.
