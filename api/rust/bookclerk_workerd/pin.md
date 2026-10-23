**bookclerk_workerd > pin**

# Module: pin

## Contents

**Structs**

- [`WorkerdAsset`](#workerdasset) - GitHub release asset name + sha256 (hex) of the `.gz` payload.

**Functions**

- [`asset_for_target`](#asset_for_target) - Look up the pinned download asset for a `std::env::consts` OS/arch pair.
- [`binary_name`](#binary_name) - On-disk binary name after extracting the gzipped release asset.
- [`download_url`](#download_url) - Build the GitHub release download URL for a pinned artifact name.
- [`host_asset`](#host_asset) - Resolve the release asset for the compiling host OS/arch.

**Constants**

- [`BUNDLED_WORKERD_COMPAT_DATE`](#bundled_workerd_compat_date) - Newest compatibility date this Bookclerk build claims to understand.
- [`WORKERD_RELEASE_TAG`](#workerd_release_tag) - GitHub release tag (includes leading `v`).
- [`WORKERD_VERSION_STAMP`](#workerd_version_stamp) - Filename written beside the binary so ensure can skip re-downloads.

---

## bookclerk_workerd::pin::BUNDLED_WORKERD_COMPAT_DATE

*Constant*: `&str`

Newest compatibility date this Bookclerk build claims to understand.



## bookclerk_workerd::pin::WORKERD_RELEASE_TAG

*Constant*: `&str`

GitHub release tag (includes leading `v`).



## bookclerk_workerd::pin::WORKERD_VERSION_STAMP

*Constant*: `&str`

Filename written beside the binary so ensure can skip re-downloads.



## bookclerk_workerd::pin::WorkerdAsset

*Struct*

GitHub release asset name + sha256 (hex) of the `.gz` payload.

**Fields:**
- `artifact: &'static str` - Cloudflare release asset filename (for example `workerd-linux-64.gz`).
- `sha256_hex: &'static str` - Expected SHA-256 hex digest of the gzipped download bytes.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &WorkerdAsset) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> WorkerdAsset`



## bookclerk_workerd::pin::asset_for_target

*Function*

Look up the pinned download asset for a `std::env::consts` OS/arch pair.

# Arguments

* `os` - Host OS string (`linux`, `macos`, or `windows`).
* `arch` - Host CPU string (`x86_64` or `aarch64`).

# Returns

[`WorkerdAsset`](pin.md#bookclerk_workerdpinworkerdasset) when the pin JSON lists that target; otherwise `None`.

```rust
fn asset_for_target(os: &str, arch: &str) -> Option<WorkerdAsset>
```



## bookclerk_workerd::pin::binary_name

*Function*

On-disk binary name after extracting the gzipped release asset.

```rust
fn binary_name() -> &'static str
```



## bookclerk_workerd::pin::download_url

*Function*

Build the GitHub release download URL for a pinned artifact name.

# Arguments

* `artifact` - Release asset filename from `WorkerdAsset::artifact`.

# Returns

Absolute `https://github.com/cloudflare/workerd/releases/download/...` URL.

```rust
fn download_url(artifact: &str) -> String
```



## bookclerk_workerd::pin::host_asset

*Function*

Resolve the release asset for the compiling host OS/arch.

```rust
fn host_asset() -> Option<WorkerdAsset>
```



