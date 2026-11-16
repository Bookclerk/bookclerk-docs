# bookclerk_demo_seed::covers

Baking the covers a new demo session shows into the image's cover cache.

A visitor's first pages (the Discover shelves they land on and the
Library) request a cover for every title they show. Without a baked
cache, each one is downloaded through the `covers` plugin and decoded
by the media worker while the visitor waits, on a container with a
quarter of a CPU. The bake warms those covers instead, in two steps
(`packaging/demo/bake-seed.sh`):

1. With bookclerkd stopped, `--list-covers <file>` reads the library and
   writes the cover URLs those pages can show ([`cover_urls`](#cover_urls)): each held
   work's cover, then the covers of the store listings Discover ranks
   from (the cached store calls and the stored candidates), then every
   stored work cover, at most [`MAX_BAKED_COVERS`](#max_baked_covers).
2. With bookclerkd running again, `--bake-covers <file> <base-url>` signs
   each URL with the bake's cover key and requests it from bookclerkd
   ([`warm`](#warm)), which downloads and processes it into `cache/covers/`.

The cache is keyed by the image URL, not by the signing key, so the
entries stay valid after `--bake-finish` deletes the key and each
container makes its own.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WarmReport`](#warmreport) | struct | What warming the covers did. |
| [`cover_urls`](#cover_urls) | function | The cover URLs a new session's first pages can show, best first, without duplicates and at most `MAX_BAKED_COVERS`. |
| [`list_text`](#list_text) | function | The URL list file's text: one URL per line. |
| [`parse_list`](#parse_list) | function | The URLs in a list file's text, skipping blank lines. |
| [`read_key`](#read_key) | function | Reads the cover cache's signing key that bookclerkd made for this bake. |
| [`warm`](#warm) | function | Requests each cover in `urls` from bookclerkd at `base`, so it downloads and caches them. |
| [`MAX_BAKED_COVERS`](#max_baked_covers) | constant | Most covers a bake warms. The seeded library and the store calls Discover caches name about 300; the cap keeps a larger seed from growing the image without bound (each cover is three JPEGs, about 100 KiB together). |
| [`WARM_BUDGET`](#warm_budget) | constant | Longest the whole warm may take; covers not reached by then are left for a container to fetch. |

## Structs

### WarmReport

```rust
pub struct WarmReport {
    pub cached: usize,
    pub failed: usize,
    pub skipped: usize,
}
```

What warming the covers did.

**Fields**

- `cached`: Covers bookclerkd served, and so cached.
- `failed`: Covers that failed (a download refused or an image that would not
  decode); a container tries them again.
- `skipped`: Covers not requested before [`WARM_BUDGET`](#warm_budget) ran out.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### cover_urls

```rust
pub async fn cover_urls(store: &LibraryStore) -> Result<Vec<String>>
```

The cover URLs a new session's first pages can show, best first, without
duplicates and at most [`MAX_BAKED_COVERS`](#max_baked_covers).

#### Arguments

* `store` - The seeded library, with its page index and Discover's
  cached store calls written.

#### Returns

Absolute `https` URLs, in the order: held works' covers (as the Library
page shows them), covers of the listings Discover's cached store calls
and stored candidates name, then any other stored work cover.

#### Errors

Returns an error when a library read fails.

### list_text

```rust
pub fn list_text(urls: &[String]) -> String
```

The URL list file's text: one URL per line.

### parse_list

```rust
pub fn parse_list(text: &str) -> Vec<String>
```

The URLs in a list file's text, skipping blank lines.

### read_key

```rust
pub fn read_key(cache_dir: &Path) -> Result<TokenKey>
```

Reads the cover cache's signing key that bookclerkd made for this bake.

#### Arguments

* `cache_dir` - The files dir's cache dir (`paths.cache_dir`).

#### Errors

Returns an error when `covers/token.key` is missing or is not 32 bytes:
bookclerkd has not run yet, or another program wrote it.

### warm

```rust
pub fn warm(
    base: &str,
    key: &TokenKey,
    bearer: Option<&str>,
    urls: &[String],
    budget: Duration,
) -> WarmReport
```

Requests each cover in `urls` from bookclerkd at `base`, so it downloads
and caches them.

Best effort: a failed cover is counted, never an error, since the demo
works without it.

#### Arguments

* `base` - bookclerkd's address, such as `http://127.0.0.1:8787`.
* `key` - The cover signing key bookclerkd uses ([`read_key`](#read_key)).
* `bearer` - The operator token, sent as `Authorization: Bearer`, when
  the bake has one.
* `urls` - The cover URLs ([`cover_urls`](#cover_urls)).
* `budget` - Longest the whole warm may take.

## Constants

### MAX_BAKED_COVERS

```rust
pub const MAX_BAKED_COVERS: usize = 400;
```

Most covers a bake warms. The seeded library and the store calls
Discover caches name about 300; the cap keeps a larger seed from
growing the image without bound (each cover is three JPEGs, about
100 KiB together).

### WARM_BUDGET

```rust
pub const WARM_BUDGET: Duration = _;
```

Longest the whole warm may take; covers not reached by then are left
for a container to fetch.
