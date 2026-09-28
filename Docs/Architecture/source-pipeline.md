# Source pipeline

Last verified against source: 2026-09-27.

## What this repo is

Each `sources/<id>/` directory is an independent Rust crate compiled to `wasm32-unknown-unknown` — a scraper for one novel website. There is no Cargo workspace; every source crate stands alone with its own `Cargo.toml`, `Cargo.lock`, and `.cargo/config.toml`.

This repo is one link in a pipeline:

1. **buny-rs** — the Rust framework. Defines the `Source` trait family, the `register_source!` macro, the host FFI wrappers (HTTP, HTML/CSS selectors, JS eval, KV storage), and the `buny` CLI (`init`/`package`/`build`/`serve`/`verify`/`logcat`).
2. **buny-sources** (this repo) — the actual site scrapers, compiled to `.bunpack` packages.

Further downstream, these packages are consumed by a WASM host that loads sources and exposes them to reader applications.

## The `Source` trait contract

Defined in `buny-rs/crates/lib/src/structs/source.rs` — always read it live before writing signatures, it changes over time and the bundled `buny-rs/examples/example-source` has drifted from it before (missing a parameter, referencing a removed type).

Core trait, required by every source:

```rust
pub trait Source {
    fn new() -> Self;
    fn get_search_novel_list(&self, query: Option<String>, page: i32, filters: Vec<FilterValue>) -> Result<NovelPageResult>;
    fn get_novel_update(&self, novel: Novel, needs_details: bool, needs_chapters: bool, page: i32) -> Result<Novel>;
    fn get_chapter_content_list(&self, novel: Novel, chapter: Chapter) -> Result<Vec<ContentBlock>>;
}
```

Optional extension traits (each `: Source`), implement only the ones the site supports: `ListingProvider` (home-page novel listings), `Home` (curated home layout), `DynamicListings`, `DynamicFilters`, `DynamicSettings`, `AlternateCoverProvider`, `BaseUrlProvider`, `NotificationHandler`, `DeepLinkHandler`, `BasicLoginHandler`, `WebLoginHandler`, `MigrationHandler`.

A single macro call at the bottom of `src/lib.rs` wires everything to WASM exports — nothing else needs to be hand-written:

```rust
register_source!(SourceStruct, ListingProvider, Home, DeepLinkHandler);
```

HTML/HTTP access goes through the `buny` crate's FFI wrappers, not `scraper`/`select.rs`: `Request::get(url)?.html()? -> Document`, then jQuery-style `.select(css)`, `.select_first(css)`, `.attr()`, `.text()` (SwiftSoup-backed on the host side).

## Two authoring patterns

**JSON API** — the site's own frontend calls a JSON API; the source calls the same endpoints with `Request::get(url)?.json_owned()?` (`buny` feature `json` + `serde_json` with `alloc`). Preferred whenever the site has one: stable field names instead of CSS selectors. References: `sources/en.chikari` (plain REST, paged chapters), `sources/en.novelbuddy` (Next.js `/_next/data` routes + a REST API, build-id refresh).

**HTML** — scrape rendered pages with `Request::get(url)?.html()?` and CSS selectors. Reference: `sources/en.royalroad` (also `en.novel-fire`, `en.novelsonline`, `en.novelfull`, `en.novelarchive`).

Both use the same file set: `Cargo.toml`, `.cargo/config.toml` (must include `rustflags = ["-C", "link-arg=--import-undefined"]`, which `buny init` currently omits), `res/{source.json,filter.json,icon.png}`, `src/lib.rs`, optionally one `src/traits/<trait>.rs` per optional trait re-exported from `src/traits/mod.rs`.

The former theme pattern (`templates/madtheme`, a `trait Impl` for a Next.js CMS) is gone: `en.novelbuddy` was rewritten as a standalone source in `f40332e`, and the template isn't tracked in git.

`sources/en.novelbin` is an empty gitignored stub — not a reference example.

## Manifest and package format

Each source ships `res/source.json` (required — `info.id/name/version/url/contentRating(0=safe,1=mature,2=nsfw)/languages`, `listings[]`), optionally `res/filter.json` (static search filters) and `res/settings.json`, and `res/icon.png` (required for verification — must be exactly 128×128 and fully opaque). `id` convention: `{languageCode}.{name}`.

`buny package` builds the wasm, renames it `main.wasm`, and zips `res/*` + `main.wasm` into `package.bunpack`. `buny build` flattens many `.bunpack` files into an `index.json` + `sources/`/`icons/` tree — the format `buny serve`/CI publishes for client apps to consume.

## Validation gates (CI)

A source isn't done when it compiles. Two independent gates, both enforced in `.github/workflows/`:

1. **`clippy.yaml`** — `cargo clippy` on every changed `sources/*`/`templates/*` dir must produce zero warnings.
2. **`pr.yaml`** — `buny package` then `buny verify` on the resulting `.bunpack` must both succeed. `verify` checks the wasm parses and exports the required functions, validates `source.json` (and `settings.json`) against the JSON Schemas bundled in the `buny` CLI, and checks the icon dimensions/opacity. It does **not** validate `filter.json`: `verify.rs` looks for `Payload/filters.json` while packages contain `filter.json` (a buny-rs bug as of 2026-09-27), so filters must be checked against `filters.schema.json` by hand.

`build.yaml` runs on push to `main`: builds all sources and publishes the combined index to `gh-pages`.

There are no unit tests or HTML fixtures anywhere in this repo — correctness is validated by building against live sites (via `buny-test-runner`, a native host that implements the same FFI surface as the app, backed by real `reqwest`/`scraper` calls) and by the two CI gates above.

## Cross-repo wire contract (why field order matters)

The WASM host has no shared IDL with buny-rs — the two sides agree on the ABI purely by convention:

- **Which exports exist** — `register_source!(Type, Trait1, Trait2, ...)` on the Rust side emits one `#[export_name]` function per trait; the host probes for those exact export names by string to build a feature set (duck typing, not declared capability).
- **How arguments/results are encoded** — Postcard, a positional (not field-name-keyed) wire format. Field order must match between the Rust structs in `buny-rs/crates/lib/src/structs/mod.rs` and the host's corresponding types for every shared type (`Novel`, `Chapter`, `ContentBlock`, `NovelPageResult`, `Listing`, `Filter`, `FilterValue`, `Setting`, `Home`).

This only matters when changing the *shape* of a shared struct (which isn't part of normal source-authoring work — sources just populate existing structs). It's called out here because nothing enforces the match except manual review, and a silent mismatch fails as a decode error at runtime, not a build error.
