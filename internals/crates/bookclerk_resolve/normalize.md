# bookclerk_resolve::normalize

Title and author cleaning for compares.

Port of `cleanTitleForCompares` and `cleanAuthorForCompares` from
audiobookshelf (`server/finders/BookFinder.js`), moved here from
`bookclerk-enrich`'s `match_score` so the resolver's work matcher and the
enrichment score clean text the same way.

Because it derives from audiobookshelf, this module is licensed under
GPL-3.0-only rather than the workspace's AGPL-3.0-or-later.

| Item | Kind | Summary |
| --- | --- | --- |
| [`has_subtitle`](#has_subtitle) | function | True when the title already embeds a subtitle after `: ` or ` - `. |
| [`clean_title`](#clean_title) | function | Cleans a title for compares (AudioBookshelf `cleanTitleForCompares`). |
| [`clean_author`](#clean_author) | function | Cleans an author name for compares (AudioBookshelf `cleanAuthorForCompares`). |

## Functions

### has_subtitle

```rust
pub fn has_subtitle(title: &str) -> bool
```

True when the title already embeds a subtitle after `: ` or ` - `.

#### Arguments

* `title` - Display title.

#### Returns

True when a subtitle separator is present.

### clean_title

```rust
pub fn clean_title(title: &str, keep_subtitle: bool) -> String
```

Cleans a title for compares (AudioBookshelf `cleanTitleForCompares`).

The caller decodes HTML entities first; this function treats `&amp;`
as literal text.

#### Arguments

* `title` - Display title.
* `keep_subtitle` - When true, retain subtitle text for soft compares.

#### Returns

String result for this operation.

### clean_author

```rust
pub fn clean_author(author: &str) -> String
```

Cleans an author name for compares (AudioBookshelf `cleanAuthorForCompares`).

The caller decodes HTML entities first.

#### Arguments

* `author` - String `author` for this call.

#### Returns

String result for this operation.
