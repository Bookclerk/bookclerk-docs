**bookclerk_plugin_catalog > evaluate**

# Module: evaluate

## Contents

**Functions**

- [`evaluate_install`](#evaluate_install) - Evaluate identity + provenance for an install directory.
- [`evaluate_install_in`](#evaluate_install_in) - Evaluate identity + provenance, consulting the host install ledger under
- [`stamp_platform_receipt`](#stamp_platform_receipt) - Build a host-owned platform-bundled receipt **and** ledger row.

---

## bookclerk_plugin_catalog::evaluate::evaluate_install

*Function*

Evaluate identity + provenance for an install directory.

Equivalent to [`evaluate_install_in`](evaluate.md#bookclerk_plugin_catalogevaluateevaluate_install_in) with no host files directory. Without
a files-dir ledger, platform trust cannot be established.

# Errors

Returns an error when the payload cannot be hashed, `plugin.toml` is
missing, a receipt is malformed, or a present receipt is not recorded in
the host ledger.

```rust
fn evaluate_install(root: &std::path::Path, manifest: &bookclerk_plugin_manifest::PluginManifest) -> crate::error::Result<crate::identity::PluginInstallIdentity>
```



## bookclerk_plugin_catalog::evaluate::evaluate_install_in

*Function*

Evaluate identity + provenance, consulting the host install ledger under
`files_dir` when provided.

Receipt fields are **not** a trust anchor. `PluginProvenance::PlatformBundled`
requires a matching [`InstallLedger`](ledger.md#bookclerk_plugin_catalogledgerinstallledger) row written by host tooling, a known
platform [`PluginKey`](identity.md#bookclerk_plugin_catalogidentitypluginkey), and exact current payload digests.

A genuinely absent receipt is `PluginProvenance::LocalDevelopment`. Any
other receipt/hash failure fails closed.

# Errors

Returns an error when hashing fails, a receipt is unreadable or malformed,
the stored PluginKey is invalid, or a present receipt has no matching
host ledger entry.

```rust
fn evaluate_install_in(root: &std::path::Path, manifest: &bookclerk_plugin_manifest::PluginManifest, files_dir: Option<&std::path::Path>) -> crate::error::Result<crate::identity::PluginInstallIdentity>
```



## bookclerk_plugin_catalog::evaluate::stamp_platform_receipt

*Function*

Build a host-owned platform-bundled receipt **and** ledger row.

Only [`crate::identity::PLATFORM_ARTIFACTS`](identity.md#bookclerk_plugin_catalogidentityplatform_artifacts) may be stamped this way.
`files_dir` is the Bookclerk files directory that owns `install-ledger.json`.

# Errors

Returns an error when `package_name`/`manifest.id` are not a known
platform pair, hashing fails, or the ledger cannot be written.

```rust
fn stamp_platform_receipt(root: &std::path::Path, files_dir: &std::path::Path, package_name: &str, manifest: &bookclerk_plugin_manifest::PluginManifest, version: &str) -> crate::error::Result<crate::receipt::InstallReceipt>
```



