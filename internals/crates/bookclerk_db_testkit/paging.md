# bookclerk_db_testkit::paging

Checks that a paged listing returns what the full listing returns.

Under `crate::Profile::Minimum` a result holds one row, so every
listing method pages. These helpers walk a cursor-paged method to the end
and compare the walk with the unpaged result.

| Item | Kind | Summary |
| --- | --- | --- |
| [`collect_pages`](#collect_pages) | function | Walks `next_page` from the first page until it reports no further cursor, returning every item in page order. |
| [`assert_pages_match_full`](#assert_pages_match_full) | function | Asserts that walking `next_page` at `page_size` yields exactly `full`. |
| [`assert_pages_match_full_within`](#assert_pages_match_full_within) | function | `assert_pages_match_full` with the walk bounded at `max_pages`. |

## Functions

### collect_pages

```rust
pub async fn collect_pages<T, C, F, Fut>(
    page_size: u64,
    max_pages: usize,
    next_page: F,
) -> Vec<T>
where
    F: FnMut(Option<C>, u64) -> Fut,
    Fut: Future<Output = (Vec<T>, Option<C>)>,
```

Walks `next_page` from the first page until it reports no further
cursor, returning every item in page order.

`next_page` takes the cursor to start after (`None` for the first page)
and the page size, and returns the page's items and the cursor for the
page after it (`None` when this was the last page).

A page may be empty and still carry a cursor: a listing that skips rows
it cannot read (unreadable job payloads, say) answers that way for a
page of skipped rows, and the walk goes on to the next page.

#### Panics

Panics when the walk exceeds `max_pages`, so a cursor that never
advances fails the test instead of hanging it. Size `max_pages` from the
rows the listing reads, skipped ones included.

### assert_pages_match_full

```rust
pub async fn assert_pages_match_full<T, C, F, Fut>(
    full: &[T],
    page_size: u64,
    next_page: F,
)
where
    T: PartialEq + Debug,
    F: FnMut(Option<C>, u64) -> Fut,
    Fut: Future<Output = (Vec<T>, Option<C>)>,
```

Asserts that walking `next_page` at `page_size` yields exactly `full`.

The walk is bounded at one page per item of `full` plus two, which
covers a listing that caps its pages below the requested size (one row
per page under a one-row result cap). A listing that skips rows it
cannot read may need more pages than it returns items; walk it with
[`assert_pages_match_full_within`](#assert_pages_match_full_within) and a bound sized from the rows it
reads.

#### Panics

Panics when the paged walk differs from `full`, or on the conditions of
[`collect_pages`](#collect_pages).

### assert_pages_match_full_within

```rust
pub async fn assert_pages_match_full_within<T, C, F, Fut>(
    full: &[T],
    page_size: u64,
    max_pages: usize,
    next_page: F,
)
where
    T: PartialEq + Debug,
    F: FnMut(Option<C>, u64) -> Fut,
    Fut: Future<Output = (Vec<T>, Option<C>)>,
```

[`assert_pages_match_full`](#assert_pages_match_full) with the walk bounded at `max_pages`.

#### Panics

Panics when the paged walk differs from `full`, or on the conditions of
[`collect_pages`](#collect_pages).
