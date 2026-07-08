# Source pipeline

Last verified against source: 2026-07-07.

## What this repo is

Each `sources/<id>/` directory is an independent Rust crate compiled to `wasm32-unknown-unknown` — a scraper for one novel website. There is no Cargo workspace; every source crate stands alone with its own `Cargo.toml`, `Cargo.lock`, and `.cargo/config.toml`. `templates/madtheme/` is a shared crate that several sources depend on by path when their site matches a common CMS pattern (see below).

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

**Custom** — a full hand-written `impl Source`. Reference: `sources/en.royalroad` (also `en.novel-fire`, `en.novelsonline`). File set: `Cargo.toml`, `.cargo/config.toml`, `res/{source.json,filter.json,icon.png}`, `src/lib.rs`, one `src/traits/<trait>.rs` per optional trait re-exported from `src/traits/mod.rs`.

**Theme-based** — for sites built on a specific CMS pattern (a Next.js `__NEXT_DATA__` script tag on novel pages plus a REST search endpoint), `templates/madtheme/src/imp.rs` provides a `trait Impl` with default-implemented scraping methods parametrized by `Params{base_url, api_url, novel_path, use_slug_search, default_rating, date_format}`. A matching source is then just:

```rust
struct NovelBuddy;
impl Impl for NovelBuddy {
    fn new() -> Self { Self }
    fn params(&self) -> Params { Params { base_url: "...".into(), api_url: "...".into(), ..Default::default() } }
}
register_source!(MadTheme<NovelBuddy>, ListingProvider);
```

Reference: `sources/en.novelbuddy` (the only current example, ~15 lines total). Prefer this pattern whenever the site fits — it means near-zero scraping code and any theme-wide fix (e.g. a CMS markup change) is made once in `templates/madtheme` rather than per source.

Two directories, `sources/en.novelarchive` and `sources/en.novelbin`, are empty gitignored stubs with no committed `Cargo.toml`/`src/` — not usable as reference examples.

## Manifest and package format

Each source ships `res/source.json` (required — `info.id/name/version/url/contentRating(0=safe,1=mature,2=nsfw)/languages`, `listings[]`), optionally `res/filter.json` (static search filters) and `res/settings.json`, and `res/icon.png` (required for verification — must be exactly 128×128 and fully opaque). `id` convention: `{languageCode}.{name}`.

`buny package` builds the wasm, renames it `main.wasm`, and zips `res/*` + `main.wasm` into `package.bunpack`. `buny build` flattens many `.bunpack` files into an `index.json` + `sources/`/`icons/` tree — the format `buny serve`/CI publishes for client apps to consume.

## Validation gates (CI)

A source isn't done when it compiles. Two independent gates, both enforced in `.github/workflows/`:

1. **`clippy.yaml`** — `cargo clippy` on every changed `sources/*`/`templates/*` dir must produce zero warnings.
2. **`pr.yaml`** — `buny package` then `buny verify` on the resulting `.bunpack` must both succeed. `verify` validates `source.json`/`filter.json`/`settings.json` against the JSON Schemas bundled in the `buny` CLI, and checks the icon dimensions/opacity.

`build.yaml` runs on push to `main`: builds all sources and publishes the combined index to `gh-pages`.

There are no unit tests or HTML fixtures anywhere in this repo — correctness is validated by building against live sites (via `buny-test-runner`, a native host that implements the same FFI surface as the app, backed by real `reqwest`/`scraper` calls) and by the two CI gates above.

## Cross-repo wire contract (why field order matters)

The WASM host has no shared IDL with buny-rs — the two sides agree on the ABI purely by convention:

- **Which exports exist** — `register_source!(Type, Trait1, Trait2, ...)` on the Rust side emits one `#[export_name]` function per trait; the host probes for those exact export names by string to build a feature set (duck typing, not declared capability).
- **How arguments/results are encoded** — Postcard, a positional (not field-name-keyed) wire format. Field order must match between the Rust structs in `buny-rs/crates/lib/src/structs/mod.rs` and the host's corresponding types for every shared type (`Novel`, `Chapter`, `ContentBlock`, `NovelPageResult`, `Listing`, `Filter`, `FilterValue`, `Setting`, `Home`).

This only matters when changing the *shape* of a shared struct (which isn't part of normal source-authoring work — sources just populate existing structs). It's called out here because nothing enforces the match except manual review, and a silent mismatch fails as a decode error at runtime, not a build error.
