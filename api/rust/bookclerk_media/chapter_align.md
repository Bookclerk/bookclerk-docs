**bookclerk_media > chapter_align**

# Module: chapter_align

## Contents

**Structs**

- [`ChapterAlignOptions`](#chapteralignoptions) - Tunables for silence / speech-onset chapter snapping.

**Functions**

- [`align_chapter_starts`](#align_chapter_starts) - Snap chapter starts just before nearby speech onsets (spoken chapter titles).
- [`scale_chapters_to_duration`](#scale_chapters_to_duration) - Scale rebased chapter starts so they span the probed plain-audio duration.

---

## bookclerk_media::chapter_align::ChapterAlignOptions

*Struct*

Tunables for silence / speech-onset chapter snapping.

**Fields:**
- `window_ms: u64` - Search window on each side of the expected chapter start.
- `min_silence_ms: u64` - Minimum contiguous quiet frames before a speech onset counts.
- `frame_ms: u64` - RMS frame size used for energy analysis.
- `silence_rms_floor: f32` - Absolute speech-band RMS floor (s16) treated as quiet.
- `silence_rms_ratio: f32` - Fraction of window median speech-band RMS used as the quiet threshold.
- `max_lead_in_ms: u64` - Maximum ms to place the marker before the spoken-title onset. The
- `vocal_low_hz: f32` - Bandpass low cutoff for vocal detection (Hz).
- `vocal_high_hz: f32` - Bandpass high cutoff for vocal detection (Hz).

**Traits:** Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ChapterAlignOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_media::chapter_align::align_chapter_starts

*Function*

Snap chapter starts just before nearby speech onsets (spoken chapter titles).

Detects the speech-band onset of the spoken title, then walks backward
through quiet frames up to `max_lead_in_ms` so the marker precedes the title
without landing on a prior vocal waveform. Chapters at `0` are left
unchanged. Failures keep the original timestamp. Results stay monotonic.

Opens the media once and reuses a seek index across chapter windows so a
long book stays cheap (~tens of seconds of decode total, not a full pass).

```rust
fn align_chapter_starts(path: &std::path::Path, chapters: &[(String, u64)], opts: ChapterAlignOptions) -> Vec<(String, u64)>
```



## bookclerk_media::chapter_align::scale_chapters_to_duration

*Function*

Scale rebased chapter starts so they span the probed plain-audio duration.

When Audible content duration (`runtime - intro - outro`) differs from the
plain file, apply a uniform scale. No-ops when either duration is missing or
the relative delta is tiny.

```rust
fn scale_chapters_to_duration(chapters: &[(String, u64)], content_duration_ms: Option<u64>, plain_duration_ms: Option<u64>) -> Vec<(String, u64)>
```



