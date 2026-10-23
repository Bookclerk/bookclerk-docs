**bookclerk_media > job**

# Module: job

## Contents

**Enums**

- [`MediaJob`](#mediajob) - A single codec operation.
- [`MediaJobOutput`](#mediajoboutput) - What a finished job produced.
- [`MediaJobReply`](#mediajobreply) - Wire form of a finished job. `Result` is not `Serialize`, so the worker and

---

## bookclerk_media::job::MediaJob

*Enum*

A single codec operation.

These are the CPU-bound paths that decode or encode attacker-influenced
audio through LAME and FDK-AAC, both of which are C libraries.

**Variants:**
- `EncodeMp3{ input: std::path::PathBuf, output: std::path::PathBuf, lame: Box<bookclerk_config::LameConfig>, max_sample_rate: Option<u32>, max_output_bytes: Option<u64> }` - Re-encode audio to MP3 via Symphonia and LAME.
- `RemuxTrimmed{ input: std::path::PathBuf, output: std::path::PathBuf, trim: bookclerk_mp4::TrimRange }` - Copy or trim a progressive M4B/M4A into a new file.
- `Fixup{ request: Box<crate::metadata::FixupRequest> }` - Write metadata tags, cover art, and chapters.
- `PackageM4b{ request: Box<crate::package_m4b::PackageM4bRequest> }` - Package ordered audio parts into one M4B.
- `AlignChapters{ path: std::path::PathBuf, chapters: Vec<(String, u64)>, options: crate::chapter_align::ChapterAlignOptions }` - Snap chapter starts to spoken-title onsets by local waveform analysis.

**Methods:**

- `fn read_paths(self: &Self) -> Vec<PathBuf>` - Paths the job reads but must not modify.
- `fn write_dirs(self: &Self) -> Vec<PathBuf>` - Directories the job writes into.
- `fn label(self: &Self) -> &'static str` - Short label for logs and error messages.
- `fn prepare_output_dirs(self: &Self) -> std::io::Result<()>` - Create the output directories this job needs.
- `fn run(self: Self) -> Result<MediaJobOutput>` - Execute the job on the calling thread.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MediaJob`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_media::job::MediaJobOutput

*Enum*

What a finished job produced.

**Variants:**
- `File{ output: std::path::PathBuf }` - A single written file.
- `FileWithChapters{ output: std::path::PathBuf, chapters: Vec<(String, u64)> }` - A written file plus the chapter list that was muxed into it.
- `Chapters{ chapters: Vec<(String, u64)> }` - Adjusted chapter starts; writes nothing.

**Methods:**

- `fn output(self: &Self) -> Option<&Path>` - The written file, when this job produced one.
- `fn chapters(self: &Self) -> Option<&[(String, u64)]>` - The chapter list, when this job produced one.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &MediaJobOutput) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> MediaJobOutput`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_media::job::MediaJobReply

*Enum*

Wire form of a finished job. `Result` is not `Serialize`, so the worker and
the pool exchange this instead.

**Variants:**
- `Ok(MediaJobOutput)` - The job succeeded.
- `Err{ message: String }` - The job failed. Carries the rendered error, since [`crate::MediaError`](error.md#bookclerk_mediaerrormediaerror)

**Trait Implementations:**

- **From**
  - `fn from(result: Result<MediaJobOutput>) -> Self`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> MediaJobReply`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



