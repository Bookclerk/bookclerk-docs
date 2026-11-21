# bookclerk_library::master_key_recovery

What an install's operator did about a recovery copy of its key.

A key kept in the OS keychain never sits in a file a backup picks up, so
the desktop window asks its operator to save a recovery copy
([`crate::export_master_key`](master_key.md#export_master_key)) and reminds them under Settings until they
do. The answer lives in `master-key-recovery.json` beside `master.key`:
which key it was given for (the key's SHA-256 fingerprint), whether a copy
was saved or the step skipped, and when. A record for another key (one
restored or replaced since) counts as no record, so a new key asks again.

The file holds no secret: the fingerprint is the same one a `BCK3`
keychain pointer carries.

| Item | Kind | Summary |
| --- | --- | --- |
| [`RecoveryCopyChoice`](#recoverycopychoice) | enum | What the operator chose about a recovery copy. |
| [`RecoveryCopyState`](#recoverycopystate) | enum | The recorded answer for the key in use, if any. |
| [`recovery_record_path`](#recovery_record_path) | function | Path of the record in `files_dir`. |
| [`recovery_copy_state`](#recovery_copy_state) | function | Reads what the operator recorded about a recovery copy of `key`. |
| [`record_recovery_copy`](#record_recovery_copy) | function | Records that the operator saved a recovery copy of `key`, or skipped it. |
| [`RECOVERY_RECORD_FILE_NAME`](#recovery_record_file_name) | constant | File name of the record, beside `master.key` in the files directory. |

## Enums

### RecoveryCopyChoice

```rust
pub enum RecoveryCopyChoice {
    Saved,
    Skipped,
}
```

What the operator chose about a recovery copy.

**Variants**

- `Saved`: The operator saved a recovery copy of this key.
- `Skipped`: The operator skipped saving one, after a warning.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RecoveryCopyState

```rust
pub enum RecoveryCopyState {
    Unrecorded,
    Recorded { choice: RecoveryCopyChoice, recorded_at: i64 },
}
```

The recorded answer for the key in use, if any.

**Variants**

- `Unrecorded`: Nothing is recorded for this key: no copy saved, step not skipped.
- `Recorded`: The operator answered for this key.
  - `choice`: What they chose.
  - `recorded_at`: When, in whole seconds since the Unix epoch (UTC).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### recovery_record_path

```rust
pub fn recovery_record_path(files_dir: &Path) -> PathBuf
```

Path of the record in `files_dir`.

#### Arguments

* `files_dir` - the files directory holding `master.key`.

#### Returns

`<files_dir>/master-key-recovery.json`.

### recovery_copy_state

```rust
pub fn recovery_copy_state(files_dir: &Path, key: &MasterKey) -> RecoveryCopyState
```

Reads what the operator recorded about a recovery copy of `key`.

A missing or unreadable record, or one written for a different key,
reads as [`RecoveryCopyState::Unrecorded`]: the safe answer is to ask
again.

#### Arguments

* `files_dir` - the files directory holding `master.key`.
* `key` - the key in use.

#### Returns

The recorded answer for `key`, or [`RecoveryCopyState::Unrecorded`](#recoverycopystate).

### record_recovery_copy

```rust
pub fn record_recovery_copy(
    files_dir: &Path,
    key: &MasterKey,
    choice: RecoveryCopyChoice,
) -> Result<RecoveryCopyState>
```

Records that the operator saved a recovery copy of `key`, or skipped it.

The record replaces any earlier one atomically (a temp file renamed over
it), so a crash leaves the old answer or the new one.

#### Arguments

* `files_dir` - the files directory holding `master.key`.
* `key` - the key in use, whose fingerprint the record names.
* `choice` - what the operator chose.

#### Returns

The state now recorded.

#### Errors

Returns an error when the record can't be written.

## Constants

### RECOVERY_RECORD_FILE_NAME

```rust
pub const RECOVERY_RECORD_FILE_NAME: &str = "master-key-recovery.json";
```

File name of the record, beside `master.key` in the files directory.
