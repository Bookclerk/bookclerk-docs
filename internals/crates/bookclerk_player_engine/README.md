# bookclerk_player_engine

Native playback for the desktop window.

The window's custom scheme cannot stream audio by `Range`, so the window
does not play through its webview. The decoding happens in the confined
media worker (`bookclerk_media::MediaPool::open_pcm`); this crate does
what is left, all of it trusted signal processing on PCM:

- [`Stretcher`]: speed change without a pitch change (WSOLA).
- [`Resampler`]: the book's sample rate to the output device's.
- [`Renderer`]: both, plus channel mapping, pulled by the device.
- [`TrackPlayer`]: one track at a time, fed from a [`TrackOpener`](#trackopener), with
  [`PlayerEvent`](#playerevent)s for the page that drives it.

Nothing here touches an audio device or the OS: the desktop crate owns
the output stream and the system media controls, so this crate builds
and tests everywhere.

Style: `docs/contributing/code-documentation.md`. Design: the playback
ADR, `docs/adr/playback.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`TrackOpener`](#trackopener) | trait | Opens tracks. |
| [`TrackStream`](#trackstream) | trait | One track's decoded samples from a start position. |
| [`TrackHeader`](#trackheader) | struct | The layout and start of a decoded stream. |
| [`TrackPlayer`](#trackplayer) | struct | A one-track player over a shared `Renderer`. |
| [`OutputFormat`](#outputformat) | struct | The output device's layout. |
| [`Renderer`](#renderer) | struct | The playback pipeline between the decoder and the device. |
| [`Resampler`](#resampler) | struct | A converter for one stream of interleaved samples. |
| [`Stretcher`](#stretcher) | struct | A speed changer for one stream of interleaved samples. |
| [`PlayerEvent`](#playerevent) | enum | What the player reports, in the order it happens. |
| [`MAX_SPEED`](#max_speed) | constant | Fastest speed accepted. |
| [`MIN_SPEED`](#min_speed) | constant | Slowest speed accepted. |

## Traits

### TrackOpener

```rust
pub trait TrackOpener: Send + Sync + 'static {
    type Stream: TrackStream;
    fn open(
        &self,
        track: &str,
        start_ms: u64,
    ) -> impl Future<Output = Result<Self::Stream, String>> + Send;
}
```

Opens tracks.

#### TrackOpener::Stream

```rust
type Stream: TrackStream
```

What an opened track streams.

#### TrackOpener::open

```rust
fn open(
    &self,
    track: &str,
    start_ms: u64,
) -> impl Future<Output = Result<Self::Stream, String>> + Send
```

Opens `track` from `start_ms`.

##### Errors

Why the track cannot be played, in words for the listener.

### TrackStream

```rust
pub trait TrackStream: Send + 'static {
    fn header(&self) -> TrackHeader;
    fn next_chunk(&mut self) -> impl Future<Output = Option<Result<Vec<i16>, String>>> + Send;
}
```

One track's decoded samples from a start position.

#### TrackStream::header

```rust
fn header(&self) -> TrackHeader
```

The stream's layout and start.

#### TrackStream::next_chunk

```rust
fn next_chunk(&mut self) -> impl Future<Output = Option<Result<Vec<i16>, String>>> + Send
```

The next chunk of interleaved samples; `None` at the end.

##### Errors

Why the rest cannot be decoded, in words for a log.

## Structs

### TrackHeader

```rust
pub struct TrackHeader {
    pub sample_rate: u32,
    pub channels: u16,
    pub start_ms: u64,
    pub duration_ms: Option<u64>,
}
```

The layout and start of a decoded stream.

**Fields**

- `sample_rate`: Decoded frames per second.
- `channels`: Decoded channels: 1 or 2.
- `start_ms`: Where the first sample lies on the track, in milliseconds.
- `duration_ms`: The track's length in milliseconds, when known.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### TrackPlayer

```rust
pub struct TrackPlayer<O: TrackOpener> {
    /* private fields */
}
```

A one-track player over a shared [`Renderer`](#renderer).

#### TrackPlayer::new

```rust
pub fn new(
    opener: O,
    renderer: Arc<Mutex<Renderer>>,
    runtime: &Handle,
) -> (Self, UnboundedReceiver<PlayerEvent>)
```

A player that opens tracks with `opener` and feeds `renderer`.

##### Arguments

* `opener` - Opens a track's decoded stream.
* `renderer` - The pipeline the device pulls from.
* `runtime` - Where the feeder and the clock run.

##### Returns

The player, and the receiving end of its events.

#### TrackPlayer::load

```rust
pub fn load(&self, track: &str, start_ms: u64)
```

Loads `track` and positions it at `start_ms`, keeping play or
pause as it was. Replaces any track loaded before.

#### TrackPlayer::seek

```rust
pub fn seek(&self, position_ms: u64)
```

Moves to `position_ms` in the loaded track.

#### TrackPlayer::play

```rust
pub fn play(&self)
```

Plays the loaded track (or, after its end, plays it again from the
start).

#### TrackPlayer::pause

```rust
pub fn pause(&self)
```

Pauses.

#### TrackPlayer::set_speed

```rust
pub fn set_speed(&self, speed: f32)
```

Sets the speed (0.5 to 3; anything else is clamped).

#### TrackPlayer::stop

```rust
pub fn stop(&self)
```

Unloads the track and stops.

#### TrackPlayer::position_ms

```rust
pub fn position_ms(&self) -> u64
```

The position in the loaded track, in milliseconds.

Implements: `Debug`, `Drop`

### OutputFormat

```rust
pub struct OutputFormat {
    pub sample_rate: u32,
    pub channels: u16,
}
```

The output device's layout.

**Fields**

- `sample_rate`: Frames per second the device plays.
- `channels`: Interleaved channels the device takes; at least 1.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Renderer

```rust
pub struct Renderer {
    /* private fields */
}
```

The playback pipeline between the decoder and the device.

#### Renderer::new

```rust
pub fn new(output: OutputFormat) -> Self
```

A renderer for a device with `output`'s layout, paused, at speed 1.

#### Renderer::output

```rust
pub fn output(&self) -> OutputFormat
```

The device's layout.

#### Renderer::begin

```rust
pub fn begin(&mut self, sample_rate: u32, channels: u16, start_ms: u64) -> u64
```

Starts a new stream, dropping any before it.

##### Arguments

* `sample_rate` - The decoded rate, at least 1.
* `channels` - Decoded channels: 1 or 2.
* `start_ms` - Where the first sample lies on the track.

##### Returns

The stream's generation, which [`Self::push`](#rendererpush) and
[`Self::finish_input`](#rendererfinish_input) must name.

#### Renderer::clear

```rust
pub fn clear(&mut self)
```

Drops the stream: the device hears silence until the next begins.
A decoder still pushing to it is ignored.

#### Renderer::push

```rust
pub fn push(&mut self, generation: u64, samples: &[i16]) -> bool
```

Adds decoded samples to the stream of `generation`.

##### Returns

False when that stream was replaced or cleared.

#### Renderer::finish_input

```rust
pub fn finish_input(&mut self, generation: u64)
```

Marks the end of the stream of `generation`'s input.

#### Renderer::set_speed

```rust
pub fn set_speed(&mut self, speed: f32)
```

Changes the speed, clamped to what the stretcher allows.

#### Renderer::set_playing

```rust
pub fn set_playing(&mut self, playing: bool)
```

Plays or pauses; paused, the device hears silence and the position
holds.

#### Renderer::is_ended

```rust
pub fn is_ended(&self) -> bool
```

Whether the stream played out to its end.

#### Renderer::buffered_ms

```rust
pub fn buffered_ms(&self) -> u64
```

Decoded audio held and not yet played, in milliseconds of the
track.

#### Renderer::position_ms

```rust
pub fn position_ms(&self) -> Option<u64>
```

Where the output has got to on the track, in milliseconds; `None`
with no stream.

#### Renderer::render

```rust
pub fn render(&mut self, out: &mut [f32])
```

Fills one device buffer (interleaved in the device's channels).

Anything not filled from the stream is silence: while paused, with
no stream, while the decoder catches up, or after the end.

Implements: `Debug`

### Resampler

```rust
pub struct Resampler {
    /* private fields */
}
```

A converter for one stream of interleaved samples.

#### Resampler::new

```rust
pub fn new(from_rate: u32, to_rate: u32, channels: usize) -> Self
```

A converter from `from_rate` to `to_rate` for `channels` channels.

#### Resampler::push

```rust
pub fn push(&mut self, samples: &[f32])
```

Adds input samples (a whole number of frames).

#### Resampler::pull

```rust
pub fn pull(&mut self, out: &mut Vec<f32>, max_frames: usize, flush: bool) -> usize
```

Converts what the held input allows into `out`, up to `max_frames`.

##### Arguments

* `out` - Where converted samples go.
* `max_frames` - Most output frames to make.
* `flush` - The input ended: treat what lies past it as silence.

##### Returns

Frames made.

#### Resampler::pending_frames

```rust
pub fn pending_frames(&self) -> u64
```

Input frames held that output has not covered yet.

Implements: `Debug`

### Stretcher

```rust
pub struct Stretcher {
    /* private fields */
}
```

A speed changer for one stream of interleaved samples.

#### Stretcher::new

```rust
pub fn new(sample_rate: u32, channels: usize) -> Self
```

A stretcher for `channels` interleaved channels at `sample_rate`,
at speed 1.

##### Arguments

* `sample_rate` - Frames per second, at least 1.
* `channels` - Interleaved channels, at least 1.

#### Stretcher::speed

```rust
pub fn speed(&self) -> f32
```

The current speed.

#### Stretcher::set_speed

```rust
pub fn set_speed(&mut self, speed: f32)
```

Changes the speed, clamped to [`MIN_SPEED`](#min_speed)..=[`MAX_SPEED`](#max_speed). The
change applies from the next segment; samples already made keep the
old speed.

#### Stretcher::push

```rust
pub fn push(&mut self, samples: &[f32])
```

Adds decoded samples (a whole number of frames).

#### Stretcher::finish

```rust
pub fn finish(&mut self)
```

Marks the end of the input: what is left is played out.

#### Stretcher::buffered_frames

```rust
pub fn buffered_frames(&self) -> u64
```

Input frames held and not yet played.

#### Stretcher::source_frames

```rust
pub fn source_frames(&self) -> f64
```

Absolute input frames the output pulled so far stands for.

#### Stretcher::is_drained

```rust
pub fn is_drained(&self) -> bool
```

Whether the input ended and everything has been pulled.

#### Stretcher::pull

```rust
pub fn pull(&mut self, out: &mut Vec<f32>, max_frames: usize) -> usize
```

Moves up to `max_frames` stretched frames into `out`.

##### Returns

Frames moved; fewer than asked when more input is needed (or it
ended).

Implements: `Debug`

## Enums

### PlayerEvent

```rust
pub enum PlayerEvent {
    Loaded { duration_ms: Option<u64>, position_ms: u64 },
    Time { position_ms: u64 },
    Playing,
    Paused,
    Ended,
    Error { message: String },
}
```

What the player reports, in the order it happens.

**Variants**

- `Loaded`: A track's stream opened: its length, when known, and where it
  starts.
  - `duration_ms`: The track's length in milliseconds.
  - `position_ms`: Where playback starts on the track, in milliseconds.
- `Time`: The position while playing, about four times a second, and after
  each seek.
  - `position_ms`: Milliseconds into the track.
- `Playing`: Playback started or resumed.
- `Paused`: Playback paused.
- `Ended`: The track played to its end; the player is paused.
- `Error`: The track could not be opened or decoded further; the player is
  paused.
  - `message`: What went wrong, for the listener.

Implements: `Clone`, `Debug`, `PartialEq`

## Constants

### MAX_SPEED

```rust
pub const MAX_SPEED: f32 = 3.0;
```

Fastest speed accepted.

### MIN_SPEED

```rust
pub const MIN_SPEED: f32 = 0.5;
```

Slowest speed accepted.
