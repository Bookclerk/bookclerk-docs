**bookclerk_plugin_sdk > fetch_dir**

# Module: fetch_dir

## Contents

**Structs**

- [`FetchWorkDir`](#fetchworkdir) - An open fetch work directory received from the host for one `fetchTitle` call.
- [`UploadFile`](#uploadfile) - An open upload file received from the host for one `putFile` call.

**Functions**

- [`fetch_work_dir`](#fetch_work_dir) - Resolves the directory a `fetchTitle` call should write into.
- [`upload_file_path`](#upload_file_path) - Resolves the local file path for a `putFile` call.

---

## bookclerk_plugin_sdk::fetch_dir::FetchWorkDir

*Struct*

An open fetch work directory received from the host for one `fetchTitle` call.

Holds the descriptor open for its lifetime so `/proc/self/fd/N` (or
`/dev/fd/N`) stays valid for the duration of the call. When the side channel
is not armed, falls back to the `cache_dir` field on
[`FetchTitleParams`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedfetchtitleparams) from the RPC params (absolute path string the host
already prepared inside the guest jail).

**Methods:**

- `fn open(params: &FetchTitleParams) -> Result<Self>` - Resolves the directory a `fetchTitle` call should write downloaded audio into.
- `fn path(self: &Self) -> &Path` - Absolute filesystem path of the fetch work directory.

**Trait Implementations:**

- **Deref**
  - `fn deref(self: &Self) -> &<Self as >::Target`



## bookclerk_plugin_sdk::fetch_dir::UploadFile

*Struct*

An open upload file received from the host for one `putFile` call.

Same FD-lifetime rules as [`FetchWorkDir`]: keep the value alive while
reading the path so the `/proc/self/fd/N` symlink remains valid.

**Methods:**

- `fn open(local_path: Option<&str>) -> Result<Self>` - Resolves the local file path for a `putFile` upload.
- `fn path(self: &Self) -> &Path` - Absolute filesystem path of the file the host wants uploaded.

**Trait Implementations:**

- **Deref**
  - `fn deref(self: &Self) -> &<Self as >::Target`



## bookclerk_plugin_sdk::fetch_dir::fetch_work_dir

*Function*

Resolves the directory a `fetchTitle` call should write into.

Thin wrapper around `FetchWorkDir::open`.

# Arguments

* `params` - Host `fetchTitle` params (see `FetchWorkDir::open`).

# Returns

Open work directory for this RPC.

# Errors

Propagates `FetchWorkDir::open` failures.

```rust
fn fetch_work_dir(params: &crate::protocol::FetchTitleParams) -> crate::error::Result<FetchWorkDir>
```



## bookclerk_plugin_sdk::fetch_dir::upload_file_path

*Function*

Resolves the local file path for a `putFile` call.

Thin wrapper around `UploadFile::open`.

# Arguments

* `local_path` - Optional path from host params (see `UploadFile::open`).

# Returns

Open upload file handle for this RPC.

# Errors

Propagates `UploadFile::open` failures.

```rust
fn upload_file_path(local_path: Option<&str>) -> crate::error::Result<UploadFile>
```



