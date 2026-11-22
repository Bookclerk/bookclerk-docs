# bookclerk_onnxruntime

Pin and fetch of the ONNX Runtime shared library.

The model plugin (`crates/bookclerk-plugins/platform/ai-embedder`) runs
its graphs with Microsoft's C++ ONNX Runtime, loaded as a shared
library that sits beside the plugin's executable. This crate names the
one release every build uses (`onnxruntime-pin.json`) and fetches it:
`cargo ensure-onnxruntime`, the staging and packaging commands and the
plugin's tests all call [`ensure_onnxruntime`](#ensure_onnxruntime).

The library is taken from the release's wheel on PyPI, which carries the
same shared library as Microsoft's release archives. The wheel's
SHA-256 is checked before it is opened, and the library's own SHA-256
after it is extracted, so a cached copy is recognised by content.

No release exists for Intel macOS; [`host_asset`](#host_asset) is `None` there and
the plugin is left out of that build.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Pin`](#pin) | struct | The pinned release: one version and one wheel per platform. |
| [`Asset`](#asset) | struct | Where one platform's library comes from and what it must be. |
| [`pin`](#pin-1) | function | Reads the embedded pin. |
| [`host_asset`](#host_asset) | function | The pinned wheel for the platform this build targets. |
| [`extract_library`](#extract_library) | function | Takes the shared library out of a wheel. |
| [`ensure_onnxruntime`](#ensure_onnxruntime) | function | Makes sure the pinned ONNX Runtime library is in `dir`, downloading it when it is missing or is another release. |
| [`ensure_in_target`](#ensure_in_target) | function | Makes sure the pinned library is in the build tree's shared cache, `<target>/onnxruntime-<version>/`, which the staging commands and the model plugin's tests all read. |
| [`LIBRARY_FILE_NAME`](#library_file_name) | constant | Name of the shared library beside the plugin executable on this platform. The model plugin looks for exactly this name. |

## Structs

### Pin

```rust
pub struct Pin {
    pub version: String,
    pub assets: BTreeMap<String, Asset>,
}
```

The pinned release: one version and one wheel per platform.

**Fields**

- `version`: ONNX Runtime release (`1.32.0`).
- `assets`: The wheel for each platform, keyed `<os>-<arch>` in the words of
  [`std::env::consts`](https://doc.rust-lang.org/stable/std/env/consts/index.html) (`linux-x86_64`, `macos-aarch64`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`

### Asset

```rust
pub struct Asset {
    pub wheel_url: String,
    pub wheel_sha256_hex: String,
    pub member: String,
    pub library_bytes: u64,
    pub library_sha256_hex: String,
}
```

Where one platform's library comes from and what it must be.

**Fields**

- `wheel_url`: HTTPS address of the wheel.
- `wheel_sha256_hex`: SHA-256 of the wheel, lowercase hexadecimal.
- `member`: Path of the shared library inside the wheel.
- `library_bytes`: Length of the shared library in bytes.
- `library_sha256_hex`: SHA-256 of the shared library, lowercase hexadecimal.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`

## Functions

### pin

```rust
pub fn pin() -> Result<Pin>
```

Reads the embedded pin.

#### Errors

When `onnxruntime-pin.json` is not the document [`Pin`](#pin) describes; a
test keeps that from reaching a build.

### host_asset

```rust
pub fn host_asset() -> Result<Option<Asset>>
```

The pinned wheel for the platform this build targets.

#### Returns

`None` when ONNX Runtime publishes no library for the platform.

#### Errors

When the embedded pin cannot be read.

### extract_library

```rust
pub fn extract_library(wheel: &[u8], asset: &Asset) -> Result<Vec<u8>>
```

Takes the shared library out of a wheel.

#### Arguments

* `wheel` - The wheel's bytes, already checked against the pin.
* `asset` - The pin entry naming the member and what it must be.

#### Errors

When the wheel is not a zip archive, the member is missing, or the
member's length or SHA-256 differs from the pin.

### ensure_onnxruntime

```rust
pub fn ensure_onnxruntime(dir: &Path) -> Result<Option<PathBuf>>
```

Makes sure the pinned ONNX Runtime library is in `dir`, downloading it
when it is missing or is another release.

#### Arguments

* `dir` - Directory to hold [`LIBRARY_FILE_NAME`](#library_file_name); created when missing.

#### Returns

The library's path, or `None` on a platform ONNX Runtime has no release
for.

#### Errors

When the download fails, a digest differs from the pin, or the file
cannot be written. A symbolic link in the library's place is refused.

### ensure_in_target

```rust
pub fn ensure_in_target(target: &Path) -> Result<Option<PathBuf>>
```

Makes sure the pinned library is in the build tree's shared cache,
`<target>/onnxruntime-<version>/`, which the staging commands and the
model plugin's tests all read.

#### Arguments

* `target` - Cargo's target directory.

#### Returns

The library's path, or `None` on a platform ONNX Runtime has no release
for.

#### Errors

As [`ensure_onnxruntime`](#ensure_onnxruntime).

## Constants

### LIBRARY_FILE_NAME

```rust
pub const LIBRARY_FILE_NAME: &str = _;
```

Name of the shared library beside the plugin executable on this
platform. The model plugin looks for exactly this name.
