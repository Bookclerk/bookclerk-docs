**bookclerk_library > wishlist_merge**

# Module: wishlist_merge

## Contents

**Functions**

- [`apply_merged_sources`](#apply_merged_sources) - Apply merged bib / editions / hints onto a wishlist row from its source snapshots.
- [`pick_better_description`](#pick_better_description) - Prefer HTML blurbs over plain text; otherwise the longer string wins.

---

## bookclerk_library::wishlist_merge::apply_merged_sources

*Function*

Apply merged bib / editions / hints onto a wishlist row from its source snapshots.

Updates `row` in place from `row.sources`, clearing merged fields when no
sources remain.

# Arguments

* `row` - Wishlist title-request record whose `sources` snapshots are merged.

```rust
fn apply_merged_sources(row: & mut crate::models::TitleRequestRecord)
```



## bookclerk_library::wishlist_merge::pick_better_description

*Function*

Prefer HTML blurbs over plain text; otherwise the longer string wins.

# Arguments

* `a` - First candidate description (may be HTML or plain).
* `b` - Second candidate description.

# Returns

The richer non-empty description, or `None` when both are empty.

```rust
fn pick_better_description(a: Option<&str>, b: Option<&str>) -> Option<String>
```



