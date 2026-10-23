**bookclerk_plugin_catalog > mutation_lock**

# Module: mutation_lock

## Contents

**Structs**

- [`PluginMutationLock`](#pluginmutationlock) - RAII exclusive lock covering one host plugin namespace (`$FILES_DIR`).

**Functions**

- [`acquire_plugins_lock`](#acquire_plugins_lock) - Acquires the host mutation lock for `plugins_root`'s parent files dir.

**Constants**

- [`PLUGIN_MUTATION_LOCK_FILE`](#plugin_mutation_lock_file) - Filename under `$BOOKCLERK_FILES_DIR` for the host plugin mutation lock.

---

## bookclerk_plugin_catalog::mutation_lock::PLUGIN_MUTATION_LOCK_FILE

*Constant*: `&str`

Filename under `$BOOKCLERK_FILES_DIR` for the host plugin mutation lock.



## bookclerk_plugin_catalog::mutation_lock::PluginMutationLock

*Struct*

RAII exclusive lock covering one host plugin namespace (`$FILES_DIR`).

Hold this guard for the full local mutation transaction: configured alias
preflight, ledger read, tree/state hold, ledger write, health, and
commit or rollback. Drop (or process exit) releases the OS lock.

Binding the guard is required: `PluginMutationLock::acquire(dir)?;` without
an owning binding drops the lock immediately.

**Methods:**

- `fn path(files_dir: &Path) -> PathBuf` - Absolute path to `.plugin-mutation.lock` under `files_dir`.
- `fn acquire(files_dir: &Path) -> Result<Self>` - Acquires an exclusive inter-process lock for `files_dir`.
- `fn files_dir(self: &Self) -> &Path` - Host files directory this lock owns.
- `fn covers_plugins_root(self: &Self, plugins_root: &Path) -> bool` - True when `plugins_root` is the `plugins/` directory under this lock.
- `fn require_plugins_root(self: &Self, plugins_root: &Path) -> Result<()>` - Rejects a plugins root that is not this lock's host namespace.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_catalog::mutation_lock::acquire_plugins_lock

*Function*

Acquires the host mutation lock for `plugins_root`'s parent files dir.

# Errors

Returns when the lock cannot be taken. `None` when `plugins_root` has no
parent (no host files dir).

```rust
fn acquire_plugins_lock(plugins_root: &std::path::Path) -> crate::error::Result<Option<PluginMutationLock>>
```



