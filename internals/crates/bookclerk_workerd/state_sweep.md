# bookclerk_workerd::state_sweep

Removes isolate state directories that a killed launcher left behind.

Each isolate session gets a fresh state directory under the guest's
`TMPDIR` (see [`crate::config::workerd_state_dir`](config.md#workerd_state_dir)), which the launcher
removes when it returns. When the host stops a session by killing the
launcher, as it does when a plugin goes to sleep, that cleanup never
runs, and every start of a sleeping plugin would leave about half a
megabyte in the plugin's `tmp` until its disk budget refuses to start it.

So each launcher holds an exclusive lock on [`OWNER_LOCK_FILE`](#owner_lock_file) inside its
state directory for as long as it lives; the operating system drops the
lock when the process dies, however it dies. Before allocating its own
directory, a launcher removes every sibling directory of the same plugin
whose lock it can take, and every one older than [`UNLOCKED_GRACE`](#unlocked_grace) that
never got a lock file.

| Item | Kind | Summary |
| --- | --- | --- |
| [`StateDirOwner`](#statedirowner) | struct | Proof that this process owns a state directory; dropping it (or the process dying) releases the lock. |
| [`claim`](#claim) | function | Takes the owner lock on `state_dir`. |
| [`sweep_abandoned`](#sweep_abandoned) | function | Removes the abandoned state directories of the plugin at `plugin_root`. |
| [`OWNER_LOCK_FILE`](#owner_lock_file) | constant | The lock file a live launcher holds inside its state directory. |
| [`UNLOCKED_GRACE`](#unlocked_grace) | constant | How old a state directory without a lock file must be before it counts as abandoned: well past the moment between creating a directory and locking it. |

## Structs

### StateDirOwner

```rust
pub struct StateDirOwner {
    /* private fields */
}
```

Proof that this process owns a state directory; dropping it (or the
process dying) releases the lock.

Implements: `Debug`

## Functions

### claim

```rust
pub fn claim(state_dir: &Path) -> Result<StateDirOwner>
```

Takes the owner lock on `state_dir`.

#### Arguments

* `state_dir` - The session's freshly created state directory.

#### Errors

Returns an error when the lock file cannot be created or locked.

### sweep_abandoned

```rust
pub fn sweep_abandoned(plugin_root: &Path) -> Vec<PathBuf>
```

Removes the abandoned state directories of the plugin at `plugin_root`.

Only directories named for this plugin root are considered, links are
never followed, and a directory a live launcher holds is left alone.
Failures are skipped: a directory that cannot be removed now is tried
again by the next start.

#### Arguments

* `plugin_root` - The plugin install directory the state is named for.

#### Returns

The directories removed.

## Constants

### OWNER_LOCK_FILE

```rust
pub const OWNER_LOCK_FILE: &str = ".owner.lock";
```

The lock file a live launcher holds inside its state directory.

### UNLOCKED_GRACE

```rust
pub const UNLOCKED_GRACE: Duration = _;
```

How old a state directory without a lock file must be before it counts
as abandoned: well past the moment between creating a directory and
locking it.
