# bookclerk_media

Clear-media packaging: remux, metadata fix-up, and MP3 encode.

## Audience

Host acquire / media-worker code. Store plugins may call helpers that are
re-exported here, but DRM (Adrm / CENC) stays in those plugins. The
ISO-BMFF reading and writing underneath is [`bookclerk_mp4`](../bookclerk_mp4/README.md), which both this
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
| [`RangeSource`](#rangesource) | trait | Where a file's bytes come from: a local file, a storage plugin's ranged read, or a buffer in tests. |
| [`PcmFormat`](#pcmformat) | struct | The PCM layout of a decoded stream. |
| [`ChapterAlignOptions`](#chapteralignoptions) | struct | Tunables for silence / speech-onset chapter snapping. |
| [`FixupRequest`](#fixuprequest) | struct | Request to fix up audiobook metadata after decrypt / download. |
| [`OpusEncodeOptions`](#opusencodeoptions) | struct | How an Opus copy is encoded. |
| [`PackageM4bRequest`](#packagem4brequest) | struct | Request to package ordered MP3 parts into one M4B. |
| [`EncodedSample`](#encodedsample) | struct | A short sample of a file, encoded by a worker. |
| [`PcmStream`](#pcmstream) | struct | Decoded samples of one file from one start position. |
| [`SampleStream`](#samplestream) | struct | A book sample being cut: the MP3 bytes a worker sends, as it encodes them. |
| [`MediaPool`](#mediapool) | struct | Bounded pool of confined media workers. |
| [`MediaPoolConfig`](#mediapoolconfig) | struct | Pool sizing and isolation settings. |
| [`AudioProbe`](#audioprobe) | struct | What a probe found in one audio file. |
| [`FileRange`](#filerange) | struct | A file on this machine, read a range at a time. |
| [`MediaOutcome`](#mediaoutcome) | struct | Outcome of a successful media operation. |
| [`OutChannels`](#outchannels) | enum | How many channels the caller wants decoded PCM in. |
| [`MediaError`](#mediaerror) | enum | Errors from in-process codecs, MP4 plumbing, and confined media workers. |
| [`MediaJob`](#mediajob) | enum | A single codec operation. |
| [`MediaJobOutput`](#mediajoboutput) | enum | What a finished job produced. |
| [`MediaJobReply`](#mediajobreply) | enum | Wire form of a finished job. `Result` is not `Serialize`, so the worker and the pool exchange this instead. |
| [`Confinement`](#confinement) | enum | How strictly a worker must be confined before it will process a job. |
| [`decode_pcm_bytes`](#decode_pcm_bytes) | function | Decodes a whole media file held in memory to PCM, the path the `media_decode` fuzz target and its replay test drive. |
| [`write_output_within_budget`](#write_output_within_budget) | function | Appends `data` only when the result stays within `max`. |
| [`check_archive_report`](#check_archive_report) | function | Checks a worker's report against the limits and the files on disk. |
| [`extract_archive`](#extract_archive) | function | Expands `input` into the empty folder `output_dir` in a confined media worker, then checks the result. |
| [`align_chapter_starts`](#align_chapter_starts) | function | Snap chapter starts just before nearby speech onsets (spoken chapter titles). |
| [`scale_chapters_to_duration`](#scale_chapters_to_duration) | function | Scale rebased chapter starts so they span the probed plain-audio duration. |
| [`bookclerk_tool_tag`](#bookclerk_tool_tag) | function | `bookclerk <version>` string embedded in encoder / freeform TOOL tags. |
| [`fixup_audiobook`](#fixup_audiobook) | function | Apply metadata tags, optional cover embed, and optional chapters. |
| [`package_keeping_codec`](#package_keeping_codec) | function | Packages `parts` into one M4B at `output` keeping their audio, with a chapter at the start of each part titled from `titles`. |
| [`remux_trimmed`](#remux_trimmed) | function | Remux a progressive clear M4B/M4A with an optional media-time trim (chapter split). |
| [`encode_to_opus_native`](#encode_to_opus_native) | function | Encodes the audio in `input` to an Ogg Opus file at `output`. |
| [`package_m4b_from_mp3`](#package_m4b_from_mp3) | function | Decode/remux ordered parts → progressive M4B. |
| [`package_m4b_from_pcm`](#package_m4b_from_pcm) | function | Encode interleaved PCM to AAC-LC and mux an M4B (test / internal helper). |
| [`serve_pcm`](#serve_pcm) | function | Serves one PCM decode over `input` and `output` (the worker's stdin and stdout): reads `Open`, then streams `Format`, `Pcm` frames and `End`, or `Failed` when the file cannot be decoded. Opened with `Sample` instead, streams `Format`, `Encoded` frames and `End`. |
| [`init_pool`](#init_pool) | function | Install the process-wide pool if none is installed yet. |
| [`init_pool_from_config`](#init_pool_from_config) | function | Build the pool from `[media]` config and install it, logging what it will do. |
| [`pool`](#pool) | function | The process-wide pool, creating a default one if startup never installed it. |
| [`replace_pool`](#replace_pool) | function | Swap in `pool`, returning the one it replaced. |
| [`encode_to_mp3`](#encode_to_mp3) | function | Re-encode audio to MP3 via Symphonia + LAME (classic Libation `DecryptToLossy`). |
| [`encode_to_opus`](#encode_to_opus) | function | Encode audio to Ogg Opus via Symphonia + libopus. |
| [`remux_trimmed_async`](#remux_trimmed_async) | function | Copy/trim a progressive M4B/M4A into a new file (chapter split helper). |
| [`align_chapter_starts_async`](#align_chapter_starts_async) | function | Snap chapter starts to spoken-title onsets by local waveform analysis. |
| [`Result`](#result) | type alias | Result alias for media packaging and worker operations in this crate. |
| [`MAX_SAMPLE_RATE_HZ`](#max_sample_rate_hz) | constant | Highest sample rate a decoded stream may report, in Hz. Anything above is refused as malformed rather than handed to an encoder. |
| [`COVER_THREADS_ENV`](#cover_threads_env) | constant | Environment variable carrying the cover worker's decode thread count. |
| [`COVER_WORKER_ARG`](#cover_worker_arg) | constant | First worker argument that selects the cover server. |
| [`BOOKCLERK_TOOL_NAME`](#bookclerk_tool_name) | constant | Product / CLI name written into tool attribution tags. |
| [`MAX_OPUS_KBPS`](#max_opus_kbps) | constant | Highest bitrate an Opus copy is encoded at, in kbit/s (per stream; the Opus limit for stereo). |
| [`MIN_OPUS_KBPS`](#min_opus_kbps) | constant | Lowest bitrate an Opus copy is encoded at, in kbit/s. |
| [`MAX_SAMPLE_BYTES`](#max_sample_bytes) | constant | Largest sample the host takes from a worker (16 MiB): ten minutes of mono MP3 at 64 kbit/s is under 5 MiB. |
| [`SAMPLE_TIMEOUT`](#sample_timeout) | constant | Longest a worker may take to cut one sample, start to end. |
| [`JAIL_BIN_ENV`](#jail_bin_env) | constant | Environment variable naming the jail launcher (shared with plugin hosts). |
| [`JAIL_BIN_NAME`](#jail_bin_name) | constant | File name of the jail launcher when it sits beside the host / worker. |
| [`WORKER_BIN_ENV`](#worker_bin_env) | constant | Environment variable naming the worker binary explicitly. |
| [`WORKER_BIN_NAME`](#worker_bin_name) | constant | File name of the worker binary when it sits beside the host executable. |
| [`WORKER_ENFORCEMENT_ENV`](#worker_enforcement_env) | constant | Environment variable the pool sets on each worker to select its failure mode. |
| [`TAG_WORKER_ARG`](#tag_worker_arg) | constant | First worker argument that selects the tag reader. |

## Re-exports

- [`bookclerk_cover::CoverSet`](../bookclerk_cover/README.md#coverset)
- [`bookclerk_cover::CoverSize`](../bookclerk_cover/README.md#coversize)
- [`bookclerk_mp4::brand_durations_from_chapter_info`](../bookclerk_mp4/README.md#brand_durations_from_chapter_info)
- [`bookclerk_mp4::brand_trim_range`](../bookclerk_mp4/README.md#brand_trim_range)
- [`bookclerk_mp4::rebase_chapters_after_brand_trim`](../bookclerk_mp4/README.md#rebase_chapters_after_brand_trim)
- [`bookclerk_mp4::runtime_length_ms_from_chapter_info`](../bookclerk_mp4/README.md#runtime_length_ms_from_chapter_info)
- [`bookclerk_mp4::BrandDurations`](../bookclerk_mp4/README.md#branddurations)
- [`bookclerk_mp4::extract_mp4a_config`](../bookclerk_mp4/README.md#extract_mp4a_config)
- [`bookclerk_mp4::parse_mp4`](../bookclerk_mp4/README.md#parse_mp4)
- [`bookclerk_mp4::track_duration_ms`](../bookclerk_mp4/README.md#track_duration_ms)
- [`bookclerk_mp4::Mp4aConfig`](../bookclerk_mp4/README.md#mp4aconfig)
- [`bookclerk_mp4::SampleEntryKind`](../bookclerk_mp4/README.md#sampleentrykind)
- [`bookclerk_mp4::TrimRange`](../bookclerk_mp4/README.md#trimrange)
- [`bookclerk_tags::ByteSpan`](../bookclerk_tags/README.md#bytespan)
- [`bookclerk_tags::TagRecord`](../bookclerk_tags/README.md#tagrecord)
- [`pcm_wire::PcmHeader`](pcm_wire.md#pcmheader)
- [`pcm_wire::PCM_WORKER_ARG`](pcm_wire.md#pcm_worker_arg)

## Modules

- [`capabilities`](capabilities.md): What this build of the media worker can read and write.
- [`pcm_wire`](pcm_wire.md): Frames the host and `bookclerk-media-worker pcm` exchange over the worker's stdio.

## Traits

### RangeSource

```rust
pub trait RangeSource: Send + Sync {
    fn read_range(&self, span: ByteSpan) -> impl Future<Output = Result<Vec<u8>>> + Send;
}
```

Where a file's bytes come from: a local file, a storage plugin's ranged
read, or a buffer in tests.

#### RangeSource::read_range

```rust
fn read_range(&self, span: ByteSpan) -> impl Future<Output = Result<Vec<u8>>> + Send
```

Reads exactly `span` from the file.

##### Arguments

* `span` - Lies inside the file; the host checked it.

##### Errors

Any I/O failure; the read of that file then fails.

Implemented by: `FileRange`

## Structs

### PcmFormat

```rust
pub struct PcmFormat {
    pub sample_rate: u32,
    pub channels: u16,
}
```

The PCM layout of a decoded stream.

**Fields**

- `sample_rate`: Samples per second per channel, in Hz; between 1 and
  [`MAX_SAMPLE_RATE_HZ`](#max_sample_rate_hz).
- `channels`: Interleaved channels in every chunk: 1 (mono) or 2 (stereo).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

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
- `chapters`: Chapter titles + start offsets in milliseconds, embedded as a
  QuickTime chapter track in an M4B and as ID3 `CHAP`/`CTOC` frames in
  an MP3.
- `replace_chapters`: When true, replace any chapters the file already has (an M4B chapter
  list/track, or MP3 `CHAP`/`CTOC` frames) with [`Self::chapters`](#fixuprequest)
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

### OpusEncodeOptions

```rust
pub struct OpusEncodeOptions {
    pub bitrate_kbps: u16,
    pub mono: bool,
}
```

How an Opus copy is encoded.

**Fields**

- `bitrate_kbps`: Target bitrate in kbit/s, between [`MIN_OPUS_KBPS`](#min_opus_kbps) and
  [`MAX_OPUS_KBPS`](#max_opus_kbps) (others are clamped). Opus encodes in variable
  bitrate around it.
- `mono`: Mix down to one channel.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PackageM4bRequest

```rust
pub struct PackageM4bRequest {
    pub parts: Vec<PathBuf>,
    pub output: PathBuf,
    pub chapter_titles: Vec<String>,
    pub keep_codec: bool,
}
```

Request to package ordered MP3 parts into one M4B.

**Fields**

- `parts`: Ordered MP3 chapter / part files.
- `output`: Destination `.m4b` path.
- `chapter_titles`: Optional chapter titles (length may match `parts`).
- `keep_codec`: Keep Opus, FLAC, ALAC, E-AC-3 or AC-4 audio as it is inside the
  M4B instead of encoding it to AAC (`output.keep_codec_in_m4b`).
  Parts that cannot be kept together are encoded to AAC as usual.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### EncodedSample

```rust
pub struct EncodedSample {
    pub header: PcmHeader,
    pub mp3: Vec<u8>,
}
```

A short sample of a file, encoded by a worker.

**Fields**

- `header`: The encoded layout (mono), where the sample starts, and the file's
  stated duration.
- `mp3`: The MP3 stream: constant bit rate, no tag frames.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PcmStream

```rust
pub struct PcmStream {
    /* private fields */
}
```

Decoded samples of one file from one start position.

Dropping it stops the worker.

#### PcmStream::header

```rust
pub fn header(&self) -> PcmHeader
```

The sample rate, channels, actual start and stated duration.

#### PcmStream::next_chunk

```rust
pub async fn next_chunk(&mut self) -> Option<Result<Vec<i16>>>
```

The next chunk of interleaved samples: a whole number of frames in
[`Self::header`](#pcmstreamheader)'s layout.

##### Returns

`None` once the file decoded to its end.

##### Errors

[`MediaError::Native`](#mediaerror) when the worker could not decode the rest,
[`MediaError::Io`](#mediaerror) when the source failed, and
[`MediaError::Worker`](#mediaerror) when the worker died, stalled or broke the
protocol. The stream ends after an error.

Implements: `Debug`

### SampleStream

```rust
pub struct SampleStream {
    /* private fields */
}
```

A book sample being cut: the MP3 bytes a worker sends, as it encodes
them.

The bytes are checked as they arrive: a worker that sends PCM instead,
more than [`MAX_SAMPLE_BYTES`](#max_sample_bytes) in all, or misses [`SAMPLE_TIMEOUT`](#sample_timeout)
(counted from the worker's start) ends the stream with an error.
Dropping it stops the worker and frees its sample slot.

#### SampleStream::header

```rust
pub fn header(&self) -> PcmHeader
```

The encoded layout (mono), where the sample starts, and the file's
stated duration.

#### SampleStream::next_bytes

```rust
pub async fn next_bytes(&mut self) -> Option<Result<Vec<u8>>>
```

The next MP3 bytes the worker encoded.

##### Returns

`None` once the sample is complete (or after an error was returned).

##### Errors

What the worker reports, or [`MediaError::Worker`](#mediaerror) when it sends
samples instead of bytes, more than [`MAX_SAMPLE_BYTES`](#max_sample_bytes) in all, or
misses [`SAMPLE_TIMEOUT`](#sample_timeout). The stream ends after an error.

#### SampleStream::collect

```rust
pub async fn collect(self) -> Result<EncodedSample>
```

Reads the sample to its end.

##### Errors

As [`Self::next_bytes`](#samplestreamnext_bytes).

Implements: `Debug`

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

#### MediaPool::process_cover

```rust
pub async fn process_cover(&self, cover: Vec<u8>) -> Result<CoverSet>
```

Turns one untrusted downloaded cover image into the JPEG sizes the
host serves.

Covers do not take a job slot: they run on one long-lived worker
(jailed to no path and no network) that decodes on as many threads as
this pool has workers, so search results do not queue behind an
acquire.

##### Arguments

* `cover` - The downloaded bytes, at most
  [`bookclerk_cover::MAX_INPUT_BYTES`](../bookclerk_cover/README.md#max_input_bytes).

##### Errors

[`MediaError::Cover`](#mediaerror) when the image is refused or takes longer than
30 s, [`MediaError::NotIsolated`](#mediaerror) when isolation is required but no
worker was found, and [`MediaError::Worker`](#mediaerror) when the worker cannot be
started or dies.

#### MediaPool::read_tags

```rust
pub async fn read_tags<S: RangeSource>(&self, size: u64, source: &S) -> Result<TagRecord>
```

Reads the metadata embedded in one untrusted media file (`tags.read`):
text tags, chapters, running time, where the cover lies, and a digest
of the metadata region.

Tag reads do not take a job slot: they run on one long-lived worker,
jailed to no path and no network, that asks for the byte ranges it
needs, which this call fetches from `source`. Only the metadata region
and a few headers are fetched, never the audio. Read a page of files
by awaiting several calls at once; up to 16 files are read at a time.

##### Arguments

* `size` - The file's size in bytes.
* `source` - Where its bytes come from (a local file, a storage
  plugin's ranged read).

##### Errors

[`MediaError::Tags`](#mediaerror) when the file is not an MP4, MP3 or FLAC file or
is malformed, [`MediaError::Io`](#mediaerror) when `source` fails,
[`MediaError::NotIsolated`](#mediaerror) when isolation is required but no worker
was found, and [`MediaError::Worker`](#mediaerror) when the worker cannot be
started, dies, stalls for 30 s, or answers outside the reader's
limits (the worker is then restarted).

#### MediaPool::open_pcm

```rust
pub async fn open_pcm<S: RangeSource + 'static>(
    &self,
    size: u64,
    start_ms: u64,
    source: S,
) -> Result<PcmStream>
```

Decodes one audio file to PCM for a player, from `start_ms` to its
end (`bookclerk-media-worker pcm`).

Each call starts its own worker, jailed to no path and no network,
which asks for the byte ranges its demuxer reads; this call fetches
them from `source`. It does not take a job slot. The stream holds
at most a few chunks ahead of the player, so a paused player keeps
the worker waiting rather than decoding the whole book. Seeking is a
new call; dropping the stream stops its worker. With isolation off,
playback is refused, never decoded in this process.

##### Arguments

* `size` - The file's size in bytes.
* `start_ms` - Where to start, in milliseconds into the file.
* `source` - Where its bytes come from.

##### Errors

[`MediaError::NotIsolated`](#mediaerror) when isolation is off or no worker was
found, [`MediaError::Native`](#mediaerror) when the file cannot be decoded,
[`MediaError::Io`](#mediaerror) when `source` fails, and [`MediaError::Worker`](#mediaerror)
when the worker cannot be started, stalls for 30 s before its
format, or breaks the protocol.

#### MediaPool::stream_sample

```rust
pub async fn stream_sample<S: RangeSource + 'static>(
    &self,
    size: u64,
    start_ms: u64,
    length_ms: u64,
    source: S,
) -> Result<SampleStream>
```

Starts cutting a short sample of one audio file, folded to mono and
encoded as MP3, for a book's details (`bookclerk-media-worker pcm`,
opened with `Sample`). The bytes come back as the worker encodes
them, so a player can start on the first ones.

Like [`Self::open_pcm`](#mediapoolopen_pcm), each call starts its own worker, jailed to
no path and no network, which asks for the byte ranges it reads. It
does not take a job slot; samples have their own limit, as many at
once as there are job slots, and a call waits for one. The stream
holds its slot until it is dropped. The worker decodes only the
sample's span, and the bytes returned are its encoding, never the
file's own. With isolation off, samples are refused, never cut in
this process.

##### Arguments

* `size` - The file's size in bytes.
* `start_ms` - Where the sample starts, in milliseconds into the
  file.
* `length_ms` - How long it runs; at most
  [`crate::pcm_wire::MAX_SAMPLE_MS`](pcm_wire.md#max_sample_ms), and less when the file ends
  first.
* `source` - Where the file's bytes come from.

##### Errors

[`MediaError::NotIsolated`](#mediaerror) when isolation is off or no worker was
found, [`MediaError::Native`](#mediaerror) when the file cannot be opened,
[`MediaError::Io`](#mediaerror) when `source` fails, and [`MediaError::Worker`](#mediaerror)
when the worker cannot be started, breaks the protocol or states no
format in time. Later failures (a decode error, more than
[`crate::MAX_SAMPLE_BYTES`](#max_sample_bytes), or more than [`crate::SAMPLE_TIMEOUT`](#sample_timeout)
in all) end the stream with an error.

#### MediaPool::cut_sample

```rust
pub async fn cut_sample<S: RangeSource + 'static>(
    &self,
    size: u64,
    start_ms: u64,
    length_ms: u64,
    source: S,
) -> Result<EncodedSample>
```

Cuts a short sample of one audio file to its end: [`Self::stream_sample`](#mediapoolstream_sample)
read whole.

##### Errors

As [`Self::stream_sample`](#mediapoolstream_sample), and whatever ends its stream.

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

#### MediaPool::probe_file

```rust
pub async fn probe_file(&self, path: &Path) -> Result<AudioProbe>
```

Probes the audio file at `path` on this machine.

##### Errors

[`crate::MediaError::Io`](#mediaerror) when the file cannot be read, and every
error of [`Self::read_tags`]: the file is not MP4, MP3 or FLAC, or
isolation is required and no worker was found.

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

### AudioProbe

```rust
pub struct AudioProbe {
    pub content_type: &'static str,
    pub bitrate_kbps: Option<u32>,
    pub lossless: bool,
    pub codec: Option<AudioCodec>,
    pub format: AudioFormat,
    pub duration_ms: Option<u64>,
}
```

What a probe found in one audio file.

**Fields**

- `content_type`: Media type of the container (`audio/mp4`, `audio/mpeg`,
  `audio/flac`).
- `bitrate_kbps`: Average bitrate in kilobits per second: the file's size over its
  running time, cover and tags included. `None` when the file states
  no running time.
- `lossless`: True when the audio encoding is lossless (FLAC, ALAC). An MP4 file
  whose encoding the reader does not know counts as lossy.
- `codec`: The audio encoding, when the tag reader knows it.
- `format`: Channels, sample rate and AAC profile, as far as the headers state
  them.
- `duration_ms`: The running time a player sees, in milliseconds, as the container
  states it (an MP4's movie duration, which edit lists shape; an Opus
  stream's samples past its pre-skip). `None` when the file states
  none, never zero.

#### AudioProbe::from_record

```rust
pub fn from_record(size: u64, record: &TagRecord) -> Self
```

The probe of a file of `size` bytes whose tags read as `record`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### FileRange

```rust
pub struct FileRange {
    /* private fields */
}
```

A file on this machine, read a range at a time.

#### FileRange::new

```rust
pub fn new(path: &Path) -> Self
```

A source over the file at `path`. Nothing is opened until a range is
read.

Implements: `Clone`, `Debug`, `RangeSource`

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

### OutChannels

```rust
pub enum OutChannels {
    SourceUpToStereo,
    Mono,
}
```

How many channels the caller wants decoded PCM in.

**Variants**

- `SourceUpToStereo`: The source's own count, folded to stereo when it has more than two.
- `Mono`: One channel: the average of the source's first two.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### MediaError

```rust
pub enum MediaError {
    InputMissing(PathBuf),
    OutputMissing(PathBuf),
    Mp4(String),
    Native(String),
    Worker { job: &'static str, detail: String },
    NotIsolated { job: &'static str, detail: String },
    Cover(String),
    Tags(String),
    Archive(String),
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
- `Cover`: A cover image was refused: not a supported image, too large, or
  undecodable. The message comes from the cover pipeline.
- `Tags`: A file's embedded tags could not be read: not an MP4, MP3 or FLAC
  file, or a malformed one. The message comes from the tag reader.
- `Archive`: An uploaded archive could not be expanded, or the worker's report of
  the expansion did not hold up when the host checked it.
- `Io`: Local filesystem I/O failure.
- `Other`: Catch-all for wrapped media failures.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<Mp4Error>`

### MediaJob

```rust
pub enum MediaJob {
    EncodeMp3 { input: PathBuf, output: PathBuf, lame: Box<LameConfig>, max_sample_rate: Option<u32>, max_output_bytes: Option<u64> },
    EncodeOpus { input: PathBuf, output: PathBuf, options: OpusEncodeOptions, max_output_bytes: Option<u64> },
    RemuxTrimmed { input: PathBuf, output: PathBuf, trim: TrimRange },
    Fixup { request: Box<FixupRequest> },
    PackageM4b { request: Box<PackageM4bRequest> },
    AlignChapters { path: PathBuf, chapters: Vec<(String, u64)>, options: ChapterAlignOptions },
    ExtractArchive { input: PathBuf, output_dir: PathBuf, limits: Limits },
}
```

A single codec operation.

These are the CPU-bound paths that decode or encode attacker-influenced
audio through LAME, FDK-AAC and libopus, all of which are C libraries.

**Variants**

- `EncodeMp3`: Re-encode audio to MP3 via Symphonia and LAME.
  - `input`: Absolute path of the source audio to re-encode.
  - `output`: Absolute path where the MP3 should be written.
  - `lame`: LAME encoder knobs for this encode.
  - `max_sample_rate`: Optional ceiling for output sample rate in Hz (`None` = leave source).
  - `max_output_bytes`: Hard cap on bytes written to `output`. `None` does not cap the file.
- `EncodeOpus`: Encode audio to Ogg Opus via Symphonia and libopus.
  - `input`: Absolute path of the source audio to encode.
  - `output`: Absolute path where the `.opus` file should be written.
  - `options`: Bitrate and channel choice.
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
- `ExtractArchive`: Expand an uploaded ZIP or tar archive into an empty folder under
  caps ([`bookclerk_archive::extract`](../bookclerk_archive/README.md#extract)).
  - `input`: Absolute path of the archive.
  - `output_dir`: Absolute path of the empty folder the files go into.
  - `limits`: The caps for this archive.

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
    Extracted { report: Box<ExtractReport> },
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
- `Extracted`: What an archive expansion wrote and refused.
  - `report`: The walk's report; the host re-checks it before trusting it.

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

### decode_pcm_bytes

```rust
pub fn decode_pcm_bytes<F>(
    bytes: Vec<u8>,
    out: OutChannels,
    on_pcm: F,
) -> Result<PcmFormat>
where
    F: FnMut(PcmFormat, &[i16]) -> Result<()>,
```

Decodes a whole media file held in memory to PCM, the path the
`media_decode` fuzz target and its replay test drive.

#### Arguments

* `bytes` - The file.
* `out` - The channel layout wanted.
* `on_pcm` - Called once per decoded packet with the stream's layout and
  that packet's interleaved samples (a whole number of frames).

#### Errors

[`MediaError::Native`](#mediaerror) when no known container or audio track is found,
no decoder handles the codec, the demuxer fails, or the file decodes to
no PCM at all; or what `on_pcm` returns.

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

### check_archive_report

```rust
pub fn check_archive_report(
    report: &ExtractReport,
    output_dir: &Path,
    limits: &Limits,
) -> Result<()>
```

Checks a worker's report against the limits and the files on disk.

#### Errors

[`MediaError::Archive`](#mediaerror) naming what does not hold.

### extract_archive

```rust
pub async fn extract_archive(
    input: &Path,
    output_dir: &Path,
    limits: Limits,
) -> Result<ExtractReport>
```

Expands `input` into the empty folder `output_dir` in a confined media
worker, then checks the result.

The report is validated against `limits` (`ExtractReport::validate`),
and every file it lists must be a regular file (not a link) at its path
under `output_dir`, with the size the report states. A stopped walk is
returned as is: the caller discards the folder.

#### Arguments

* `input` - The archive.
* `output_dir` - An empty folder; created if missing.
* `limits` - The caps for this archive.

#### Errors

[`MediaError::Archive`](#mediaerror) when `output_dir` is not empty, the input is not
a recognised archive, or the report does not hold up;
[`MediaError::NotIsolated`](#mediaerror) when isolation is required and no worker is
available; worker and I/O failures otherwise.

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

### package_keeping_codec

```rust
pub fn package_keeping_codec(
    parts: &[PathBuf],
    output: &Path,
    titles: &[String],
) -> Result<Option<Vec<(String, u64)>>>
```

Packages `parts` into one M4B at `output` keeping their audio, with a
chapter at the start of each part titled from `titles`.

#### Returns

The chapters `(title, start ms)`, or `None` when the parts cannot be
kept together and should be transcoded instead.

#### Errors

A read or write error once packaging has started.

### remux_trimmed

```rust
pub fn remux_trimmed(input: &Path, output: &Path, trim: TrimRange) -> Result<MediaOutcome>
```

Remux a progressive clear M4B/M4A with an optional media-time trim (chapter split).

#### Errors

Returns an error when the operation fails.

### encode_to_opus_native

```rust
pub fn encode_to_opus_native(
    input: &Path,
    output: &Path,
    options: OpusEncodeOptions,
    max_output_bytes: Option<u64>,
) -> Result<MediaOutcome>
```

Encodes the audio in `input` to an Ogg Opus file at `output`.

#### Arguments

* `input` - Any file the decode front opens.
* `output` - Where the `.opus` file is written (replaced if present).
* `options` - Bitrate and channel choice.
* `max_output_bytes` - Fail rather than write past this many bytes.

#### Errors

[`MediaError::InputMissing`](#mediaerror) when the input does not exist; what the
decode front returns for audio it cannot open or decode;
[`MediaError::Native`](#mediaerror) when libopus refuses a setting or a frame, the
input decodes to nothing, or the byte budget would be passed; an I/O
error when the output cannot be written.

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

### serve_pcm

```rust
pub fn serve_pcm<R, W>(input: R, output: W) -> Result<(), String>
where
    R: Read + Send + Sync + 'static,
    W: Write + Send + Sync + 'static,
```

Serves one PCM decode over `input` and `output` (the worker's stdin and
stdout): reads `Open`, then streams `Format`, `Pcm` frames and `End`, or
`Failed` when the file cannot be decoded. Opened with `Sample` instead,
streams `Format`, `Encoded` frames and `End`.

#### Arguments

* `input` - Request frames from the host.
* `output` - Reply frames to the host.

#### Errors

Why the stream broke: the host sent no `Open` or `Sample`, or a pipe
failed. A file
that cannot be decoded is not an error here: the host is told with
`Failed`.

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

### encode_to_opus

```rust
pub async fn encode_to_opus(
    input: &Path,
    output: &Path,
    options: OpusEncodeOptions,
    max_output_bytes: Option<u64>,
) -> Result<MediaOutcome>
```

Encode audio to Ogg Opus via Symphonia + libopus.

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

### MAX_SAMPLE_RATE_HZ

```rust
pub const MAX_SAMPLE_RATE_HZ: u32 = 768_000;
```

Highest sample rate a decoded stream may report, in Hz. Anything above
is refused as malformed rather than handed to an encoder.

### COVER_THREADS_ENV

```rust
pub const COVER_THREADS_ENV: &str = "BOOKCLERK_COVER_THREADS";
```

Environment variable carrying the cover worker's decode thread count.

### COVER_WORKER_ARG

```rust
pub const COVER_WORKER_ARG: &str = "covers";
```

First worker argument that selects the cover server.

### BOOKCLERK_TOOL_NAME

```rust
pub const BOOKCLERK_TOOL_NAME: &str = "bookclerk";
```

Product / CLI name written into tool attribution tags.

### MAX_OPUS_KBPS

```rust
pub const MAX_OPUS_KBPS: u16 = 510;
```

Highest bitrate an Opus copy is encoded at, in kbit/s (per stream; the
Opus limit for stereo).

### MIN_OPUS_KBPS

```rust
pub const MIN_OPUS_KBPS: u16 = 6;
```

Lowest bitrate an Opus copy is encoded at, in kbit/s.

### MAX_SAMPLE_BYTES

```rust
pub const MAX_SAMPLE_BYTES: usize = _;
```

Largest sample the host takes from a worker (16 MiB): ten minutes of
mono MP3 at 64 kbit/s is under 5 MiB.

### SAMPLE_TIMEOUT

```rust
pub const SAMPLE_TIMEOUT: Duration = _;
```

Longest a worker may take to cut one sample, start to end.

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

### TAG_WORKER_ARG

```rust
pub const TAG_WORKER_ARG: &str = "tags";
```

First worker argument that selects the tag reader.
