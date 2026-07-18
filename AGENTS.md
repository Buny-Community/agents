# buny-sources repo context

`buny-sources` hosts "sources" — individual Rust-to-WebAssembly scrapers for novel websites — that
plug into the Buny reading ecosystem. Full pipeline details: this repo's `Docs/Architecture/source-pipeline.md`
(`Docs/` lives in `Buny-Community/agents`, not in `buny-sources` itself — see below).

This file is background for anyone working in `buny-sources` via the `source-dev` plugin. A Claude
Code plugin can't inject a project's `CLAUDE.md`, so this file exists for the `write-source`/
`test-source`/`doctor-source` skills (and you) to read on demand instead of asking every consumer
to merge this into their own `buny-sources/CLAUDE.md`. The essentials `source-dev` needs on every
invocation are already folded into `agents/source-dev.md`; this is the fuller picture for anything
that agent doesn't already cover.

## Stack

- Rust, every crate targets `wasm32-unknown-unknown`, `crate-type = ["cdylib"]`
- No Cargo workspace — each `sources/<id>/` is an independent crate
- HTML/HTTP via the `buny` crate's FFI wrappers (`Request::get(url)?.html()?`, jQuery-style CSS
  `.select()`), not `scraper`/`select.rs`
- No unit tests or HTML fixtures in this repo — correctness is validated live

## Repo layout

- `sources/<id>/` — one crate per website (e.g. `en.royalroad`, `en.novelbuddy`). `id` convention:
  `{languageCode}.{name}`.
- `templates/madtheme/` — shared theme crate for sites on a common CMS pattern; several sources
  depend on it by path instead of implementing `Source` from scratch.
- `.github/workflows/` — `clippy.yaml` (lint gate), `pr.yaml` (package + verify gate), `build.yaml`
  (publishes the combined source index to `gh-pages` on push to `main`).

`buny-sources` itself has no `Docs/` directory. The pipeline/architecture and decision-log docs
referenced throughout this file (`Docs/Architecture/`, `Docs/memory/`) live in this repo,
`Buny-Community/agents`, alongside `AGENTS.md` — not inside `buny-sources`.

## Adding or fixing a source

Use the `/write-source <url>`, `/test-source <id>`, or `/doctor-source <id>` skills — they dispatch
to the `source-dev` agent (see `Buny-Community/agents`' README for install instructions). Don't
hand-scaffold: the agent knows the current `Source` trait signatures (they drift — read them live
from `buny-rs`, don't trust memory), the madtheme-vs-custom decision, and both CI gates below.

## Build / lint / verify (run inside a source dir)

```
cargo build --target wasm32-unknown-unknown --release   # build
cargo clippy                                              # lint — must be zero warnings
buny package                                               # bundle to package.bunpack
buny verify package.bunpack                                 # validate manifest + icon
```

## Related repos

| Repo | Role |
|---|---|
| buny-rs | `Source` trait, `register_source!` macro, `buny` CLI. Defines the Rust framework for building WASM sources. |

## Hard rules

- Never copy `buny-rs/examples/example-source` verbatim — it's stale (missing a parameter,
  references a removed type). Use
  `buny-rs/crates/cli/src/supporting/templates/source-lib.rs.template` instead.
- `sources/en.novelbin` is an empty stub (gitignored build artifacts only, nothing committed) —
  don't treat it as a reference example. (`sources/en.novelarchive` was a stub too as of
  2026-07-07, but is now a real, complete source — don't trust this list without checking `git ls-files
  sources/<id>` first, since stub status changes as sources get written.)
- A source isn't done when it builds — it needs zero `cargo clippy` warnings **and** a passing
  `buny verify`.
- Read the live `Source` trait at `buny-rs/crates/lib/src/structs/source.rs` before writing
  signatures.
- The `source-dev` agent scaffolds, fixes, and verifies sources. See this repo's
  `Docs/memory/tooling-agent-sync.md` for notes on multi-repo coordination of related tools.
- This tooling lives in a separate repo (`Buny-Community/agents`) distributed as a Claude Code
  plugin. Don't recreate `agents/source-dev.md` or the `skills/` inside `buny-sources` itself.
