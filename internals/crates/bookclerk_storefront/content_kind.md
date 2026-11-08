# bookclerk_storefront::content_kind

Content-kind labels storefronts report for podcast shows and episodes.

The host's library model re-exports these so a storefront guest and the
host classify a `content_kind` string the same way.

| Item | Kind | Summary |
| --- | --- | --- |
| [`is_episode`](#is_episode) | function | True when `content_kind` is a podcast episode. |
| [`is_podcast_parent`](#is_podcast_parent) | function | True when `content_kind` is a podcast parent / show (no audio to acquire). |

## Functions

### is_episode

```rust
pub fn is_episode(content_kind: &str) -> bool
```

True when `content_kind` is a podcast episode.

### is_podcast_parent

```rust
pub fn is_podcast_parent(content_kind: &str) -> bool
```

True when `content_kind` is a podcast parent / show (no audio to acquire).
