**bookclerk_config > settings**

# Module: settings

## Contents

**Structs**

- [`AuthConfig`](#authconfig) - Auth encryption settings (`[auth]` section).
- [`Config`](#config) - Top-level Bookclerk configuration.
- [`DaemonAuthConfig`](#daemonauthconfig) - Operator token / session settings for the daemon HTTP API.
- [`DaemonConfig`](#daemonconfig) - Daemon / HTTP control plane settings.
- [`DiagnosticsConfig`](#diagnosticsconfig) - Opt-in sharing of recent **redacted** logs on crash or error bursts.
- [`DiscoveryConfig`](#discoveryconfig) - Discovery / recommendations / embeddings settings.
- [`LibraryConfig`](#libraryconfig) - Library / scan related settings.
- [`NodeApiConfig`](#nodeapiconfig) - `[daemon.node_api]`: transport policy for host nodes.
- [`ProcessEnvGuard`](#processenvguard) - Process-wide lock around `BOOKCLERK_*` environment reads and test writes.

**Enums**

- [`AudioQuality`](#audioquality) - Audible download bitrate tier (`high` / `normal` in TOML).
- [`NodeApiMode`](#nodeapimode) - How host nodes may reach the node API (`[daemon.node_api] mode`).

**Functions**

- [`read_node_api_spki_pin`](#read_node_api_spki_pin) - The SPKI pin of the daemon's self-signed node API certificate, once the

**Constants**

- [`LETS_ENCRYPT_DIRECTORY`](#lets_encrypt_directory) - Let's Encrypt's production ACME directory.
- [`NODE_API_ACME_DIR`](#node_api_acme_dir) - Directory under `$BOOKCLERK_FILES_DIR` holding the ACME account and
- [`NODE_API_SELF_SIGNED_DIR`](#node_api_self_signed_dir) - Directory under `$BOOKCLERK_FILES_DIR` holding the self-signed node API
- [`NODE_API_SPKI_PIN_FILE`](#node_api_spki_pin_file) - File in [`NODE_API_SELF_SIGNED_DIR`](settings.md#bookclerk_configsettingsnode_api_self_signed_dir) holding the SPKI pin
- [`TRAY_HANDOFF_TTL_SECS_DEFAULT`](#tray_handoff_ttl_secs_default) - Default tray/CLI loopback sign-in link lifetime (seconds).
- [`TRAY_HANDOFF_TTL_SECS_MAX`](#tray_handoff_ttl_secs_max) - Maximum allowed `DaemonAuthConfig::tray_handoff_ttl_secs`.
- [`TRAY_HANDOFF_TTL_SECS_MIN`](#tray_handoff_ttl_secs_min) - Minimum allowed `DaemonAuthConfig::tray_handoff_ttl_secs`.

---

## bookclerk_config::settings::AudioQuality

*Enum*

Audible download bitrate tier (`high` / `normal` in TOML).

**Variants:**
- `High` - Highest offered bitrate for the title (default).
- `Normal` - Standard / lower bitrate when the store offers a choice.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &AudioQuality) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> AudioQuality`
- **Default**
  - `fn default() -> AudioQuality`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::settings::AuthConfig

*Struct*

Auth encryption settings (`[auth]` section).

Bookclerk always encrypts credentials via the process DEK (`master.key`).
Set `BOOKCLERK_AUTH_PASSWORD` (preferred) or `[auth].password` to wrap the
DEK with a passphrase at rest (`BCK1` → `BCK2`). A later password can be
applied via CLI (`config master-key wrap` / `config set auth.password`) or
daemon config reload — no GUI in this surface.

**Fields:**
- `password: Option<String>` - Optional passphrase wrapping `master.key`. Prefer `BOOKCLERK_AUTH_PASSWORD`
- `oidc: crate::OidcBrokerConfig` - Optional upstream OIDC/OAuth identity broker (`[auth.oidc]`).

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &AuthConfig) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> AuthConfig`
- **Clone**
  - `fn clone(self: &Self) -> AuthConfig`



## bookclerk_config::settings::Config

*Struct*

Top-level Bookclerk configuration.

**Fields:**
- `paths: Option<crate::paths::Paths>` - Filesystem layout (resolved at load time; not always present in TOML).
- `library: LibraryConfig` - Library / scan / acquire behaviour (`[library]`).
- `database: crate::database::DatabaseConfig` - Library database backend plugin (`[database]`).
- `output: crate::output::OutputConfig` - Acquire packaging (`[output]`). After control-plane enrollment every
- `daemon: DaemonConfig` - HTTP control-plane / tray settings (`[daemon]`).
- `auth: AuthConfig` - Master-key wrapping passphrase (`[auth]`).
- `integrations: crate::plugins::IntegrationsConfig` - Connect-portal settings (`[integrations]`). Not diagnostics.
- `discovery: DiscoveryConfig` - Discovery / recommendations / request queue (`[discovery]`).
- `diagnostics: DiagnosticsConfig` - Opt-in crash / error-burst report upload (`[diagnostics]`).
- `media: crate::MediaConfig` - Media decode / encode worker pool (`[media]`).
- `jobs: crate::JobsConfig` - Durable job queue (`[jobs]`).
- `events: crate::EventsConfig` - Durable domain-event outbox (`[events]`).
- `plugins: crate::PluginsConfig` - How external plugin guests are run (`[plugins]`).
- `events_revision: Option<i64>` - Revision of the DB-authoritative `[events]` document after overlay.
- `events_authority: Option<String>` - Cluster id that produced `Self::events_revision`.
- `output_revision: Option<i64>` - Revision of the DB-authoritative `[output]` document (`core.output`)
- `output_authority: Option<String>` - Cluster id that produced `Self::output_revision`. Not serialized.

**Methods:**

- `fn load(cli_files_dir: Option<PathBuf>, config_path: Option<PathBuf>) -> Result<Self>` - Load config: defaults ← optional TOML file ← environment overrides.
- `fn from_toml_file(path: &Path) -> Result<Self>` - Deserialize configuration from a TOML file on disk.
- `fn from_toml_str(text: &str, origin: &str) -> Result<Self>` - Deserialize configuration from a TOML string (tests / embedded configs).
- `fn apply_env_overrides(self: & mut Self)` - Apply `BOOKCLERK_*` environment overrides.
- `fn validate(self: &Self) -> Result<()>` - Soft validation of cross-field constraints.
- `fn auth_password(self: &Self) -> Option<String>` - Resolve the passphrase that wraps `master.key`.
- `fn register_known_secrets(self: &Self)` - Register config/env secrets for exact-value log redaction.
- `fn resolve_relative_paths(self: & mut Self)` - Resolve relative file paths (Widevine CDM, scratch dir, database
- `fn warn_unsupported_options(self: &Self)` - Warn / note about Widevine / auth encryption setup.
- `fn write_toml_file(self: &Self, path: &Path) -> Result<()>` - Write the config as TOML (skips resolved `paths`).
- `fn to_toml_string(self: &Self) -> Result<String>` - Serialize to a TOML string suitable for `config.toml`.
- `fn paths(self: &Self) -> &Paths` - Resolved paths (panic only if `load` was not used — callers should use `load`).
- `fn download_cache_dir(self: &Self) -> PathBuf` - Scratch directory for downloads / decrypt (`InProgress` or default cache).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &Config) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Config`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Config`



## bookclerk_config::settings::DaemonAuthConfig

*Struct*

Operator token / session settings for the daemon HTTP API.

**Fields:**
- `enabled: bool` - When true, `/api/*` (and legacy control routes) require Bearer or session cookie.
- `session_ttl_hours: u64` - Browser session lifetime in hours after `POST /api/auth/login`.
- `login_max_failures: u32` - Failed `POST /api/auth/login` attempts (per client IP) before a lockout.
- `login_lockout_secs: u64` - How long a client stays locked out after exceeding `Self::login_max_failures`.
- `require_second_factor: bool` - When true, password login requires TOTP or the user must sign in with a passkey.
- `tray_handoff_ttl_secs: u64` - How long a tray/CLI loopback `?code=` ticket stays valid (clamped to 30..=900).

**Methods:**

- `fn tray_handoff_ttl_secs_clamped(self: &Self) -> u64` - Ticket lifetime used when minting a loopback sign-in code.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DaemonAuthConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DaemonAuthConfig`



## bookclerk_config::settings::DaemonConfig

*Struct*

Daemon / HTTP control plane settings.

**Fields:**
- `listen: crate::ListenAddrs` - Bind addresses for the HTTP control plane (string or array in TOML).
- `json_logs: bool` - Emit JSON logs on stderr when true (journald sink is always structured).
- `tray: bool` - When true, `bookclerkd` starts an in-process system tray in a graphical
- `auth: DaemonAuthConfig` - Operator authentication for the HTTP API / GUI.
- `trusted_proxies: Vec<String>` - Peer addresses trusted to set `X-Forwarded-For` / `Forwarded` (CIDR or IP).
- `node_api: NodeApiConfig` - Transport policy for the host-node API (`[daemon.node_api]`).

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DaemonConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DaemonConfig) -> bool`



## bookclerk_config::settings::DiagnosticsConfig

*Struct*

Opt-in sharing of recent **redacted** logs on crash or error bursts.

Defaults to disabled. Operators flip `share_reports` and set `collector_url`
(or bake `BOOKCLERK_DIAGNOSTICS_COLLECTOR_URL` at `cargo build` time).
The client POSTs to `/submit`; a GitHub Action calls `/report`.

Bookclerk never manages log-file rotation — use journald / the container
runtime (Windows Event Log / macOS unified logging are future work).

**Fields:**
- `share_reports: bool` - When true, share redacted crash/ERROR-burst reports with the collector.
- `collector_url: String` - Worker origin (HTTPS). When empty, uses `BOOKCLERK_DIAGNOSTICS_COLLECTOR_URL`
- `upload_on_crash: bool` - Upload the ring buffer from the panic hook.
- `upload_on_error_burst: bool` - Upload when ERROR volume crosses the burst threshold.
- `upload_on_warn_burst: bool` - Upload when WARN volume crosses the warn-burst threshold.
- `error_burst_threshold: u32` - Number of ERROR events inside the window that trigger an upload.
- `error_burst_window_secs: u64` - Sliding window length for error-burst detection, in seconds.
- `warn_burst_threshold: u32` - Number of WARN events inside the window that trigger an upload.
- `warn_burst_window_secs: u64` - Sliding window length for warn-burst detection, in seconds.
- `ring_buffer_capacity: u32` - Max redacted events retained for upload (all levels through TRACE).

**Methods:**

- `fn effective_collector_url(self: &Self) -> String` - Resolved Worker origin: config/env `collector_url`, else baked deploy URL.
- `fn effective_submit_url(self: &Self) -> String` - HTTPS endpoint clients POST redacted JSON to (`…/submit`).
- `fn upload_ready(self: &Self) -> bool` - True when report sharing is enabled and a collector URL can be resolved.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DiagnosticsConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DiagnosticsConfig`



## bookclerk_config::settings::DiscoveryConfig

*Struct*

Discovery / recommendations / embeddings settings.

**Fields:**
- `embeddings_enabled: bool` - Run local embeddings for similarity scoring of storefront candidates.
- `embedding_model: String` - Model id (`all-minilm-l6-v2-q` preferred; runtime may fall back to `local-hash-v1`).
- `embed_intra_threads: usize` - ONNX intra-op threads (keep at 1 on small VPSes).
- `openlibrary_enabled: bool` - Fill metadata gaps via Open Library (low-volume, cached; not for bulk).
- `openlibrary_contact_email: Option<String>` - Contact email for Open Library User-Agent (API guidelines).
- `openlibrary_max_requests_per_run: usize` - Max Open Library HTTP calls per enrich run.
- `worldcat_enabled: bool` - Reserved for a future WorldCat provider (requires API key).
- `storefront_candidates: bool` - Expand recommendations from storefront catalogs (unowned titles).
- `storefront_seed_limit: usize` - Max local taste seeds used for storefront expansion.
- `storefront_max_remote_calls: usize` - Cap remote storefront HTTP calls per recommend run.
- `exclude_graphicaudio_series_sets: bool` - When true, drop GraphicAudio Magento series-set SKUs from candidates.
- `listen_sync_interval_minutes: u64` - How often `bookclerkd` syncs ABS listening progress (0 = disabled).
- `recommend_limit: usize` - Default number of titles returned by a recommendations run (default 20).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DiscoveryConfig) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DiscoveryConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`



## bookclerk_config::settings::LETS_ENCRYPT_DIRECTORY

*Constant*: `&str`

Let's Encrypt's production ACME directory.



## bookclerk_config::settings::LibraryConfig

*Struct*

Library / scan related settings.

**Fields:**
- `auto_acquire: bool` - Automatically acquire newly scanned titles (daemon).
- `scan_interval_minutes: u64` - Scan interval in minutes for `bookclerkd` (0 = disabled).
- `import_episodes: bool` - Import podcast episodes during scan (`ImportEpisodes`).
- `import_plus_titles: bool` - Import Audible Plus titles during scan (`ImportPlusTitles`).
- `download_episodes: bool` - Acquire podcast episodes (`DownloadEpisodes`; distinct from `ImportEpisodes`).
- `save_podcasts_to_parent_folder: bool` - Save podcast episodes into the parent show's folder (`SavePodcastsToParentFolder`).
- `enrich_from_audible: bool` - After scan, enrich non-Audible rows (e.g. Libro.fm) from public Audible/Audnexus metadata.
- `enrich_min_confidence: u8` - Minimum match confidence (0–100) to accept an Audible ASIN enrichment.
- `fix_storage_layout: bool` - When matching storage to the library, relocate matched audio (and

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &LibraryConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> LibraryConfig`



## bookclerk_config::settings::NODE_API_ACME_DIR

*Constant*: `&str`

Directory under `$BOOKCLERK_FILES_DIR` holding the ACME account and
certificate cache (mode 0700 on Unix).



## bookclerk_config::settings::NODE_API_SELF_SIGNED_DIR

*Constant*: `&str`

Directory under `$BOOKCLERK_FILES_DIR` holding the self-signed node API
certificate, its key, and its SPKI pin (mode 0700 on Unix).



## bookclerk_config::settings::NODE_API_SPKI_PIN_FILE

*Constant*: `&str`

File in [`NODE_API_SELF_SIGNED_DIR`](settings.md#bookclerk_configsettingsnode_api_self_signed_dir) holding the SPKI pin
(base64url SHA-256 of the certificate's SubjectPublicKeyInfo).



## bookclerk_config::settings::NodeApiConfig

*Struct*

`[daemon.node_api]`: transport policy for host nodes.

**Fields:**
- `mode: NodeApiMode` - Which connections may reach the node API.
- `acme_contact: Option<String>` - ACME account contact (`mailto:ops@example.com`). Required for `acme`.
- `tls_listen: Option<String>` - Address of the built-in TLS listener (`0.0.0.0:8443`). Required for
- `acme_directory: Option<String>` - ACME directory URL. Defaults to Let's Encrypt production
- `acme_directory_ca: Option<String>` - PEM file of extra root certificates trusted for the ACME directory's

**Methods:**

- `fn is_default(self: &Self) -> bool` - True for the default table (skipped when writing `config.toml`).
- `fn validate(self: &Self, trusted_proxies: &[String], public_origin: Option<&str>) -> std::result::Result<(), String>` - Checks that the mode has what it needs. An invalid table makes the

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> NodeApiConfig`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> NodeApiConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeApiConfig) -> bool`



## bookclerk_config::settings::NodeApiMode

*Enum*

How host nodes may reach the node API (`[daemon.node_api] mode`).

Plain HTTP is accepted only on loopback, in every mode.

**Variants:**
- `Loopback` - Loopback connections only (the default).
- `Proxy` - A TLS-terminating reverse proxy listed in `daemon.trusted_proxies`.
- `Acme` - The built-in TLS listener with an ACME (Let's Encrypt) certificate.
- `SelfSigned` - The built-in TLS listener with a self-signed certificate whose SPKI

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Stable TOML spelling.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeApiMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NodeApiMode`
- **Default**
  - `fn default() -> NodeApiMode`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::settings::ProcessEnvGuard

*Struct*

Process-wide lock around `BOOKCLERK_*` environment reads and test writes.

`std::env::set_var` races with `std::env::var`. `Config::load` holds this
lock while it reads overrides. A test that publishes an override holds the
same lock for the whole mutation, including the later load on that thread.

**Methods:**

- `fn enter() -> Self` - Blocks until this thread owns the process environment lock.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`



## bookclerk_config::settings::TRAY_HANDOFF_TTL_SECS_DEFAULT

*Constant*: `u64`

Default tray/CLI loopback sign-in link lifetime (seconds).



## bookclerk_config::settings::TRAY_HANDOFF_TTL_SECS_MAX

*Constant*: `u64`

Maximum allowed `DaemonAuthConfig::tray_handoff_ttl_secs`.



## bookclerk_config::settings::TRAY_HANDOFF_TTL_SECS_MIN

*Constant*: `u64`

Minimum allowed `DaemonAuthConfig::tray_handoff_ttl_secs`.



## bookclerk_config::settings::read_node_api_spki_pin

*Function*

The SPKI pin of the daemon's self-signed node API certificate, once the
daemon has generated it. Enrollment links carry it in
`[daemon.node_api] mode = "self_signed"`.

```rust
fn read_node_api_spki_pin(files_dir: &std::path::Path) -> Option<String>
```



