**bookclerk > commands > discover**

# Module: commands::discover

## Contents

**Enums**

- [`DiscoverCommand`](#discovercommand) - `bookclerk discover` verbs: recommend, enrich, embed, listening sync, wishlist.
- [`WishlistCommand`](#wishlistcommand) - Personal wishlist helpers that also feed the shared global queue.

**Functions**

- [`run`](#run) - Dispatches a discover verb against the library store and source registry.

---

## bookclerk::commands::discover::DiscoverCommand

*Enum*

`bookclerk discover` verbs: recommend, enrich, embed, listening sync, wishlist.

**Variants:**
- `Recommend{ limit: Option<usize>, no_purchase_hints: bool, user: Option<String>, no_listening: bool, listening_providers: Vec<String> }` - Rebuild works graph, optionally enrich + embed, then print recommendations.
- `RebuildWorks` - Rebuild the works graph from ownership rows.
- `EnrichOpenlibrary` - Run Open Library enrichment for metadata gaps.
- `Embed{ hash: bool }` - Embed dirty works.
- `SyncListening` - Sync listening progress from all capable integrations into the library DB.
- `Wishlist{ command: WishlistCommand }` - Personal wishlist helpers (also feed the shared global queue).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`
- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`



## bookclerk::commands::discover::WishlistCommand

*Enum*

Personal wishlist helpers that also feed the shared global queue.

**Variants:**
- `Add{ title: String, authors: Option<String>, asin: Option<String>, isbn: Option<String>, notes: Option<String> }` - Add an open wishlist item.
- `List` - List open wishlist items (operator-owned when no portal identity).
- `Remove{ uuid: String }` - Un-wishlist by uuid (sets status to cancelled).

**Trait Implementations:**

- **FromArgMatches**
  - `fn from_arg_matches(__clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn from_arg_matches_mut(__clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<Self, clap::Error>`
  - `fn update_from_arg_matches(self: & mut Self, __clap_arg_matches: &clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
  - `fn update_from_arg_matches_mut<'b>(self: & mut Self, __clap_arg_matches: & mut clap::ArgMatches) -> ::std::result::Result<(), clap::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Subcommand**
  - `fn augment_subcommands<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn augment_subcommands_for_update<'b>(__clap_app: clap::Command) -> clap::Command`
  - `fn has_subcommand(__clap_name: &str) -> bool`



## bookclerk::commands::discover::run

*Function*

Dispatches a discover verb against the library store and source registry.

```rust
fn run(cfg: &bookclerk_config::Config, format: crate::format_out::OutputFormat, command: DiscoverCommand) -> anyhow::Result<()>
```



