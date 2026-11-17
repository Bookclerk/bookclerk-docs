# bookclerk_workerd::python_snapshot

Python start-up snapshots for workerd isolates.

A Python isolate spends seconds loading Pyodide and importing the plugin
before it can answer. workerd can save the isolate's memory after those
imports (a "dedicated" snapshot) and start later isolates from it instead.
The host keeps one snapshot per installed plugin version, read-only to the
guest, and hands the launcher two paths through the environment:

- [`LOAD_ENV`]: the stored snapshot to start from, when the host has one.
- [`SAVE_ENV`]: an empty staging directory the jail may write. When there
  is nothing to load, or the stored snapshot fails to start, the launcher
  starts workerd saving a fresh snapshot there, and renames it to
  [`READY_FILE`](#ready_file) once the isolate answered. The host promotes only a
  ready file into its store.

Neither path is a secret, and a missing or broken snapshot only costs the
slow start: the launcher always falls back to starting without one.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SnapshotPlan`](#snapshotplan) | struct | What the host asked the launcher to do with snapshots for this start. |
| [`SnapshotStart`](#snapshotstart) | enum | One way to start workerd for a Python isolate. |
| [`mark_ready`](#mark_ready) | function | Marks the snapshot saved into `staging` as complete. |
| [`mark_loaded`](#mark_loaded) | function | Leaves the `LOADED_FILE` marker in `staging`: this start loaded the stored snapshot and saves nothing. |
| [`LOAD_ENV`](#load_env) | constant | Environment variable naming the stored snapshot file to start from. |
| [`SAVE_ENV`](#save_env) | constant | Environment variable naming the staging directory a fresh snapshot is saved into. |
| [`SAVED_FILE`](#saved_file) | constant | The file name workerd writes a saved snapshot under, inside the snapshot directory. |
| [`READY_FILE`](#ready_file) | constant | The file name a saved snapshot is renamed to once the isolate it came from answered, so the host never promotes a half-written file. |
| [`LOADED_FILE`](#loaded_file) | constant | The marker the launcher leaves in the staging directory when the isolate started from the stored snapshot, so the host stops waiting for a save. |
| [`MAX_SNAPSHOT_BYTES`](#max_snapshot_bytes) | constant | The largest snapshot the host promotes, in bytes. A dedicated snapshot of a plugin with the Bookclerk SDK is about 45 MB; anything far past that is refused rather than stored. |

## Structs

### SnapshotPlan

```rust
pub struct SnapshotPlan {
    pub load: Option<PathBuf>,
    pub save: Option<PathBuf>,
}
```

What the host asked the launcher to do with snapshots for this start.

**Fields**

- `load`: The stored snapshot to start from, when the host has one.
- `save`: The staging directory a fresh snapshot may be saved into.

#### SnapshotPlan::from_env

```rust
pub fn from_env() -> Self
```

Reads the plan from [`LOAD_ENV`](#load_env) and [`SAVE_ENV`](#save_env).

A value that is empty or not an absolute path is ignored, so a bad
value only costs the fast start.

#### SnapshotPlan::starts

```rust
pub fn starts(&self) -> Vec<SnapshotStart>
```

The starts to try, in order: loading the stored snapshot when there
is one, then saving a fresh one when a staging directory was given,
then a plain start.

##### Returns

At least one start; the last is the fallback when every earlier one
fails to become ready.

#### SnapshotPlan::start_in_order

```rust
pub async fn start_in_order<T, F, Fut>(&self, start: F) -> Result<T>
where
    F: FnMut(SnapshotStart) -> Fut,
    Fut: Future<Output = Result<T>>,
```

Runs `start` for each of [`Self::starts`](#snapshotplanstarts) until one becomes ready,
then records the outcome in the staging directory.

A start that saved a snapshot marks it ready for the host to
promote; a start that loaded the stored one leaves the
[`LOADED_FILE`](#loaded_file) marker so the host stops waiting. A failed start is
logged and the next one tried.

##### Arguments

* `start` - Starts workerd one way and waits for it to answer.

##### Returns

What the first ready start returned.

##### Errors

Returns the last start's error when every start failed.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Enums

### SnapshotStart

```rust
pub enum SnapshotStart {
    Load(PathBuf),
    Save(PathBuf),
    Plain,
}
```

One way to start workerd for a Python isolate.

**Variants**

- `Load`: Start from the stored snapshot at this path.
- `Save`: Start normally and save a snapshot into this staging directory.
- `Plain`: Start normally without snapshots.

#### SnapshotStart::serve_args

```rust
pub fn serve_args(&self) -> Vec<OsString>
```

The `workerd serve` arguments for this start.

##### Returns

No arguments for [`SnapshotStart::Plain`](#snapshotstart), or for a load path without
a parent directory or file name.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### mark_ready

```rust
pub fn mark_ready(staging: &Path) -> Result<()>
```

Marks the snapshot saved into `staging` as complete.

Called once the isolate that saved it answered, so the file is whole.

#### Arguments

* `staging` - The directory workerd saved the snapshot into.

#### Errors

Returns the I/O error when there is no saved snapshot or it cannot be
renamed.

### mark_loaded

```rust
pub fn mark_loaded(staging: &Path) -> Result<()>
```

Leaves the [`LOADED_FILE`](#loaded_file) marker in `staging`: this start loaded the
stored snapshot and saves nothing.

#### Arguments

* `staging` - The staging directory the host gave this start.

#### Errors

Returns the I/O error when the marker cannot be written.

## Constants

### LOAD_ENV

```rust
pub const LOAD_ENV: &str = "BOOKCLERK_PYTHON_SNAPSHOT_LOAD";
```

Environment variable naming the stored snapshot file to start from.

### SAVE_ENV

```rust
pub const SAVE_ENV: &str = "BOOKCLERK_PYTHON_SNAPSHOT_SAVE";
```

Environment variable naming the staging directory a fresh snapshot is
saved into.

### SAVED_FILE

```rust
pub const SAVED_FILE: &str = "snapshot.bin";
```

The file name workerd writes a saved snapshot under, inside the snapshot
directory.

### READY_FILE

```rust
pub const READY_FILE: &str = "snapshot.ready";
```

The file name a saved snapshot is renamed to once the isolate it came
from answered, so the host never promotes a half-written file.

### LOADED_FILE

```rust
pub const LOADED_FILE: &str = "snapshot.loaded";
```

The marker the launcher leaves in the staging directory when the isolate
started from the stored snapshot, so the host stops waiting for a save.

### MAX_SNAPSHOT_BYTES

```rust
pub const MAX_SNAPSHOT_BYTES: u64 = _;
```

The largest snapshot the host promotes, in bytes. A dedicated snapshot of
a plugin with the Bookclerk SDK is about 45 MB; anything far past that is
refused rather than stored.
