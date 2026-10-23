**bookclerk_plugin_source_audible > widevine**

# Module: widevine

## Contents

**Structs**

- [`WidevineCdm`](#widevinecdm) - Loaded Widevine CDM client.
- [`WidevineDownload`](#widevinedownload) - Result of a Widevine (or Mpeg-fallback) download into `work_dir`.

**Functions**

- [`effective_cdm_provider`](#effective_cdm_provider) - Resolve provider URL: `None` config → built-in Bookclerk AudibleCdm; empty/`off` → disabled.
- [`ensure_widevine_cdm`](#ensure_widevine_cdm) - Load an account's Widevine CDM, or auto-provision one via the classic
- [`load_widevine_cdm`](#load_widevine_cdm) - Resolve and load a bring-your-own `.wvd` device blob from local files.

**Constants**

- [`DEFAULT_WIDEVINE_CDM_PROVIDER`](#default_widevine_cdm_provider) - Classic Libation AudibleCdm endpoint — provisions a unique L3 `.wvd` per account.

---

## bookclerk_plugin_source_audible::widevine::DEFAULT_WIDEVINE_CDM_PROVIDER

*Constant*: `&str`

Classic Libation AudibleCdm endpoint — provisions a unique L3 `.wvd` per account.



## bookclerk_plugin_source_audible::widevine::WidevineCdm

*Struct*

Loaded Widevine CDM client.

**Fields:**
- `security_level: u8` - Security level.



## bookclerk_plugin_source_audible::widevine::WidevineDownload

*Struct*

Result of a Widevine (or Mpeg-fallback) download into `work_dir`.

**Fields:**
- `path: std::path::PathBuf` - Filesystem path for this value.
- `drm_type: String` - DRM type.
- `content_format: Option<String>` - Content format.
- `content_size: Option<u64>` - Content size.
- `kid: Option<String>` - CENC key id (hex); absent for plain Mpeg.
- `key: Option<String>` - CENC content key (hex).
- `needs_decrypt: bool` - Needs decrypt.
- `pdf_url: Option<String>` - Pdf URL.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> WidevineDownload`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::widevine::effective_cdm_provider

*Function*

Resolve provider URL: `None` config → built-in Bookclerk AudibleCdm; empty/`off` → disabled.

```rust
fn effective_cdm_provider(configured: Option<&str>) -> Option<&str>
```



## bookclerk_plugin_source_audible::widevine::ensure_widevine_cdm

*Function*

Load an account's Widevine CDM, or auto-provision one via the classic
Libation AudibleCdm provider. CDMs are stored in the `encrypted_secrets` DB
table only — nothing is written under `Accounts/`.

Resolution order:
1. `encrypted_secrets` (DB) for `account_stem`
2. BYO `.wvd` file (`output.widevine_cdm` / `{files_dir}/widevine.wvd`),
   imported into the DB when possible
3. Provision a fresh L3 CDM from the provider (auth material from the DB) and
   persist it to the DB

Widevine drmlicense requires an **Android**-registered account
(`bookclerk auth login` always registers as Android). Spatial/Atmos (L1) remains unavailable.

# Errors

Returns an error when the operation fails.

```rust
fn ensure_widevine_cdm(files_dir: &std::path::Path, configured: Option<&std::path::Path>, account_stem: Option<&str>, provider_url: Option<&str>, scope: Option<&bookclerk_library::SourceScope>) -> crate::error::Result<(WidevineCdm, std::path::PathBuf)>
```



## bookclerk_plugin_source_audible::widevine::load_widevine_cdm

*Function*

Resolve and load a bring-your-own `.wvd` device blob from local files.

This is a one-shot import path only (no `Accounts/`). Search order:
1. Explicit `output.widevine_cdm` path (absolute, or relative to `files_dir`)
2. `{files_dir}/widevine.wvd`

# Errors

Returns an error when the operation fails.

```rust
fn load_widevine_cdm(files_dir: &std::path::Path, configured: Option<&std::path::Path>) -> crate::error::Result<(WidevineCdm, std::path::PathBuf)>
```



