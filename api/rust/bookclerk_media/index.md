# bookclerk_media

Clear-media packaging: remux, metadata fix-up, and MP3 encode.

## Audience

Host acquire / media-worker code. Store plugins may call helpers that are
re-exported here, but DRM (Adrm / CENC) stays in those plugins. The
ISO-BMFF reading and writing underneath is [`bookclerk_mp4`](../bookclerk_mp4/index.md), which both this
crate and those plugins share; the keys and ciphers stay with the plugin.

Style: `docs/contributing/code-documentation.md`. Product narrative: `docs/server/media.md`.

## Where the work runs

The async entry points in this module do not decode anything themselves.
They build a [`MediaJob`](#mediajob) and hand it to a [`MediaPool`](#mediapool), which runs it in a
short-lived child process confined to the paths that job declared. Hosts
install the pool once at startup with [`init_pool`](#init_pool); anything that skips
that gets a default pool, which still isolates as long as the worker binary
is installed alongside the host.

The synchronous functions ([`remux_trimmed`](#remux_trimmed), [`align_chapter_starts`](#align_chapter_starts),
[`package_m4b_from_pcm`](#package_m4b_from_pcm), and the MP4 parsing helpers) run in-process by
design. They are cheap, take already-parsed input, or are the implementation
the worker itself calls.

| Item | Kind | Summary |
| --- | --- | --- |
| [`BrandDurations`](#branddurations) | struct | Brand intro/outro durations from Audible `chapter_info`. |
| [`ChapterAlignOptions`](#chapteralignoptions) | struct | Tunables for silence / speech-onset chapter snapping. |
| [`FixupRequest`](#fixuprequest) | struct | Request to fix up audiobook metadata after decrypt / download. |
| [`PackageM4bRequest`](#packagem4brequest) | struct | Request to package ordered MP3 parts into one M4B. |
| [`MediaPool`](#mediapool) | struct | Bounded pool of confined media workers. |
| [`MediaPoolConfig`](#mediapoolconfig) | struct | Pool sizing and isolation settings. |
| [`MediaOutcome`](#mediaoutcome) | struct | Outcome of a successful media operation. |
| [`MediaError`](#mediaerror) | enum | Errors from in-process codecs, MP4 plumbing, and confined media workers. |
| [`MediaJob`](#mediajob) | enum | A single codec operation. |
| [`MediaJobOutput`](#mediajoboutput) | enum | What a finished job produced. |
| [`MediaJobReply`](#mediajobreply) | enum | Wire form of a finished job. `Result` is not `Serialize`, so the worker and the pool exchange this instead. |
| [`Confinement`](#confinement) | enum | How strictly a worker must be confined before it will process a job. |
| [`write_output_within_budget`](#write_output_within_budget) | function | Appends `data` only when the result stays within `max`. |
| [`brand_durations_from_chapter_info`](#brand_durations_from_chapter_info) | function | Read brand intro/outro durations from chapter_info JSON. |
| [`brand_trim_range`](#brand_trim_range) | function | Build a media trim window that drops Audible branding audio. |
| [`rebase_chapters_after_brand_trim`](#rebase_chapters_after_brand_trim) | function | Adjust flat chapter start offsets after brand trim so cues/metadata stay aligned. |
| [`runtime_length_ms_from_chapter_info`](#runtime_length_ms_from_chapter_info) | function | Read `runtime_length_ms` / `runtimeLengthMs` from chapter_info JSON. |
| [`align_chapter_starts`](#align_chapter_starts) | function | Snap chapter starts just before nearby speech onsets (spoken chapter titles). |
| [`scale_chapters_to_duration`](#scale_chapters_to_duration) | function | Scale rebased chapter starts so they span the probed plain-audio duration. |
| [`bookclerk_tool_tag`](#bookclerk_tool_tag) | function | `bookclerk <version>` string embedded in encoder / freeform TOOL tags. |
| [`fixup_audiobook`](#fixup_audiobook) | function | Apply metadata tags, optional cover embed, and optional chapters. |
| [`remux_trimmed`](#remux_trimmed) | function | Remux a progressive clear M4B/M4A with an optional media-time trim (chapter split). |
| [`package_m4b_from_mp3`](#package_m4b_from_mp3) | function | Decode/remux ordered parts → progressive M4B. |
| [`package_m4b_from_pcm`](#package_m4b_from_pcm) | function | Encode interleaved PCM to AAC-LC and mux an M4B (test / internal helper). |
| [`init_pool`](#init_pool) | function | Install the process-wide pool if none is installed yet. |
| [`init_pool_from_config`](#init_pool_from_config) | function | Build the pool from `[media]` config and install it, logging what it will do. |
| [`pool`](#pool) | function | The process-wide pool, creating a default one if startup never installed it. |
| [`replace_pool`](#replace_pool) | function | Swap in `pool`, returning the one it replaced. |
| [`encode_to_mp3`](#encode_to_mp3) | function | Re-encode audio to MP3 via Symphonia + LAME (classic Libation `DecryptToLossy`). |
| [`remux_trimmed_async`](#remux_trimmed_async) | function | Copy/trim a progressive M4B/M4A into a new file (chapter split helper). |
| [`align_chapter_starts_async`](#align_chapter_starts_async) | function | Snap chapter starts to spoken-title onsets by local waveform analysis. |
| [`Result`](#result) | type alias | Result alias for media packaging and worker operations in this crate. |
| [`BOOKCLERK_TOOL_NAME`](#bookclerk_tool_name) | constant | Product / CLI name written into tool attribution tags. |
| [`JAIL_BIN_ENV`](#jail_bin_env) | constant | Environment variable naming the jail launcher (shared with plugin hosts). |
| [`JAIL_BIN_NAME`](#jail_bin_name) | constant | File name of the jail launcher when it sits beside the host / worker. |
| [`WORKER_BIN_ENV`](#worker_bin_env) | constant | Environment variable naming the worker binary explicitly. |
| [`WORKER_BIN_NAME`](#worker_bin_name) | constant | File name of the worker binary when it sits beside the host executable. |
| [`WORKER_ENFORCEMENT_ENV`](#worker_enforcement_env) | constant | Environment variable the pool sets on each worker to select its failure mode. |

## Re-exports

- [`bookclerk_mp4::extract_mp4a_config`](../bookclerk_mp4/index.md#extract_mp4a_config)
- [`bookclerk_mp4::parse_mp4`](../bookclerk_mp4/index.md#parse_mp4)
- [`bookclerk_mp4::track_duration_ms`](../bookclerk_mp4/index.md#track_duration_ms)
- [`bookclerk_mp4::Mp4aConfig`](../bookclerk_mp4/index.md#mp4aconfig)
- [`bookclerk_mp4::SampleEntryKind`](../bookclerk_mp4/index.md#sampleentrykind)
- [`bookclerk_mp4::TrimRange`](../bookclerk_mp4/index.md#trimrange)

## Structs

### BrandDurations

```rust
pub struct BrandDurations {
    pub intro_ms: u64,
    pub outro_ms: u64,
}
```

Brand intro/outro durations from Audible `chapter_info`.

**Fields**

- `intro_ms`: Brand intro (pre-roll) duration in milliseconds.
- `outro_ms`: Brand outro (post-roll) duration in milliseconds.

#### BrandDurations::is_empty

```rust
pub fn is_empty(self) -> bool
```

True when both intro and outro are zero (nothing to trim).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### ChapterAlignOptions

```rust
pub struct ChapterAlignOptions {
    pub window_ms: u64,
    pub min_silence_ms: u64,
    pub frame_ms: u64,
    pub silence_rms_floor: f32,
    pub silence_rms_ratio: f32,
    pub max_lead_in_ms: u64,
    pub vocal_low_hz: f32,
    pub vocal_high_hz: f32,
}
```

Tunables for silence / speech-onset chapter snapping.

**Fields**

- `window_ms`: Search window on each side of the expected chapter start.
- `min_silence_ms`: Minimum contiguous quiet frames before a speech onset counts.
- `frame_ms`: RMS frame size used for energy analysis.
- `silence_rms_floor`: Absolute speech-band RMS floor (s16) treated as quiet.
- `silence_rms_ratio`: Fraction of window median speech-band RMS used as the quiet threshold.
- `max_lead_in_ms`: Maximum ms to place the marker before the spoken-title onset. The
  marker stops earlier when walking back would hit prior vocal energy.
- `vocal_low_hz`: Bandpass low cutoff for vocal detection (Hz).
- `vocal_high_hz`: Bandpass high cutoff for vocal detection (Hz).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Serialize`

### FixupRequest

```rust
pub struct FixupRequest {
    pub input: PathBuf,
    pub output: PathBuf,
    pub title: String,
    pub author: Option<String>,
    pub narrator: Option<String>,
    pub cover: Option<PathBuf>,
    pub chapters: Vec<(String, u64)>,
    pub replace_chapters: bool,
    pub subtitle: Option<String>,
    pub publisher: Option<String>,
    pub year: Option<String>,
    pub genre: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub description: Option<String>,
    pub language: Option<String>,
    pub tool: Option<String>,
}
```

Request to fix up audiobook metadata after decrypt / download.

**Fields**

- `input`: Absolute path of the clear media file to tag (copied/written to output).
- `output`: Absolute path where the tagged file should be written.
- `title`: Primary title embedded in the container tags.
- `author`: Author / artist tag when known.
- `narrator`: Narrator tag when known (ABS / iTunes freeform).
- `cover`: Optional cover image path to embed (JPEG/PNG).
- `chapters`: Chapter titles + start offsets in milliseconds (embedded for M4B).
- `replace_chapters`: When true, replace any existing M4B chapter list/track with [`Self::chapters`](#fixuprequest)
  (used when overlaying Audible chapter trees onto Libro packaged M4Bs).
  When false, preserve existing chapters if the file already has them.
- `subtitle`: Subtitle / album subtitle when known.
- `publisher`: Publisher imprint for the publisher tag.
- `year`: Publish year as a string (e.g. `"2011"`).
- `genre`: Genre string; multiple genres should use `;` separators for ABS.
- `series`: Series name written to Tone / ABS freeform atoms when set.
- `series_index`: Series sequence string (e.g. `"1"`) for ABS freeform atoms.
- `asin`: Amazon ASIN written to freeform metadata when set.
- `isbn`: ISBN written to freeform metadata when set.
- `description`: Long description / comment body when set.
- `language`: Content language tag when set.
- `tool`: Tool attribution (`bookclerk 0.1.0`). Defaults to [`bookclerk_tool_tag`](#bookclerk_tool_tag) when `None`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### PackageM4bRequest

```rust
pub struct PackageM4bRequest {
    pub parts: Vec<PathBuf>,
    pub output: PathBuf,
    pub chapter_titles: Vec<String>,
}
```

Request to package ordered MP3 parts into one M4B.

**Fields**

- `parts`: Ordered MP3 chapter / part files.
- `output`: Destination `.m4b` path.
- `chapter_titles`: Optional chapter titles (length may match `parts`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### MediaPool

```rust
pub struct MediaPool {
    /* private fields */
}
```

Bounded pool of confined media workers.

#### MediaPool::new

```rust
pub fn new(config: MediaPoolConfig) -> Self
```

Build a pool from `config`, resolving the worker binary once.

Resolution failures are recorded rather than returned, so a
misconfigured host still starts and reports the problem through
[`summary`](#mediapoolsummary) and its logs. The refusal happens per job.

#### MediaPool::in_process

```rust
pub fn in_process() -> Self
```

A pool that runs every job on a blocking thread in this process.

#### MediaPool::capacity

```rust
pub fn capacity(&self) -> usize
```

Maximum jobs this pool will run at once.

#### MediaPool::is_isolated

```rust
pub fn is_isolated(&self) -> bool
```

Whether jobs run in a separate, confined process.

#### MediaPool::is_refusing

```rust
pub fn is_refusing(&self) -> bool
```

Whether this pool will refuse every job because isolation is required
but unavailable.

#### MediaPool::worker_bin

```rust
pub fn worker_bin(&self) -> Option<&PathBuf>
```

Resolved worker binary, when one was found.

#### MediaPool::summary

```rust
pub fn summary(&self) -> String
```

One-line summary for startup logs and `bookclerk doctor`.

#### MediaPool::run

```rust
pub async fn run(&self, job: MediaJob) -> Result<MediaJobOutput>
```

Run `job`, waiting for a free slot first.

##### Errors

Returns [`MediaError::NotIsolated`](#mediaerror) when confinement is required but no
worker binary was found, [`MediaError::Worker`](#mediaerror) when the worker cannot
be started, exits without a reply, or returns something unparseable,
and propagates the job's own error otherwise.

Implements: `Debug`, `Default`, `Drop`

### MediaPoolConfig

```rust
pub struct MediaPoolConfig {
    pub workers: usize,
    pub confinement: Confinement,
    pub worker_bin: Option<PathBuf>,
}
```

Pool sizing and isolation settings.

**Fields**

- `workers`: Maximum jobs running at once. Zero means "derive from available
  parallelism".
- `confinement`: How strictly workers must be confined.
- `worker_bin`: Explicit worker binary path. When absent the pool looks at
  [`WORKER_BIN_ENV`](#worker_bin_env) and then beside the current executable.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `From<&MediaConfig>`, `PartialEq`

### MediaOutcome

```rust
pub struct MediaOutcome {
    pub output: PathBuf,
}
```

Outcome of a successful media operation.

**Fields**

- `output`: Absolute path of the written media file.

Implements: `Clone`, `Debug`, `From<MediaJobOutput>`

## Enums

### MediaError

```rust
pub enum MediaError {
    InputMissing(PathBuf),
    OutputMissing(PathBuf),
    Mp4(String),
    Native(String),
    Worker { job: &'static str, detail: String },
    NotIsolated { job: &'static str, detail: String },
    Io(Error),
    Other(Error),
}
```

Errors from in-process codecs, MP4 plumbing, and confined media workers.

**Variants**

- `InputMissing`: A declared input path does not exist or is unreachable.
- `OutputMissing`: The job reported success but the expected output path was not written.
- `Mp4`: ISO-BMFF / MP4 container parse or remux failure.
- `Native`: Native encode / remux / metadata failure (non-container detail).
- `Worker`: A media worker process could not be started, died, or returned a reply
  that could not be parsed. Distinct from [`MediaError::Native`](#mediaerror) so a
  crashed codec is not mistaken for a malformed file.
  - `job`: Which operation was running.
  - `detail`: What went wrong.
- `NotIsolated`: Isolation is required but unavailable, so the job was refused rather
  than run unconfined. Separate from [`MediaError::Worker`](#mediaerror) because
  nothing went wrong at runtime — the host is misconfigured.
  - `job`: Which operation was refused.
  - `detail`: Why isolation is unavailable.
- `Io`: Local filesystem I/O failure.
- `Other`: Catch-all for wrapped media failures.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<Mp4Error>`

### MediaJob

```rust
pub enum MediaJob {
    EncodeMp3 { input: PathBuf, output: PathBuf, lame: Box<LameConfig>, max_sample_rate: Option<u32>, max_output_bytes: Option<u64> },
    RemuxTrimmed { input: PathBuf, output: PathBuf, trim: TrimRange },
    Fixup { request: Box<FixupRequest> },
    PackageM4b { request: Box<PackageM4bRequest> },
    AlignChapters { path: PathBuf, chapters: Vec<(String, u64)>, options: ChapterAlignOptions },
}
```

A single codec operation.

These are the CPU-bound paths that decode or encode attacker-influenced
audio through LAME and FDK-AAC, both of which are C libraries.

**Variants**

- `EncodeMp3`: Re-encode audio to MP3 via Symphonia and LAME.
  - `input`: Absolute path of the source audio to re-encode.
  - `output`: Absolute path where the MP3 should be written.
  - `lame`: LAME encoder knobs for this encode.
  - `max_sample_rate`: Optional ceiling for output sample rate in Hz (`None` = leave source).
  - `max_output_bytes`: Hard cap on bytes written to `output`. `None` does not cap the file.
- `RemuxTrimmed`: Copy or trim a progressive M4B/M4A into a new file.
  - `input`: Absolute path of the source M4B/M4A.
  - `output`: Absolute path where the remuxed file should be written.
  - `trim`: Inclusive start / exclusive end window in milliseconds.
- `Fixup`: Write metadata tags, cover art, and chapters.
  - `request`: Fix-up request payload (boxed for enum size).
- `PackageM4b`: Package ordered audio parts into one M4B.
  - `request`: Packaging request payload (boxed for enum size).
- `AlignChapters`: Snap chapter starts to spoken-title onsets by local waveform analysis.
  - `path`: Absolute path of the audio file to analyse.
  - `chapters`: Chapter titles paired with start offsets in milliseconds.
  - `options`: Waveform-alignment tuning (window sizes, thresholds).

#### MediaJob::read_paths

```rust
pub fn read_paths(&self) -> Vec<PathBuf>
```

Paths the job reads but must not modify.

These become the worker's read allowlist verbatim. Symbolic links are
granted at their resolved target, because that is the inode the kernel
checks — declaring a link inside the cache directory therefore grants
whatever it points at. The host builds these paths from its own cache
and output roots, so nothing untrusted chooses them.

#### MediaJob::write_dirs

```rust
pub fn write_dirs(&self) -> Vec<PathBuf>
```

Directories the job writes into.

Directories rather than files, because the codecs stage output through
temporary files alongside the destination and Landlock rules apply to a
path and everything beneath it.

#### MediaJob::label

```rust
pub fn label(&self) -> &'static str
```

Short label for logs and error messages.

#### MediaJob::prepare_output_dirs

```rust
pub fn prepare_output_dirs(&self) -> Result<()>
```

Create the output directories this job needs.

Both Landlock and Seatbelt reject a rule naming a path that does not
exist, so the destination has to be created before the jail is built.

##### Errors

Propagates directory-creation failures.

#### MediaJob::run

```rust
pub fn run(self) -> Result<MediaJobOutput>
```

Execute the job on the calling thread.

This is the entry point the worker binary calls after confining itself.
It is also used directly when process isolation is turned off.

##### Errors

Returns [`crate::MediaError::InputMissing`](#mediaerror) when a declared input cannot
be reached, and propagates codec and I/O failures otherwise.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### MediaJobOutput

```rust
pub enum MediaJobOutput {
    File { output: PathBuf },
    FileWithChapters { output: PathBuf, chapters: Vec<(String, u64)> },
    Chapters { chapters: Vec<(String, u64)> },
}
```

What a finished job produced.

**Variants**

- `File`: A single written file.
  - `output`: Path to the written media file.
- `FileWithChapters`: A written file plus the chapter list that was muxed into it.
  - `output`: Path to the written media file.
  - `chapters`: Chapter titles paired with start offsets in milliseconds.
- `Chapters`: Adjusted chapter starts; writes nothing.
  - `chapters`: Chapter titles paired with start offsets in milliseconds.

#### MediaJobOutput::output

```rust
pub fn output(&self) -> Option<&Path>
```

The written file, when this job produced one.

#### MediaJobOutput::chapters

```rust
pub fn chapters(&self) -> Option<&[(String, u64)]>
```

The chapter list, when this job produced one.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `From<MediaJobOutput>`, `PartialEq`, `Serialize`

### MediaJobReply

```rust
pub enum MediaJobReply {
    Ok(MediaJobOutput),
    Err { message: String },
}
```

Wire form of a finished job. `Result` is not `Serialize`, so the worker and
the pool exchange this instead.

**Variants**

- `Ok`: The job succeeded.
- `Err`: The job failed. Carries the rendered error, since [`crate::MediaError`](#mediaerror)
  is not reconstructible across a process boundary.
  - `message`: Human-readable failure message from the worker.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `From<Result<MediaJobOutput, MediaError>>`, `Serialize`

### Confinement

```rust
pub enum Confinement {
    Required,
    BestEffort,
    Off,
}
```

How strictly a worker must be confined before it will process a job.

**Variants**

- `Required`: Refuse to run the job unless the worker confined itself. Production
  default: a host that cannot sandbox must not silently decode untrusted
  media next to the encryption key.
- `BestEffort`: Confine where the platform allows and log what did not engage.
- `Off`: Run jobs on a blocking thread in this process. Intended for development
  and for the crate's own algorithm tests.

#### Confinement::from_env_value

```rust
pub fn from_env_value(value: &str) -> Option<Self>
```

Parse the value carried in [`WORKER_ENFORCEMENT_ENV`](#worker_enforcement_env).

#### Confinement::as_env_value

```rust
pub fn as_env_value(self) -> &'static str
```

The value to pass through [`WORKER_ENFORCEMENT_ENV`](#worker_enforcement_env).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### write_output_within_budget

```rust
pub fn write_output_within_budget(
    file: &mut impl Write,
    data: &[u8],
    written: &mut u64,
    max: Option<u64>,
) -> Result<()>
```

Appends `data` only when the result stays within `max`.

The failing call writes nothing, so the file length never passes the budget.
`max == None` does not impose a byte ceiling.

#### Errors

Returns [`MediaError::Native`](#mediaerror) when `written + data.len()` would exceed
`max`. Returns [`MediaError::Io`](#mediaerror) when the accepted bytes cannot be written.

### brand_durations_from_chapter_info

```rust
pub fn brand_durations_from_chapter_info(info: &Value) -> BrandDurations
```

Read brand intro/outro durations from chapter_info JSON.

Accepts both snake_case (`brand_intro_duration_ms`) and camelCase
(`brandIntroDurationMs`) — Audible's live API uses camelCase.

### brand_trim_range

```rust
pub fn brand_trim_range(
    brand: BrandDurations,
    runtime_length_ms: Option<u64>,
) -> Option<TrimRange>
```

Build a media trim window that drops Audible branding audio.

Classic Libation: shift the first chapter start by `intro_ms` and shorten the
last chapter by `outro_ms`, then remux only the remaining media. We express
the same idea as an absolute `[start_ms, end_ms)` window on the full file.

When `outro_ms > 0` but `runtime_length_ms` is missing, returns `None` so we
do not silently leave branding on the tail while trimming only the intro.
Callers should supply runtime (from chapter_info or a media probe) first.

### rebase_chapters_after_brand_trim

```rust
pub fn rebase_chapters_after_brand_trim(
    chapters: &[(String, u64)],
    brand: BrandDurations,
    runtime_length_ms: Option<u64>,
) -> Vec<(String, u64)>
```

Adjust flat chapter start offsets after brand trim so cues/metadata stay aligned.

Drops chapters that fall entirely inside the stripped intro, rebases survivors
so the first kept chapter starts at 0, and shortens the implied end by `outro_ms`.

### runtime_length_ms_from_chapter_info

```rust
pub fn runtime_length_ms_from_chapter_info(info: &Value) -> Option<u64>
```

Read `runtime_length_ms` / `runtimeLengthMs` from chapter_info JSON.

### align_chapter_starts

```rust
pub fn align_chapter_starts(
    path: &Path,
    chapters: &[(String, u64)],
    opts: ChapterAlignOptions,
) -> Vec<(String, u64)>
```

Snap chapter starts just before nearby speech onsets (spoken chapter titles).

Detects the speech-band onset of the spoken title, then walks backward
through quiet frames up to `max_lead_in_ms` so the marker precedes the title
without landing on a prior vocal waveform. Chapters at `0` are left
unchanged. Failures keep the original timestamp. Results stay monotonic.

Opens the media once and reuses a seek index across chapter windows so a
long book stays cheap (~tens of seconds of decode total, not a full pass).

### scale_chapters_to_duration

```rust
pub fn scale_chapters_to_duration(
    chapters: &[(String, u64)],
    content_duration_ms: Option<u64>,
    plain_duration_ms: Option<u64>,
) -> Vec<(String, u64)>
```

Scale rebased chapter starts so they span the probed plain-audio duration.

When Audible content duration (`runtime - intro - outro`) differs from the
plain file, apply a uniform scale. No-ops when either duration is missing or
the relative delta is tiny.

### bookclerk_tool_tag

```rust
pub fn bookclerk_tool_tag() -> String
```

`bookclerk <version>` string embedded in encoder / freeform TOOL tags.

### fixup_audiobook

```rust
pub async fn fixup_audiobook(req: FixupRequest) -> Result<MediaOutcome>
```

Apply metadata tags, optional cover embed, and optional chapters.

Runs in a confined media worker; see the [crate](#) documentation.

#### Errors

Returns [`MediaError::InputMissing`](#mediaerror) when the input does not exist,
[`MediaError::OutputMissing`](#mediaerror) when the worker reported success but wrote
nothing, and propagates tag-writing and worker failures otherwise.

### remux_trimmed

```rust
pub fn remux_trimmed(input: &Path, output: &Path, trim: TrimRange) -> Result<MediaOutcome>
```

Remux a progressive clear M4B/M4A with an optional media-time trim (chapter split).

#### Errors

Returns an error when the operation fails.

### package_m4b_from_mp3

```rust
pub async fn package_m4b_from_mp3(
    req: PackageM4bRequest,
) -> Result<(MediaOutcome, Vec<(String, u64)>)>
```

Decode/remux ordered parts → progressive M4B.

Clear AAC MP4/M4A/M4B parts are remuxed without re-encoding. MP3 parts are
decoded and encoded to AAC-LC via a streaming path.

Returns the output path plus chapter list `(title, start_ms)`.
Runs in a confined media worker; see the [crate](#) documentation.

#### Errors

Returns [`MediaError::InputMissing`](#mediaerror) when a part is missing, and propagates
packaging and worker failures otherwise.

### package_m4b_from_pcm

```rust
pub fn package_m4b_from_pcm(
    pcm: &[i16],
    sample_rate: u32,
    channels: u16,
    output: &Path,
    chapter_boundaries_ms: &[(String, u64)],
) -> Result<(MediaOutcome, Vec<(String, u64)>)>
```

Encode interleaved PCM to AAC-LC and mux an M4B (test / internal helper).

Feeds PCM in bounded chunks so callers with large buffers still do not
duplicate the full stream as AAC access units in memory.

#### Errors

Returns an error when the operation fails.

### init_pool

```rust
pub fn init_pool(pool: MediaPool) -> Result<(), MediaPool>
```

Install the process-wide pool if none is installed yet.

#### Errors

Returns the rejected pool when one was already installed. Use
[`replace_pool`](#replace_pool) to swap a pool that is already running.

### init_pool_from_config

```rust
pub fn init_pool_from_config(config: &MediaConfig)
```

Build the pool from `[media]` config and install it, logging what it will do.

Safe to call more than once, which is what a config reload does. An
unchanged `[media]` leaves the running pool alone; a changed one swaps in a
new pool for subsequent jobs and lets the old one drain. See
[`replace_pool`](#replace_pool).

### pool

```rust
pub fn pool() -> Arc<MediaPool>
```

The process-wide pool, creating a default one if startup never installed it.

Returns an owned handle on purpose. Holding it for the length of a job is
what lets a reload swap the pool without disturbing work already running.

### replace_pool

```rust
pub fn replace_pool(pool: MediaPool) -> Option<Arc<MediaPool>>
```

Swap in `pool`, returning the one it replaced.

Safe to call with jobs in flight, because [`pool`](#pool) hands out an `Arc` and a
job holds it for its whole duration. The moment this returns, the outgoing
pool is unreachable, so it can only drain: its permits are released as its
jobs finish and it is dropped after the last one. Nothing is interrupted and
nothing waits.

In-flight jobs keep the isolation they started with, which is the only
possible answer — a worker's confinement is applied inside the child process
at spawn and cannot be changed afterwards.

While the outgoing pool drains, total codec concurrency can briefly reach the
old pool's in-flight count plus the new pool's limit. That overshoot only
shrinks, since the old pool never admits another job.

### encode_to_mp3

```rust
pub async fn encode_to_mp3(
    input: &Path,
    output: &Path,
    lame: &LameConfig,
    max_sample_rate: Option<u32>,
    max_output_bytes: Option<u64>,
) -> Result<MediaOutcome>
```

Re-encode audio to MP3 via Symphonia + LAME (classic Libation `DecryptToLossy`).

Runs in a confined media worker; see the module documentation.

#### Errors

Returns [`MediaError::InputMissing`](#mediaerror) when `input` does not exist, and
propagates encode and worker failures otherwise.

### remux_trimmed_async

```rust
pub async fn remux_trimmed_async(
    input: &Path,
    output: &Path,
    trim: TrimRange,
) -> Result<MediaOutcome>
```

Copy/trim a progressive M4B/M4A into a new file (chapter split helper).

Runs in a confined media worker; see the module documentation.

#### Errors

Returns [`MediaError::InputMissing`](#mediaerror) when `input` does not exist, and
propagates remux and worker failures otherwise.

### align_chapter_starts_async

```rust
pub async fn align_chapter_starts_async(
    path: &Path,
    chapters: &[(String, u64)],
    options: ChapterAlignOptions,
) -> Vec<(String, u64)>
```

Snap chapter starts to spoken-title onsets by local waveform analysis.

The synchronous [`align_chapter_starts`](#align_chapter_starts) decodes audio on the calling
thread, which on an async runtime blocks a worker for the length of the
analysis. Prefer this from async code.

Analysis never fails the acquire: a decode problem leaves the original
chapter starts in place, matching the synchronous behaviour.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, MediaError>;
```

Result alias for media packaging and worker operations in this crate.

## Constants

### BOOKCLERK_TOOL_NAME

```rust
pub const BOOKCLERK_TOOL_NAME: &str = "bookclerk";
```

Product / CLI name written into tool attribution tags.

### JAIL_BIN_ENV

```rust
pub const JAIL_BIN_ENV: &str = "BOOKCLERK_PLUGIN_JAIL";
```

Environment variable naming the jail launcher (shared with plugin hosts).

### JAIL_BIN_NAME

```rust
pub const JAIL_BIN_NAME: &str = "bookclerk-jail";
```

File name of the jail launcher when it sits beside the host / worker.

### WORKER_BIN_ENV

```rust
pub const WORKER_BIN_ENV: &str = "BOOKCLERK_MEDIA_WORKER";
```

Environment variable naming the worker binary explicitly.

### WORKER_BIN_NAME

```rust
pub const WORKER_BIN_NAME: &str = "bookclerk-media-worker";
```

File name of the worker binary when it sits beside the host executable.

### WORKER_ENFORCEMENT_ENV

```rust
pub const WORKER_ENFORCEMENT_ENV: &str = "BOOKCLERK_MEDIA_WORKER_ENFORCEMENT";
```

Environment variable the pool sets on each worker to select its failure mode.
