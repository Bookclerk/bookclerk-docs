# bookclerk_demo_seed

Sample data for the hosted demo's library.

A demo visitor lands in a library of their own, as its Owner. An empty
library shows little of what Bookclerk does with more than one person, so
before bookclerkd starts, the demo image runs this crate's binary once to
add a second library user, Sam, who:

- has an account at each storefront the image ships (Audible, Libro.fm,
  Chirp and GraphicAudio), with well-known audiobooks from each already in
  the library;
- keeps a wishlist;
- owns a branch, "sams-nas", that shared part of its catalog and then
  went offline (its last heartbeat is days old).

The titles are real audiobooks. Before seeding, the binary looks each one
up in its storefront's public catalog through the storefront plugin, the
search Discover uses, then fetches the matched title's full record (the
details call title dialogs use), and the seed writes the record the store
returned:
its product id, cover art, description and the rest, as a scan and
enrichment would. A title whose lookup fails (store unreachable, no
confident match) keeps the fields listed here under a made-up product id
prefixed `demo-`, so the demo still starts without the stores.

Everything is written through the library store and the branch catalog
code the daemon uses, never raw SQL, so the seed follows the schema of
the commit it is built from. Nothing here signs in: Sam has no password
and no claim ticket, the store accounts hold no tokens, and the branch's
credential keys are placeholders that never verify a signature.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Storefront`](#storefront) | struct | A storefront the demo image ships, by its plugin id (`plugin.toml` `id`). |
| [`Lookup`](#lookup) | struct | One title to look up in a storefront's public catalog. |
| [`SeedReport`](#seedreport) | struct | What `seed` wrote, for the log and for tests. |
| [`Seeded`](#seeded) | enum | Outcome of `seed`. |
| [`committed_records`](#committed_records) | function | The committed storefront records (`records.json`) for the current sample titles; entries for titles no longer sampled are dropped. |
| [`parse_records`](#parse_records) | function | Parses a records file: a JSON object from `Lookup::key` to a catalog record, keeping only keys of current sample titles. |
| [`merge_records`](#merge_records) | function | Live lookups over the committed records: a title the live lookup found keeps that record, and one it missed falls back to `committed` unless a live record already holds that product id, so a stale committed record never pushes out a fresh one. No two titles share a product id. |
| [`records_json`](#records_json) | function | `records` as the pretty-printed JSON `records.json` holds, one key per title in sorted order, so a refresh diffs cleanly. |
| [`lookups`](#lookups) | function | Every title the seed writes, as catalog lookups: the library, the branch and the wishlist. |
| [`search_query`](#search_query) | function | The catalog search text for `lookup`: the title up to any `(…)` part, then the author's surname. |
| [`best_match`](#best_match) | function | The search hit that is the wanted title by the wanted author, or `None`. |
| [`distinct_records`](#distinct_records) | function | `records` without entries whose product id another title already took, so two sample titles never become one book (the first, in `lookups` order, keeps it). |
| [`seed`](#seed) | function | Adds the sample library user, their books, wishlist and offline branch. |
| [`with_details`](#with_details) | function | Replaces a search hit with its storefront's full catalog record (the details call Discover makes for title dialogs), so a store whose search rows are lean, such as Libro.fm, still seeds a description, publisher, genres and price. |
| [`records_path`](#records_path) | function | The `--write-records <path>` argument the seed binary was given, if any. |
| [`StoreRecords`](#storerecords) | type alias | Storefront records found for the sample titles, by `Lookup::key`. |
| [`SAMPLE_LOGIN`](#sample_login) | constant | Login name of the sample library user; its presence marks a seeded library. |
| [`STOREFRONTS`](#storefronts) | constant | Every storefront Sam has an account at, in the order the accounts are added. |
| [`SAMPLE_BRANCH`](#sample_branch) | constant | Node id of Sam's offline branch. |

## Structs

### Storefront

```rust
pub struct Storefront {
    pub id: &'static str,
    pub label: &'static str,
}
```

A storefront the demo image ships, by its plugin id (`plugin.toml` `id`).

**Fields**

- `id`: Plugin id the host enrolls the storefront under (its alias).
- `label`: Name shown on Sam's account for it.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### Lookup

```rust
pub struct Lookup {
    pub key: &'static str,
    pub store: &'static str,
    pub title: &'static str,
    pub author: &'static str,
}
```

One title to look up in a storefront's public catalog.

**Fields**

- `key`: Key the record is stored under in [`StoreRecords`](#storerecords).
- `store`: Plugin id ([`Storefront::id`](#storefront)) of the storefront to search.
- `title`: Title to search for and match.
- `author`: Author the matching record must list.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SeedReport

```rust
pub struct SeedReport {
    pub user_id: i64,
    pub books: usize,
    pub branch_titles: u64,
    pub wishlist: usize,
}
```

What [`seed`](#seed) wrote, for the log and for tests.

**Fields**

- `user_id`: Library user id of Sam.
- `books`: Books added from Sam's store accounts.
- `branch_titles`: Titles Sam's branch shared.
- `wishlist`: Wishlist items added for Sam.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Enums

### Seeded

```rust
pub enum Seeded {
    Wrote(SeedReport),
    AlreadySeeded,
}
```

Outcome of [`seed`](#seed).

**Variants**

- `Wrote`: The sample data was written.
- `AlreadySeeded`: The library already has the sample user; nothing was written.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### committed_records

```rust
pub fn committed_records() -> Result<StoreRecords>
```

The committed storefront records (`records.json`) for the current
sample titles; entries for titles no longer sampled are dropped.

#### Errors

Returns an error when `records.json` is not a JSON object of catalog
records.

### parse_records

```rust
pub fn parse_records(json: &str) -> Result<StoreRecords>
```

Parses a records file: a JSON object from [`Lookup::key`](#lookup) to a catalog
record, keeping only keys of current sample titles.

#### Errors

Returns an error when `json` is not such an object.

### merge_records

```rust
pub fn merge_records(live: StoreRecords, committed: StoreRecords) -> StoreRecords
```

Live lookups over the committed records: a title the live lookup found
keeps that record, and one it missed falls back to `committed` unless a
live record already holds that product id, so a stale committed record
never pushes out a fresh one. No two titles share a product id.

### records_json

```rust
pub fn records_json(records: &StoreRecords) -> Result<String>
```

`records` as the pretty-printed JSON `records.json` holds, one key per
title in sorted order, so a refresh diffs cleanly.

#### Errors

Returns an error when a record cannot be serialized.

### lookups

```rust
pub fn lookups() -> Vec<Lookup>
```

Every title the seed writes, as catalog lookups: the library, the
branch and the wishlist.

### search_query

```rust
pub fn search_query(lookup: &Lookup) -> String
```

The catalog search text for `lookup`: the title up to any `(…)` part,
then the author's surname.

GraphicAudio's catalog search finds a title only from its plain words:
punctuation or an author name makes it fall back to an alphabetical
listing of the whole catalog. Its query is the title's words alone.

### best_match

```rust
pub fn best_match<'a>(
    lookup: &Lookup,
    hits: &'a [CatalogHit],
    now: DateTime<Utc>,
) -> Option<&'a CatalogHit>
```

The search hit that is the wanted title by the wanted author, or `None`.

A hit that lists authors must list the wanted one (same surname and
first initial). A GraphicAudio hit without authors is judged on its
title alone, because that store never returns them; any other store's
authorless hit is refused. Among
matching hits the closest title wins, then one already released (a
pre-order of a new recording is not the edition a listener owns, by
`now`), then the store's own ranking.

### distinct_records

```rust
pub fn distinct_records(records: StoreRecords) -> StoreRecords
```

`records` without entries whose product id another title already took,
so two sample titles never become one book (the first, in
[`lookups`](#lookups) order, keeps it).

### seed

```rust
pub async fn seed(
    store: &LibraryStore,
    cluster_id: &str,
    plugin_keys: &BTreeMap<&str, String>,
    records: &StoreRecords,
    when: DateTime<Utc>,
) -> Result<Seeded>
```

Adds the sample library user, their books, wishlist and offline branch.

Runs once on a fresh demo library, before bookclerkd starts. A library
that already has [`SAMPLE_LOGIN`](#sample_login) is left alone, so running it twice
changes nothing.

#### Arguments

* `store` - The open library.
* `cluster_id` - The library's cluster id, recorded on the branch's node row.
* `plugin_keys` - PluginKey of each enrolled storefront, by plugin id
  ([`Storefront::id`](#storefront)). A storefront missing from it gets no account and
  its titles are left out, so an image without one still gets the rest.
* `records` - Storefront records for the titles, by [`Lookup::key`](#lookup)
  (pass them through [`distinct_records`](#distinct_records)). A title without one keeps
  the fields listed in this crate.
* `when` - The time the library was seeded; purchase dates and the
  branch's last heartbeat count back from it.

#### Returns

What was written, or [`Seeded::AlreadySeeded`](#seeded).

#### Errors

Returns an error when a write fails or the branch catalog refuses the
branch, which means the library code changed what makes a branch trusted.

### with_details

```rust
pub async fn with_details<F, Fut>(
    details: F,
    hit: CatalogHit,
    deadline: Instant,
) -> CatalogHit
where
    F: FnOnce(String) -> Fut,
    Fut: Future<Output = Result<Option<CatalogHit>>>,
```

Replaces a search hit with its storefront's full catalog record (the
details call Discover makes for title dialogs), so a store whose search
rows are lean, such as Libro.fm, still seeds a description, publisher,
genres and price.

#### Arguments

* `details` - The storefront's catalog-detail lookup for a product id
  (the source's `catalog_detail`); only this call is made.
* `hit` - The search row [`best_match`](#best_match) chose.
* `deadline` - When the lookup budget runs out.

#### Returns

The full record under `hit`'s product id, or `hit` unchanged when the
details call fails, times out, finds nothing, or returns a different
product than the search matched (neither the same product id nor the
same ISBN).

### records_path

```rust
pub fn records_path(args: impl Iterator<Item = String>) -> Result<Option<PathBuf>>
```

The `--write-records <path>` argument the seed binary was given, if any.

#### Arguments

* `args` - The command-line arguments after the program name.

#### Returns

Where to write the store records, or `None` to seed the library.

#### Errors

Returns an error for any other argument, or a missing path.

## Type aliases

### StoreRecords

```rust
pub type StoreRecords = BTreeMap<&'static str, CatalogHit>;
```

Storefront records found for the sample titles, by [`Lookup::key`](#lookup).

## Constants

### SAMPLE_LOGIN

```rust
pub const SAMPLE_LOGIN: &str = "sam";
```

Login name of the sample library user; its presence marks a seeded library.

### STOREFRONTS

```rust
pub const STOREFRONTS: [Storefront; 4] = _;
```

Every storefront Sam has an account at, in the order the accounts are added.

### SAMPLE_BRANCH

```rust
pub const SAMPLE_BRANCH: &str = "sams-nas";
```

Node id of Sam's offline branch.
