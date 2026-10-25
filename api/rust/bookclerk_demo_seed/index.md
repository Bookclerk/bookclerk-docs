# bookclerk_demo_seed

Sample data for the hosted demo's library.

A demo visitor lands in a library of their own, as its Owner. An empty
library shows little of what Bookclerk does with more than one person, so
before bookclerkd starts, the demo image runs this crate's binary once to
add a second library user, Sam, who:

- has a Libro.fm account linked, with public-domain titles already in the
  library;
- keeps a wishlist;
- owns a branch, "sams-nas", that shared part of its catalog and then
  went offline (its last heartbeat is days old).

Everything is written through the library store and the branch catalog
code the daemon uses, never raw SQL, so the seed follows the schema of
the commit it is built from. Nothing here signs in: Sam has no password
and no claim ticket, and the branch's credential keys are placeholders
that never verify a signature.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SeedReport`](#seedreport) | struct | What `seed` wrote, for the log and for tests. |
| [`Seeded`](#seeded) | enum | Outcome of `seed`. |
| [`seed`](#seed) | function | Adds the sample library user, their books, wishlist and offline branch. |
| [`SAMPLE_LOGIN`](#sample_login) | constant | Login name of the sample library user; its presence marks a seeded library. |
| [`SAMPLE_STORE`](#sample_store) | constant | Plugin id (`plugin.toml` `id`) of the storefront Sam's books come from. |
| [`SAMPLE_BRANCH`](#sample_branch) | constant | Node id of Sam's offline branch. |

## Structs

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
- `books`: Books added from Sam's Libro.fm account.
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

### seed

```rust
pub async fn seed(
    store: &LibraryStore,
    cluster_id: &str,
    store_plugin_key: &str,
) -> Result<Seeded>
```

Adds the sample library user, their books, wishlist and offline branch.

Runs once on a fresh demo library, before bookclerkd starts. A library
that already has [`SAMPLE_LOGIN`](#sample_login) is left alone, so running it twice
changes nothing.

#### Arguments

* `store` - The open library.
* `cluster_id` - The library's cluster id, recorded on the branch's node row.
* `store_plugin_key` - PluginKey of the enrolled [`SAMPLE_STORE`](#sample_store) plugin,
  which owns Sam's account and books.

#### Returns

What was written, or [`Seeded::AlreadySeeded`](#seeded).

#### Errors

Returns an error when a write fails or the branch catalog refuses the
branch, which means the library code changed what makes a branch trusted.

## Constants

### SAMPLE_LOGIN

```rust
pub const SAMPLE_LOGIN: &str = "sam";
```

Login name of the sample library user; its presence marks a seeded library.

### SAMPLE_STORE

```rust
pub const SAMPLE_STORE: &str = "libro";
```

Plugin id (`plugin.toml` `id`) of the storefront Sam's books come from.

### SAMPLE_BRANCH

```rust
pub const SAMPLE_BRANCH: &str = "sams-nas";
```

Node id of Sam's offline branch.
