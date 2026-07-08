# Buny Sources

This repo hosts "sources" — individual Rust-to-WebAssembly scrapers for novel websites — that plug into the Buny reading ecosystem. Full pipeline details: `Docs/Architecture/source-pipeline.md`.

## Stack

- Rust, every crate targets `wasm32-unknown-unknown`, `crate-type = ["cdylib"]`
- No Cargo workspace — each `sources/<id>/` is an independent crate
- HTML/HTTP via the `buny` crate's FFI wrappers (`Request::get(url)?.html()?`, jQuery-style CSS `.select()`), not `scraper`/`select.rs`
- No unit tests or HTML fixtures in this repo — correctness is validated live (see Validation below)

## Repo layout

- `sources/<id>/` — one crate per website (e.g. `en.royalroad`, `en.novelbuddy`). `id` convention: `{languageCode}.{name}`.
- `templates/madtheme/` — shared theme crate for sites on a common CMS pattern; several sources depend on it by path instead of implementing `Source` from scratch.
- `.github/workflows/` — `clippy.yaml` (lint gate), `pr.yaml` (package + verify gate), `build.yaml` (publishes the combined source index to `gh-pages` on push to `main`).
- `Docs/Architecture/` — how the pipeline works end to end.
- `Docs/memory/` — decisions and gotchas that aren't obvious from reading the code.

## Adding or fixing a source

Use the `source-dev` agent (`.claude/agents/source-dev.md`), or the `/write-source <url>` and `/test-source <id>` skills, which dispatch to it. Don't hand-scaffold: the agent knows the current `Source` trait signatures (they drift — read them live from `buny-rs`, don't trust memory), the madtheme-vs-custom decision, and both CI gates below.

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

- Never copy `buny-rs/examples/example-source` verbatim — it's stale (missing a parameter, references a removed type). Use `buny-rs/crates/cli/src/supporting/templates/source-lib.rs.template` instead.
- `sources/en.novelarchive` and `sources/en.novelbin` are empty stubs (gitignored build artifacts only, nothing committed) — don't treat them as reference examples.
- A source isn't done when it builds — it needs zero `cargo clippy` warnings **and** a passing `buny verify`.
- Read the live `Source` trait at `buny-rs/crates/lib/src/structs/source.rs` before writing signatures.
- The `source-dev` agent scaffolds, fixes, and verifies sources. See `Docs/memory/tooling-agent-sync.md` for notes on multi-repo coordination of related tools.
