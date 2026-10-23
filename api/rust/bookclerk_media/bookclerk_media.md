**bookclerk_media**

# Module: bookclerk_media

## Contents

**Structs**

- [`MediaOutcome`](#mediaoutcome) - Outcome of a successful media operation.

**Functions**

- [`align_chapter_starts_async`](#align_chapter_starts_async) - Snap chapter starts to spoken-title onsets by local waveform analysis.
- [`encode_to_mp3`](#encode_to_mp3) - Re-encode audio to MP3 via Symphonia + LAME (classic Libation `DecryptToLossy`).
- [`remux_trimmed_async`](#remux_trimmed_async) - Copy/trim a progressive M4B/M4A into a new file (chapter split helper).

---

## bookclerk_media::MediaOutcome

*Struct*

Outcome of a successful media operation.

**Fields:**
- `output: std::path::PathBuf` - Absolute path of the written media file.

**Trait Implementations:**

- **From**
  - `fn from(value: MediaJobOutput) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> MediaOutcome`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_media::align_chapter_starts_async

*Function*

Snap chapter starts to spoken-title onsets by local waveform analysis.

The synchronous [`align_chapter_starts`](chapter_align.md#bookclerk_mediachapter_alignalign_chapter_starts) decodes audio on the calling
thread, which on an async runtime blocks a worker for the length of the
analysis. Prefer this from async code.

Analysis never fails the acquire: a decode problem leaves the original
chapter starts in place, matching the synchronous behaviour.

```rust
fn align_chapter_starts_async(path: &std::path::Path, chapters: &[(String, u64)], options: ChapterAlignOptions) -> Vec<(String, u64)>
```



## bookclerk_media::encode_to_mp3

*Function*

Re-encode audio to MP3 via Symphonia + LAME (classic Libation `DecryptToLossy`).

Runs in a confined media worker; see the module documentation.

# Errors

Returns `MediaError::InputMissing` when `input` does not exist, and
propagates encode and worker failures otherwise.

```rust
fn encode_to_mp3(input: &std::path::Path, output: &std::path::Path, lame: &bookclerk_config::LameConfig, max_sample_rate: Option<u32>, max_output_bytes: Option<u64>) -> Result<MediaOutcome>
```



## bookclerk_media::remux_trimmed_async

*Function*

Copy/trim a progressive M4B/M4A into a new file (chapter split helper).

Runs in a confined media worker; see the module documentation.

# Errors

Returns `MediaError::InputMissing` when `input` does not exist, and
propagates remux and worker failures otherwise.

```rust
fn remux_trimmed_async(input: &std::path::Path, output: &std::path::Path, trim: TrimRange) -> Result<MediaOutcome>
```



