**bookclerk_media > brand**

# Module: brand

## Contents

**Structs**

- [`BrandDurations`](#branddurations) - Brand intro/outro durations from Audible `chapter_info`.

**Functions**

- [`brand_durations_from_chapter_info`](#brand_durations_from_chapter_info) - Read brand intro/outro durations from chapter_info JSON.
- [`brand_trim_range`](#brand_trim_range) - Build a media trim window that drops Audible branding audio.
- [`rebase_chapters_after_brand_trim`](#rebase_chapters_after_brand_trim) - Adjust flat chapter start offsets after brand trim so cues/metadata stay aligned.
- [`runtime_length_ms_from_chapter_info`](#runtime_length_ms_from_chapter_info) - Read `runtime_length_ms` / `runtimeLengthMs` from chapter_info JSON.

---

## bookclerk_media::brand::BrandDurations

*Struct*

Brand intro/outro durations from Audible `chapter_info`.

**Fields:**
- `intro_ms: u64` - Brand intro (pre-roll) duration in milliseconds.
- `outro_ms: u64` - Brand outro (post-roll) duration in milliseconds.

**Methods:**

- `fn is_empty(self: Self) -> bool` - True when both intro and outro are zero (nothing to trim).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> BrandDurations`
- **Clone**
  - `fn clone(self: &Self) -> BrandDurations`
- **PartialEq**
  - `fn eq(self: &Self, other: &BrandDurations) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_media::brand::brand_durations_from_chapter_info

*Function*

Read brand intro/outro durations from chapter_info JSON.

Accepts both snake_case (`brand_intro_duration_ms`) and camelCase
(`brandIntroDurationMs`) — Audible's live API uses camelCase.

```rust
fn brand_durations_from_chapter_info(info: &serde_json::Value) -> BrandDurations
```



## bookclerk_media::brand::brand_trim_range

*Function*

Build a media trim window that drops Audible branding audio.

Classic Libation: shift the first chapter start by `intro_ms` and shorten the
last chapter by `outro_ms`, then remux only the remaining media. We express
the same idea as an absolute `[start_ms, end_ms)` window on the full file.

When `outro_ms > 0` but `runtime_length_ms` is missing, returns `None` so we
do not silently leave branding on the tail while trimming only the intro.
Callers should supply runtime (from chapter_info or a media probe) first.

```rust
fn brand_trim_range(brand: BrandDurations, runtime_length_ms: Option<u64>) -> Option<bookclerk_mp4::TrimRange>
```



## bookclerk_media::brand::rebase_chapters_after_brand_trim

*Function*

Adjust flat chapter start offsets after brand trim so cues/metadata stay aligned.

Drops chapters that fall entirely inside the stripped intro, rebases survivors
so the first kept chapter starts at 0, and shortens the implied end by `outro_ms`.

```rust
fn rebase_chapters_after_brand_trim(chapters: &[(String, u64)], brand: BrandDurations, runtime_length_ms: Option<u64>) -> Vec<(String, u64)>
```



## bookclerk_media::brand::runtime_length_ms_from_chapter_info

*Function*

Read `runtime_length_ms` / `runtimeLengthMs` from chapter_info JSON.

```rust
fn runtime_length_ms_from_chapter_info(info: &serde_json::Value) -> Option<u64>
```



