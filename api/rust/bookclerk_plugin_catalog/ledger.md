**bookclerk_plugin_catalog > ledger**

# Module: ledger

## Contents

**Structs**

- [`InstallLedger`](#installledger) - Host-owned map of PluginKey → expected digests for verified installs.
- [`InstallLedgerEntry`](#installledgerentry) - Host-owned record of one verified installed artifact.

**Functions**

- [`record_install`](#record_install) - Records (or updates) a host-verified install in the files-dir ledger.
- [`restore_ledger_entry`](#restore_ledger_entry) - Restores `previous` for `plugin_key`, or deletes the row when `previous` is `None`.

**Constants**

- [`INSTALL_LEDGER_FILE`](#install_ledger_file) - Filename under `$BOOKCLERK_FILES_DIR` for the host install ledger.
- [`INSTALL_LEDGER_SCHEMA_VERSION`](#install_ledger_schema_version) - Schema version for [`InstallLedger`](ledger.md#bookclerk_plugin_catalogledgerinstallledger).

---

## bookclerk_plugin_catalog::ledger::INSTALL_LEDGER_FILE

*Constant*: `&str`

Filename under `$BOOKCLERK_FILES_DIR` for the host install ledger.



## bookclerk_plugin_catalog::ledger::INSTALL_LEDGER_SCHEMA_VERSION

*Constant*: `u32`

Schema version for [`InstallLedger`](ledger.md#bookclerk_plugin_catalogledgerinstallledger).



## bookclerk_plugin_catalog::ledger::InstallLedger

*Struct*

Host-owned map of PluginKey → expected digests for verified installs.

**Fields:**
- `schema_version: u32` - DTO schema version.
- `artifacts: Vec<InstallLedgerEntry>` - One row per PluginKey (last write wins).

**Methods:**

- `fn path(files_dir: &Path) -> PathBuf` - Absolute path to `install-ledger.json` under `files_dir`.
- `fn load(files_dir: &Path) -> Result<Self>` - Loads the ledger, or an empty ledger when the file is missing.
- `fn get(self: &Self, plugin_key: &PluginKey) -> Option<&InstallLedgerEntry>` - Lookup by canonical PluginKey.
- `fn upsert(self: & mut Self, entry: InstallLedgerEntry)` - Inserts or replaces the row for `entry.plugin_key`.
- `fn remove(self: & mut Self, plugin_key: &PluginKey) -> bool` - Removes the row for `plugin_key` if present.
- `fn store(self: &Self, files_dir: &Path) -> Result<()>` - Atomically writes the ledger under `files_dir`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> InstallLedger`
- **PartialEq**
  - `fn eq(self: &Self, other: &InstallLedger) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_catalog::ledger::InstallLedgerEntry

*Struct*

Host-owned record of one verified installed artifact.

**Fields:**
- `plugin_key: String` - Canonical [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey).
- `package_name: String` - Package coordinate name recorded at install/stage time.
- `manifest_id: String` - Manifest alias (`sqlite`, `local`, …).
- `manifest_sha256: String` - SHA-256 of the installed `plugin.toml` bytes.
- `payload_root_sha256: String` - Deterministic payload root over immutable packaged files.
- `provenance: crate::identity::PluginProvenance` - Host-evaluated provenance at the time the ledger row was written.
- `recorded_at: chrono::DateTime<chrono::Utc>` - RFC 3339 time when the host recorded this artifact.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &InstallLedgerEntry) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> InstallLedgerEntry`



## bookclerk_plugin_catalog::ledger::record_install

*Function*

Records (or updates) a host-verified install in the files-dir ledger.

# Errors

Returns when the ledger cannot be loaded or stored, or `plugin_key` is empty.

```rust
fn record_install(files_dir: &std::path::Path, plugin_key: &crate::identity::PluginKey, package_name: &str, manifest_id: &str, manifest_sha256: &str, payload_root_sha256: &str, provenance: crate::identity::PluginProvenance) -> crate::error::Result<()>
```



## bookclerk_plugin_catalog::ledger::restore_ledger_entry

*Function*

Restores `previous` for `plugin_key`, or deletes the row when `previous` is `None`.

# Errors

Returns when the ledger cannot be loaded or stored.

```rust
fn restore_ledger_entry(files_dir: &std::path::Path, plugin_key: &crate::identity::PluginKey, previous: Option<InstallLedgerEntry>) -> crate::error::Result<()>
```



