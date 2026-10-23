**bookclerk_media > native**

# Module: native

## Contents

**Functions**

- [`remux_trimmed`](#remux_trimmed) - Remux a progressive clear M4B/M4A with an optional media-time trim (chapter split).

---

## bookclerk_media::native::remux_trimmed

*Function*

Remux a progressive clear M4B/M4A with an optional media-time trim (chapter split).

# Errors

Returns an error when the operation fails.

```rust
fn remux_trimmed(input: &std::path::Path, output: &std::path::Path, trim: bookclerk_mp4::TrimRange) -> crate::error::Result<crate::MediaOutcome>
```



