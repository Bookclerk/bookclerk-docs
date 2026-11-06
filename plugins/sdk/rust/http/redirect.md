# bookclerk_plugin_sdk::http::redirect

Redirect follow policy (reqwest-compatible names).

## Enums

### Policy

```rust
pub enum Policy {
    None,
    Limited(usize),
    StopOnQueryContains(&'static str),
}
```

How many hops to follow after the initial request.

**Variants**

- `None`: Do not follow `Location`.
- `Limited`: Follow at most `n` hops.
- `StopOnQueryContains`: Follow at most 10 hops, but stop when the current URL's query
  contains `needle` (OAuth `authorization_code` capture — do not
  follow the post-maplanding hop that would drop the code).

#### Policy::none

```rust
pub fn none() -> Self
```

[`Policy::None`](#policy).

#### Policy::limited

```rust
pub fn limited(max: usize) -> Self
```

[`Policy::Limited`](#policy).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`
