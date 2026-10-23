**bookclerk > progress**

# Module: progress

## Contents

**Structs**

- [`BatchProgress`](#batchprogress) - Simple batch progress: `[####------] 2/5 title`.

**Functions**

- [`is_interactive`](#is_interactive) - True when interactive progress output is appropriate (stderr TTY, not redirected).

---

## bookclerk::progress::BatchProgress

*Struct*

Simple batch progress: `[####------] 2/5 title`.

**Fields:**
- `total: usize` - Expected item count (clamped to at least 1 so the bar fraction is defined).
- `current: usize` - Items completed so far, used for the bar fill and ETA.
- `label: String` - Verb shown after the counts (`scan`, `acquire`, …).
- `started: std::time::Instant` - When the bar started; used to estimate remaining minutes.
- `enabled: bool` - False when stderr is not a TTY (progress is a no-op).

**Methods:**

- `fn new<impl Into<String>>(total: usize, label: impl Trait) -> Self` - Starts a bar for `total` items; disabled automatically when stderr is redirected.
- `fn set(self: & mut Self, current: usize, detail: &str)` - Rewrites the current line with bar, counts, detail, and remaining minutes.
- `fn finish(self: & mut Self)` - Ends the current progress line so later stderr output starts on a new line.



## bookclerk::progress::is_interactive

*Function*

True when interactive progress output is appropriate (stderr TTY, not redirected).

```rust
fn is_interactive() -> bool
```



