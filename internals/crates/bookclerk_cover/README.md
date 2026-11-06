# bookclerk_cover

Cover image pipeline for untrusted cover art.

A cover reaches the host as bytes a plugin downloaded from some public
website. Nothing about them is trusted: not the `Content-Type`, not the
extension, not the dimensions the header claims. [`process`](#process) turns such
bytes into a fixed set of JPEGs the host can store and serve:

1. [`sniff`](#sniff) reads the file signature and accepts only JPEG, PNG and WebP.
2. The `image` crate decodes it with that format forced (never guessed)
   and with [`MAX_DIMENSION`](#max_dimension) and [`MAX_DECODE_ALLOC`](#max_decode_alloc) as hard limits.
3. Transparency is flattened onto white, so every output is plain RGB.
4. `fast_image_resize` (Lanczos3) fits the image into each
   [`CoverSize`](#coversize), never enlarging it.
5. `jpeg-encoder` writes each size as a 4:4:4 JPEG (no chroma
   subsampling), at quality [`THUMB_QUALITY`](#thumb_quality) for the small sizes and
   [`FULL_QUALITY`](#full_quality) for the full one.

Every codec here is pure Rust, and the function works on a byte slice with
no I/O, so the fuzzer and the tests drive exactly what the media worker
runs. [`wire`](wire.md) is the frame format the host and the worker exchange, and
[`token`](token.md) signs the cover URLs the daemon serves.

| Item | Kind | Summary |
| --- | --- | --- |
| [`CoverSet`](#coverset) | struct | The JPEGs `process` produced for one cover. |
| [`CoverFormat`](#coverformat) | enum | An image format `sniff` recognizes. |
| [`CoverSize`](#coversize) | enum | One of the sizes `process` produces. |
| [`CoverError`](#covererror) | enum | Why `process` refused a cover. |
| [`sniff`](#sniff) | function | Reads the file signature of `bytes`. |
| [`process`](#process) | function | Sniffs, decodes, resizes and re-encodes one untrusted cover. |
| [`MAX_INPUT_BYTES`](#max_input_bytes) | constant | Largest input accepted, in bytes. Matches the plugin ABI's `maxCoverBytes`, the most a cover fetcher may hand the host. |
| [`MAX_DIMENSION`](#max_dimension) | constant | Widest or tallest image decoded, in pixels. Store covers are at most a few thousand pixels on a side; anything larger is refused before decoding. |
| [`MAX_DECODE_ALLOC`](#max_decode_alloc) | constant | Most memory the decoder may allocate for one image, in bytes. |
| [`FULL_MAX_EDGE`](#full_max_edge) | constant | Longest edge of the `CoverSize::Full` output, in pixels. |
| [`THUMB_QUALITY`](#thumb_quality) | constant | JPEG quality of the `CoverSize::Thumb` and `CoverSize::Card` outputs. |
| [`FULL_QUALITY`](#full_quality) | constant | JPEG quality of the `CoverSize::Full` output. |

## Modules

- [`token`](token.md): Signed cover tokens: the remote URL of a cover plus a truncated HMAC-SHA256 under a per-node key, so the cover route only fetches URLs the API itself handed out.
- [`wire`](wire.md): Frames the host and the cover worker exchange over the worker's stdio.

## Structs

### CoverSet

```rust
pub struct CoverSet {
    pub width: u32,
    pub height: u32,
    pub thumb: Vec<u8>,
    pub card: Vec<u8>,
    pub full: Vec<u8>,
}
```

The JPEGs [`process`](#process) produced for one cover.

**Fields**

- `width`: Width of the [`CoverSize::Full`](#coversize) image, in pixels.
- `height`: Height of the [`CoverSize::Full`](#coversize) image, in pixels.
- `thumb`: [`CoverSize::Thumb`](#coversize) JPEG.
- `card`: [`CoverSize::Card`](#coversize) JPEG.
- `full`: [`CoverSize::Full`](#coversize) JPEG.

#### CoverSet::get

```rust
pub fn get(&self, size: CoverSize) -> &[u8]
```

The JPEG for `size`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### CoverFormat

```rust
pub enum CoverFormat {
    Jpeg,
    Png,
    Webp,
}
```

An image format [`sniff`](#sniff) recognizes.

**Variants**

- `Jpeg`: JPEG (`FF D8 FF`).
- `Png`: PNG (the eight-byte PNG signature).
- `Webp`: WebP (`RIFF` … `WEBP`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### CoverSize

```rust
pub enum CoverSize {
    Thumb,
    Card,
    Full,
}
```

One of the sizes [`process`](#process) produces.

**Variants**

- `Thumb`: Fits in 128 × 128: list rows and search results.
- `Card`: Fits in 360 × 360: cards and detail panes.
- `Full`: Fits in [`FULL_MAX_EDGE`](#full_max_edge) on its longer edge: the book page and the
  cover embedded into an acquired file.

#### CoverSize::ALL

```rust
pub const ALL: [Self; 3] = _
```

Every size, smallest first.

#### CoverSize::max_edge

```rust
pub fn max_edge(self) -> u32
```

The longest edge this size allows, in pixels.

#### CoverSize::as_str

```rust
pub fn as_str(self) -> &'static str
```

The size's name in URLs and file names (`128`, `360`, `full`).

#### CoverSize::parse

```rust
pub fn parse(name: &str) -> Option<Self>
```

Parses a name [`Self::as_str`](#coversizeas_str) produces.

##### Arguments

* `name` - `128`, `360` or `full`, exactly.

##### Returns

The size, or `None` for any other text.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### CoverError

```rust
pub enum CoverError {
    Empty,
    TooLarge,
    UnknownFormat,
    Decode(String),
    Resize(String),
    Encode(String),
}
```

Why [`process`](#process) refused a cover.

**Variants**

- `Empty`: The input was empty.
- `TooLarge`: The input is larger than [`MAX_INPUT_BYTES`](#max_input_bytes).
- `UnknownFormat`: The signature is not JPEG, PNG or WebP.
- `Decode`: The decoder refused the image or hit a limit.
- `Resize`: Resizing failed.
- `Encode`: Encoding failed.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### sniff

```rust
pub fn sniff(bytes: &[u8]) -> Option<CoverFormat>
```

Reads the file signature of `bytes`.

#### Arguments

* `bytes` - The start of the file (the whole file is fine).

#### Returns

The format the signature names, or `None` for anything else, including
GIF, SVG, HTML and every text format.

### process

```rust
pub fn process(bytes: &[u8]) -> Result<CoverSet, CoverError>
```

Sniffs, decodes, resizes and re-encodes one untrusted cover.

#### Arguments

* `bytes` - The downloaded file, untrusted.

#### Returns

Every [`CoverSize`](#coversize) as a JPEG.

#### Errors

A [`CoverError`](#covererror) for an empty, oversized or unrecognized input, an image
over the decode limits, or a codec failure.

## Constants

### MAX_INPUT_BYTES

```rust
pub const MAX_INPUT_BYTES: usize = _;
```

Largest input accepted, in bytes. Matches the plugin ABI's
`maxCoverBytes`, the most a cover fetcher may hand the host.

### MAX_DIMENSION

```rust
pub const MAX_DIMENSION: u32 = 5000;
```

Widest or tallest image decoded, in pixels. Store covers are at most a
few thousand pixels on a side; anything larger is refused before decoding.

### MAX_DECODE_ALLOC

```rust
pub const MAX_DECODE_ALLOC: u64 = _;
```

Most memory the decoder may allocate for one image, in bytes.

### FULL_MAX_EDGE

```rust
pub const FULL_MAX_EDGE: u32 = 2400;
```

Longest edge of the [`CoverSize::Full`](#coversize) output, in pixels.

### THUMB_QUALITY

```rust
pub const THUMB_QUALITY: u8 = 85;
```

JPEG quality of the [`CoverSize::Thumb`](#coversize) and [`CoverSize::Card`](#coversize) outputs.

### FULL_QUALITY

```rust
pub const FULL_QUALITY: u8 = 90;
```

JPEG quality of the [`CoverSize::Full`](#coversize) output.
