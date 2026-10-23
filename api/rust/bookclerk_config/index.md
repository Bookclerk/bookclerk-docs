# bookclerk_config

Host configuration, files-dir layout, and logging for Bookclerk.

## Audience

Host binaries (`bookclerk`, `bookclerkd`) and in-process host crates — not
guest plugins. Parses `config.toml`, applies env overrides, resolves
`BOOKCLERK_FILES_DIR`, and installs tracing / diagnostics.

Product narrative: `docs/configuration.md`, `docs/diagnostics.md`. Style:
`docs/code-documentation.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`DatabaseConfig`](#databaseconfig) | struct | Top-level `[database]` — selects the active backend plugin. |
| [`DatabaseD1Config`](#databased1config) | struct | Cloudflare D1 backend (`[database.d1]`). |
| [`DatabasePostgresConfig`](#databasepostgresconfig) | struct | PostgreSQL backend (`[database.postgres]`). |
| [`DatabaseSqliteConfig`](#databasesqliteconfig) | struct | Local SQLite destination (`[database.sqlite]`). |
| [`BufferedEvent`](#bufferedevent) | struct | Snapshot of a single redacted log event kept for crash / error uploads. |
| [`DiagnosticsHandle`](#diagnosticshandle) | struct | Process-global diagnostics handle installed by `crate::logging::init_tracing_with`. |
| [`UploadPayload`](#uploadpayload) | struct | JSON body POSTed to the diagnostics collector `/submit` endpoint. |
| [`EventsConfig`](#eventsconfig) | struct | `[events]` — outbox retention and local delivery-worker concurrency. |
| [`LameConfig`](#lameconfig) | struct | LAME encoder tuning (classic `Lame*` settings). |
| [`ReplacementRule`](#replacementrule) | struct | Character replacement rule (`ReplacementCharacters` in classic Settings.json). |
| [`JobsConcurrencyConfig`](#jobsconcurrencyconfig) | struct | Per-resource-class worker counts and cluster-wide job semaphores. |
| [`JobsConfig`](#jobsconfig) | struct | `[jobs]` — durable queue admission, leases, and resource-class concurrency. |
| [`OsLogLayer`](#osloglayer) | struct | Structured sink that writes **redacted** events to the platform log facility. |
| [`ListenAddrs`](#listenaddrs) | struct | One or more bind addresses for the HTTP control plane. |
| [`LoggingHandle`](#logginghandle) | struct | Result of installing the global subscriber. |
| [`TracingOptions`](#tracingoptions) | struct | Options for `init_tracing_with`. |
| [`MediaConfig`](#mediaconfig) | struct | `[media]` — decode, encode, and packaging worker pool. |
| [`NamingProfileTemplates`](#namingprofiletemplates) | struct | Concrete templates contributed by a `NamingProfile`. |
| [`ResolvedNamingTemplates`](#resolvednamingtemplates) | struct | Resolved folder / file / chapter-file templates after applying profile defaults and optional per-field overrides. |
| [`OidcBrokerConfig`](#oidcbrokerconfig) | struct | `[auth.oidc]` — optional identity broker. |
| [`OidcProviderConfig`](#oidcproviderconfig) | struct | One upstream OIDC or OAuth 2.0 provider. |
| [`DestinationNaming`](#destinationnaming) | struct | Optional naming overrides for one destination plugin. |
| [`OutputConfig`](#outputconfig) | struct | Acquire packaging (`[output]`). |
| [`PathLimits`](#pathlimits) | struct | Limits applied when hardening acquired storage keys. |
| [`Paths`](#paths) | struct | Resolved filesystem locations for Bookclerk data. |
| [`IntegrationsConfig`](#integrationsconfig) | struct | Connect-portal settings under `[integrations]`. |
| [`PluginRegistryEntry`](#pluginregistryentry) | struct | One configured plugin registry source (`[[plugins.registries]]`). |
| [`PluginsConfig`](#pluginsconfig) | struct | `[plugins]` — how external plugin guests are run. |
| [`PluginsJailConfig`](#pluginsjailconfig) | struct | Optional global resource ceilings for plugin guest jails under `[plugins.jail]`. |
| [`AuthConfig`](#authconfig) | struct | Auth encryption settings (`[auth]` section). |
| [`Config`](#config) | struct | Top-level Bookclerk configuration. |
| [`DaemonAuthConfig`](#daemonauthconfig) | struct | Operator token / session settings for the daemon HTTP API. |
| [`DaemonConfig`](#daemonconfig) | struct | Daemon / HTTP control plane settings. |
| [`DiagnosticsConfig`](#diagnosticsconfig) | struct | Opt-in sharing of recent **redacted** logs on crash or error bursts. |
| [`DiscoveryConfig`](#discoveryconfig) | struct | Discovery / recommendations / embeddings settings. |
| [`LibraryConfig`](#libraryconfig) | struct | Library / scan related settings. |
| [`NodeApiConfig`](#nodeapiconfig) | struct | `[daemon.node_api]`: transport policy for host nodes. |
| [`ProcessEnvGuard`](#processenvguard) | struct | Process-wide lock around `BOOKCLERK_*` environment reads and test writes. |
| [`DatabasePluginKind`](#databasepluginkind) | enum | Which first-party database plugin is active. |
| [`ConfigError`](#configerror) | enum | Errors produced while loading or validating configuration. |
| [`FileTimestampMode`](#filetimestampmode) | enum | Which timestamp to apply after acquire (`CreationTime` / `LastWriteTime`). |
| [`PathSanitizationMode`](#pathsanitizationmode) | enum | How to sanitize storage path segments when `crate::OutputConfig::replacement_characters` is empty (explicit rules always win). |
| [`Isolation`](#isolation) | enum | What to do when a child process cannot be confined. |
| [`OsLogFacility`](#oslogfacility) | enum | Which OS facility was attached (if any). |
| [`LogFormat`](#logformat) | enum | Output format for the stderr sink. |
| [`NamingProfile`](#namingprofile) | enum | Preset path-template profile. |
| [`OidcProvisionMode`](#oidcprovisionmode) | enum | How an upstream provider decides who may become a first-party User. |
| [`ResolveOperatorTokenEnv`](#resolveoperatortokenenv) | enum | How an operator token was resolved from env (config-layer only). |
| [`BadBookAction`](#badbookaction) | enum | How to handle acquire failures (`BadBook` setting). |
| [`MultiDestinationMode`](#multidestinationmode) | enum | Behavior when some output destinations already have a title and others do not. |
| [`OutputBackendKind`](#outputbackendkind) | enum | Which output destination plugin is active. |
| [`PathLengthMeasure`](#pathlengthmeasure) | enum | How to measure path component length when truncating. |
| [`ChapterJsonMode`](#chapterjsonmode) | enum | Whether / which chapter JSON sidecars to write beside acquired audio. |
| [`OutputFormat`](#outputformat) | enum | Post-download output formatting. Default is enriched M4B (tags + chapters). |
| [`AudioQuality`](#audioquality) | enum | Audible download bitrate tier (`high` / `normal` in TOML). |
| [`NodeApiMode`](#nodeapimode) | enum | How host nodes may reach the node API (`[daemon.node_api] mode`). |
| [`replace_file`](#replace_file) | function | Replace `to` with `from`, both in the same directory. `to` may already exist; on Windows this goes through `MoveFileExW` with `MOVEFILE_REPLACE_EXISTING \| MOVEFILE_WRITE_THROUGH` (see the module docs). The moved file keeps its own permissions, so a `0600` staging file lands as a `0600` target. |
| [`cookie_secure_suffix`](#cookie_secure_suffix) | function | Append `; Secure` when the configured public origin is HTTPS. |
| [`oidc_transaction_cookie_flags`](#oidc_transaction_cookie_flags) | function | Cookie flags for the OIDC login transaction (`state` binding). |
| [`session_cookie_flags`](#session_cookie_flags) | function | Build common session cookie attributes (`HttpOnly; SameSite=Lax` + optional Secure). |
| [`resolve_d1_api_token`](#resolve_d1_api_token) | function | Resolve the D1 API token from the environment (host-mediated secret). |
| [`resolve_postgres_url`](#resolve_postgres_url) | function | Resolve the Postgres connection URL from config (env already applied). |
| [`graphical_session_available`](#graphical_session_available) | function | True when it is reasonable to start the in-process tray (windowing + session bus). |
| [`diagnostics_global`](#diagnostics_global) | function | Accessor for the process-global diagnostics handle. |
| [`apply_replacements`](#apply_replacements) | function | Apply replacement rules to a string (classic filename sanitization). |
| [`classic_key_aliases`](#classic_key_aliases) | function | Classic Settings.json key → dotted config path for `config get` / `config set`. |
| [`default_replacement_characters`](#default_replacement_characters) | function | Alias for `windows_replacement_characters` (classic default name). |
| [`key_matches_reconcile_pattern`](#key_matches_reconcile_pattern) | function | True when `key` matches a reconcile `pattern` built with `reconciliation_wildcard_rules` (wildcard = any single character). |
| [`parse_replacement_characters`](#parse_replacement_characters) | function | Parse classic `ReplacementCharacters` JSON. |
| [`posix_replacement_characters`](#posix_replacement_characters) | function | Path-separator sanitization for POSIX local filesystems. |
| [`reconciliation_wildcard_rules`](#reconciliation_wildcard_rules) | function | Replacement rules that turn every known sanitizable character into `RECONCILE_WILDCARD`. |
| [`resolve_replacement_characters`](#resolve_replacement_characters) | function | Resolve the character-replacement map used when sanitising storage path segments. |
| [`s3_replacement_characters`](#s3_replacement_characters) | function | Characters AWS recommends avoiding in S3 object keys, plus path separators. |
| [`windows_replacement_characters`](#windows_replacement_characters) | function | NTFS-safe replacement map (classic Libation Windows defaults subset). |
| [`journald_available`](#journald_available) | function | Back-compat alias. |
| [`os_log_available`](#os_log_available) | function | Probe whether an OS log facility appears available. |
| [`init_tracing`](#init_tracing) | function | Convenience wrapper matching the historical CLI/daemon call site. |
| [`init_tracing_with`](#init_tracing_with) | function | Install a global tracing subscriber (stderr + optional OS facility + diagnostics). |
| [`strip_ansi_escapes`](#strip_ansi_escapes) | function | Remove ECMA-48/ANSI escape sequences so piped guest logs stay plain text. |
| [`allowlist_permits`](#allowlist_permits) | function | Whether an allowlist provider admits this `(email, sub)`. |
| [`email_domain_allowed`](#email_domain_allowed) | function | Whether `email` matches an allowlist of domains (empty list = unrestricted). |
| [`oidc_apple_private_key_env_key`](#oidc_apple_private_key_env_key) | function | Environment variable holding a Sign in with Apple `.p8` private key. |
| [`oidc_client_secret_env_key`](#oidc_client_secret_env_key) | function | Environment variable holding a provider's client secret. |
| [`oidc_secret_store_name`](#oidc_secret_store_name) | function | `encrypted_secrets.name` for an OIDC client secret or Apple key. |
| [`resolve_mapped_role`](#resolve_mapped_role) | function | Pick the highest Bookclerk role from mapped claim values. |
| [`generate_operator_token`](#generate_operator_token) | function | Generate a 32-byte hex operator token. |
| [`read_operator_token_env`](#read_operator_token_env) | function | Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty. |
| [`validate_operator_token`](#validate_operator_token) | function | Reject tokens that can break URL fragments, HTTP headers, or shell pastes. |
| [`normalize_storage_prefix`](#normalize_storage_prefix) | function | Normalize a storage key prefix: empty stays empty; otherwise ensure a trailing `/`. |
| [`apply_config_updates`](#apply_config_updates) | function | Load config, apply `pairs`, and write the result back to the resolved config path. |
| [`apply_config_updates_from_path`](#apply_config_updates_from_path) | function | Load `path`, apply `pairs`, and write the updated TOML back to the same path. |
| [`apply_setting_overrides`](#apply_setting_overrides) | function | Apply classic-style `-o Setting=value` overrides onto `config`. |
| [`enforce_storage_key_limits`](#enforce_storage_key_limits) | function | Enforce per-segment and full-key limits on a relative storage key. |
| [`path_len`](#path_len) | function | Measure `s` according to `measure`. |
| [`truncate_filename_stem`](#truncate_filename_stem) | function | Truncate a filename stem so `stem + "." + ext` fits in `max_filename_length`. |
| [`truncate_path_component`](#truncate_path_component) | function | Truncate `s` so its measured length is ≤ `limit`. Never splits a UTF-8 char. |
| [`resolve_config_path`](#resolve_config_path) | function | Resolve config file path: `BOOKCLERK_CONFIG` → `{files_dir}/config.toml`. |
| [`resolve_files_dir`](#resolve_files_dir) | function | Resolve the Bookclerk files directory. |
| [`detect_distro`](#detect_distro) | function | Best-effort OS distribution string (e.g. `Ubuntu 24.04.1 LTS`). |
| [`host_cpu_rate_max`](#host_cpu_rate_max) | function | Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`). |
| [`node_logical_cpus`](#node_logical_cpus) | function | Logical CPUs visible to this process (at least 1). |
| [`contains_registered_secret`](#contains_registered_secret) | function | True if `haystack` still contains any registered exact secret (post-redaction check). |
| [`is_sensitive_field`](#is_sensitive_field) | function | Returns true when a tracing field name should never be logged in cleartext. |
| [`is_upload_identifying_field`](#is_upload_identifying_field) | function | Field names scrubbed from **remote** diagnostics uploads in addition to `is_sensitive_field`. Local journal/stderr may still show them. |
| [`redact_field_value`](#redact_field_value) | function | Redact a field value: sensitive names are fully replaced; others are scrubbed. |
| [`redact_str`](#redact_str) | function | Scrub known secrets: exact registered values, then pattern matches. |
| [`register_secret`](#register_secret) | function | Register a known secret so it is always scrubbed from logs and diagnostics. |
| [`register_secrets`](#register_secrets) | function | Register several secrets at once. |
| [`register_secrets_from_env`](#register_secrets_from_env) | function | Register secrets commonly present in the process environment / config. |
| [`sanitize_for_remote_upload`](#sanitize_for_remote_upload) | function | Extra sanitization for payloads that leave the machine (collector / B2). |
| [`truncate_upload_message`](#truncate_upload_message) | function | Cap message length used in remote diagnostics events. |
| [`unregister_secret`](#unregister_secret) | function | Forget a secret `register_secret` registered, with its encoded forms, once nothing in the process holds it any more (a short-lived bearer token whose holder was dropped), so per-request credentials do not pile up in the registry every log line is checked against. Lines captured while it was registered were scrubbed then. Only for values unique to one holder: the registry keeps no count, so a value two holders share would stop being scrubbed for both. |
| [`read_node_api_spki_pin`](#read_node_api_spki_pin) | function | The SPKI pin of the daemon's self-signed node API certificate, once the daemon has generated it. Enrollment links carry it in `[daemon.node_api] mode = "self_signed"`. |
| [`Result`](#result) | type alias | Result alias for configuration load and validation helpers in this crate. |
| [`RECONCILE_WILDCARD`](#reconcile_wildcard) | constant | Sentinel placed in reconcile path *patterns* where a sanitizable character appeared in metadata. Matches any single character in an on-disk key. |
| [`DEFAULT_MAX_FILENAME_LENGTH`](#default_max_filename_length) | constant | Classic Libation `LongPath.MaxFilenameLength` — per path component. |
| [`S3_MAX_OBJECT_KEY_BYTES`](#s3_max_object_key_bytes) | constant | AWS S3 maximum object key length (UTF-8 bytes), including any prefix. |
| [`BOOKCLERK_CONFIG_ENV`](#bookclerk_config_env) | constant | Override for the config file path (`BOOKCLERK_CONFIG`). |
| [`BOOKCLERK_FILES_DIR_ENV`](#bookclerk_files_dir_env) | constant | Environment variable for the Bookclerk data directory. |
| [`REDACTED`](#redacted) | constant | Replacement token used wherever a secret or sensitive value is scrubbed. |
| [`LETS_ENCRYPT_DIRECTORY`](#lets_encrypt_directory) | constant | Let's Encrypt's production ACME directory. |
| [`NODE_API_ACME_DIR`](#node_api_acme_dir) | constant | Directory under `$BOOKCLERK_FILES_DIR` holding the ACME account and certificate cache (mode 0700 on Unix). |
| [`NODE_API_SELF_SIGNED_DIR`](#node_api_self_signed_dir) | constant | Directory under `$BOOKCLERK_FILES_DIR` holding the self-signed node API certificate, its key, and its SPKI pin (mode 0700 on Unix). |
| [`NODE_API_SPKI_PIN_FILE`](#node_api_spki_pin_file) | constant | File in `NODE_API_SELF_SIGNED_DIR` holding the SPKI pin (base64url SHA-256 of the certificate's SubjectPublicKeyInfo). |
| [`TRAY_HANDOFF_TTL_SECS_DEFAULT`](#tray_handoff_ttl_secs_default) | constant | Default tray/CLI loopback sign-in link lifetime (seconds). |
| [`TRAY_HANDOFF_TTL_SECS_MAX`](#tray_handoff_ttl_secs_max) | constant | Maximum allowed `DaemonAuthConfig::tray_handoff_ttl_secs`. |
| [`TRAY_HANDOFF_TTL_SECS_MIN`](#tray_handoff_ttl_secs_min) | constant | Minimum allowed `DaemonAuthConfig::tray_handoff_ttl_secs`. |

## Structs

### DatabaseConfig

```rust
pub struct DatabaseConfig {
    pub plugin: String,
    pub sqlite: DatabaseSqliteConfig,
    pub d1: DatabaseD1Config,
    pub postgres: DatabasePostgresConfig,
    pub plugins: Table,
}
```

Top-level `[database]` — selects the active backend plugin.

**Fields**

- `plugin`: Active plugin id (first-party `sqlite` / `d1` / `postgres`, or any
  third-party database adapter plugin id).
- `sqlite`: Local SQLite knobs (`[database.sqlite]`).
- `d1`: Cloudflare D1 knobs (`[database.d1]`).
- `postgres`: PostgreSQL knobs (`[database.postgres]`).
- `plugins`: Opaque `[database.<id>]` tables for third-party database adapters.
  
  The host does not interpret these; the granted table is delivered to
  the adapter as its `DatabaseAdapterConfig.config` payload.

#### DatabaseConfig::active_plugin

```rust
pub fn active_plugin(&self) -> Result<DatabasePluginKind>
```

Parsed first-party active plugin; `Err` for third-party adapter ids.

##### Errors

Returns an error when the id is not a first-party plugin id.

#### DatabaseConfig::plugin_table

```rust
pub fn plugin_table(&self, id: &str) -> Option<&Table>
```

Borrow the opaque `[database.<id>]` table for a third-party adapter.

#### DatabaseConfig::sqlite_path

```rust
pub fn sqlite_path(&self, files_dir: &Path) -> PathBuf
```

Resolve the SQLite file path (relative paths join `files_dir`).

#### DatabaseConfig::validate

```rust
pub fn validate(&self) -> Result<()>
```

Soft validation for the selected plugin (required ids / URL presence).

##### Errors

Returns [`crate::ConfigError::Invalid`](#configerror) when the active plugin is unknown
or missing required fields.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### DatabaseD1Config

```rust
pub struct DatabaseD1Config {
    pub enabled: bool,
    pub account_id: String,
    pub database_id: String,
    pub api_base: String,
}
```

Cloudflare D1 backend (`[database.d1]`).

The API token is supplied via the `BOOKCLERK_D1_API_TOKEN` or
`CLOUDFLARE_API_TOKEN` environment variable (no credentials file).

**Fields**

- `enabled`: Kept for symmetry with other plugins; ignored when another plugin is active.
- `account_id`: Cloudflare account id that owns the D1 database.
- `database_id`: D1 database UUID from the Cloudflare dashboard / API.
- `api_base`: Cloudflare API base (default `https://api.cloudflare.com/client/v4`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DatabasePostgresConfig

```rust
pub struct DatabasePostgresConfig {
    pub enabled: bool,
    pub url: Option<String>,
    pub url_file: Option<PathBuf>,
}
```

PostgreSQL backend (`[database.postgres]`).

Connect via a standard Postgres connection URL:
`postgres://user:password@host:5432/dbname`

The URL contains credentials — prefer `url_file` (pointing at a secrets volume)
or the `BOOKCLERK_DATABASE_POSTGRES_URL` environment variable over embedding the
URL in TOML.

**Fields**

- `enabled`: Kept for symmetry with other plugins; ignored when another plugin is active.
- `url`: PostgreSQL connection URL (e.g. `postgres://user:pass@host/db`).
  Registered as a secret for redaction.  Prefer `url_file` in production.
- `url_file`: Path to a file containing the connection URL (one line, trimmed).
  Relative paths are resolved under `$BOOKCLERK_FILES_DIR`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DatabaseSqliteConfig

```rust
pub struct DatabaseSqliteConfig {
    pub enabled: bool,
    pub path: Option<PathBuf>,
}
```

Local SQLite destination (`[database.sqlite]`).

**Fields**

- `enabled`: Kept for symmetry with other plugins; ignored when another plugin is active.
- `path`: DB file path (default `library.db` under the files dir).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### BufferedEvent

```rust
pub struct BufferedEvent {
    pub ts_unix_ms: u64,
    pub level: String,
    pub target: String,
    pub message: String,
    pub fields: Vec<(String, String)>,
}
```

Snapshot of a single redacted log event kept for crash / error uploads.

**Fields**

- `ts_unix_ms`: Event time as Unix milliseconds (wall clock).
- `level`: Tracing level name (`ERROR`, `WARN`, `INFO`, …).
- `target`: Tracing target / module path.
- `message`: Redacted primary message text.
- `fields`: Additional redacted structured fields as `(name, value)` pairs.

Implements: `Clone`, `Debug`, `Serialize`

### DiagnosticsHandle

```rust
pub struct DiagnosticsHandle {
    /* private fields */
}
```

Process-global diagnostics handle installed by [`crate::logging::init_tracing_with`](#init_tracing_with).

#### DiagnosticsHandle::upload_enabled

```rust
pub fn upload_enabled(&self) -> bool
```

Whether automatic upload is configured and ready (token/url present).

#### DiagnosticsHandle::snapshot_events

```rust
pub fn snapshot_events(&self) -> Vec<BufferedEvent>
```

Snapshot of the in-memory diagnostics ring (newest last).

#### DiagnosticsHandle::version

```rust
pub fn version(&self) -> &str
```

Bookclerk version string recorded at subscriber install.

#### DiagnosticsHandle::uploads_attempted

```rust
pub fn uploads_attempted(&self) -> u64
```

Number of upload attempts since init (successful or failed HTTP tries).

#### DiagnosticsHandle::record_event

```rust
pub fn record_event(&self, event: &Event<'_>)
```

Record a redacted event into the ring buffer and maybe trigger an error-burst upload.

#### DiagnosticsHandle::request_upload

```rust
pub fn request_upload(&self, trigger: &'static str)
```

Request an upload of the current ring (e.g. after a failed daemon job).

Explicit failure uploads bypass the burst cooldown so a recent burst cannot
suppress `cli_error` / `job_failed` reports. If another upload is in flight,
this waits briefly for the slot rather than dropping the request.

#### DiagnosticsHandle::upload_blocking

```rust
pub fn upload_blocking(&self, trigger: &str)
```

Best-effort upload of the current ring buffer (blocking).

Used by the panic hook, CLI exit path, and tests. Waits for any in-flight
upload to finish (serialized upload gate) so crash reports
are not dropped and concurrent POSTs cannot overlap.

Implements: `Clone`

### UploadPayload

```rust
pub struct UploadPayload {
    pub trigger: String,
    pub version: String,
    pub os: String,
    pub arch: String,
    pub distro: Option<String>,
    pub rustc_release: String,
    pub rustc_channel: String,
    pub archived_at_unix_ms: u64,
    pub events: Vec<BufferedEvent>,
}
```

JSON body POSTed to the diagnostics collector `/submit` endpoint.

**Fields**

- `trigger`: Why this upload was produced (`crash`, `error_burst`, `cli_error`, …).
- `version`: Bookclerk version string recorded at subscriber install.
- `os`: Host OS (`linux`, `macos`, `windows`, …).
- `arch`: CPU architecture (`x86_64`, `aarch64`, …).
- `distro`: Best-effort Linux distro id when detectable.
- `rustc_release`: `rustc` release baked in at build time (or `"unknown"`).
- `rustc_channel`: `rustc` channel baked in at build time (or `"unknown"`).
- `archived_at_unix_ms`: Upload assembly time as Unix milliseconds.
- `events`: Redacted ring-buffer snapshot (oldest first).

Implements: `Clone`, `Debug`, `Serialize`

### EventsConfig

```rust
pub struct EventsConfig {
    pub retention_days: u64,
    pub dead_letter_retention_days: u64,
    pub concurrency: u32,
}
```

`[events]` — outbox retention and local delivery-worker concurrency.

**Fields**

- `retention_days`: Days to retain acked/rejected deliveries and parent events with no
  remaining live deliveries.
- `dead_letter_retention_days`: Days to retain `dead_letter` deliveries (independent of [`Self::retention_days`](#eventsconfig)).
- `concurrency`: Local delivery workers (`[events.concurrency]`; default 1).

#### EventsConfig::apply_env_overrides

```rust
pub fn apply_env_overrides(&mut self)
```

Apply `BOOKCLERK_EVENTS_*` environment overrides.

Environment wins over TOML, matching every other section.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LameConfig

```rust
pub struct LameConfig {
    pub target: String,
    pub vbr_quality: u8,
    pub bitrate_kbps: u32,
    pub mode: String,
    pub downsample_mono: bool,
    pub constant_bitrate: bool,
}
```

LAME encoder tuning (classic `Lame*` settings).

**Fields**

- `target`: `LameTarget`: `quality`, `bitrate`, `constant`, etc.
- `vbr_quality`: `LameVBRQuality` (0–9, lower = better).
- `bitrate_kbps`: `LameBitrate` in kbps when target is bitrate/constant.
- `mode`: `LameMode`: `default`, `mono`, `stereo`.
- `downsample_mono`: `LameDownsampleMono`.
- `constant_bitrate`: `LameConstantBitrate`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ReplacementRule

```rust
pub struct ReplacementRule {
    pub find: String,
    pub replace: String,
}
```

Character replacement rule (`ReplacementCharacters` in classic Settings.json).

**Fields**

- `find`: Substring to search for in each path segment / tag value.
- `replace`: Replacement text (may be empty to strip `find`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobsConcurrencyConfig

```rust
pub struct JobsConcurrencyConfig {
    pub network: u32,
    pub plugin_instance: u32,
}
```

Per-resource-class worker counts and cluster-wide job semaphores.

**Fields**

- `network`: Scan / acquire / listening-sync workers on this node (default 1),
  and the cluster-wide number of running network jobs (the
  `jobs:network` semaphore).
- `plugin_instance`: Cluster-wide running jobs that require one plugin instance (the
  `plugin-instance:<id>` semaphore; default 1).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobsConfig

```rust
pub struct JobsConfig {
    pub max_pending: u32,
    pub lease_seconds: u64,
    pub max_attempts: u32,
    pub retention_days: u64,
    pub temp_quota_bytes: u64,
    pub concurrency: JobsConcurrencyConfig,
}
```

`[jobs]` — durable queue admission, leases, and resource-class concurrency.

**Fields**

- `max_pending`: Maximum pending+running jobs. Further admits return queue-full.
- `lease_seconds`: Lease length in seconds; expired running jobs are reclaimed.
- `max_attempts`: Maximum claims before a failure is terminal.
- `retention_days`: Days to retain succeeded/failed/cancelled rows.
- `temp_quota_bytes`: Refuse new acquire scratch dirs when registered temps exceed this.
- `concurrency`: Per-class concurrency (`[jobs.concurrency]`).

#### JobsConfig::apply_env_overrides

```rust
pub fn apply_env_overrides(&mut self)
```

Apply `BOOKCLERK_JOBS_*` environment overrides.

Environment wins over TOML, matching every other section.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OsLogLayer

```rust
pub struct OsLogLayer {
    /* private fields */
}
```

Structured sink that writes **redacted** events to the platform log facility.

#### OsLogLayer::new

```rust
pub fn new(syslog_identifier: impl Into<String>) -> Result<Self>
```

Connect to the platform facility. Returns `Err` when unavailable.

##### Errors

Returns an error when the operation fails.

#### OsLogLayer::facility

```rust
pub fn facility(&self) -> OsLogFacility
```

Which platform sink this layer is writing to.

Implements: `Layer<S>`

### ListenAddrs

```rust
pub struct ListenAddrs(/* private */);
```

One or more bind addresses for the HTTP control plane.

TOML accepts either a string or an array; we always serialize as an array.

#### ListenAddrs::new

```rust
pub fn new(addrs: Vec<String>) -> Self
```

Build from raw address strings, dropping empties; falls back to defaults
when every entry is empty.

#### ListenAddrs::parse_list

```rust
pub fn parse_list(raw: &str) -> Result<Self, String>
```

Parse a single address or a comma-separated list
(`127.0.0.1:8787,[::1]:8787`).

##### Errors

Returns an error when the operation fails.

#### ListenAddrs::as_slice

```rust
pub fn as_slice(&self) -> &[String]
```

Configured bind strings in declaration order.

#### ListenAddrs::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no bind addresses remain (should not occur after [`Self::new`](#listenaddrsnew)).

#### ListenAddrs::join_comma

```rust
pub fn join_comma(&self) -> String
```

Comma-joined form for settings API / overrides.

#### ListenAddrs::socket_addrs

```rust
pub fn socket_addrs(&self) -> Result<Vec<SocketAddr>, String>
```

Parse each entry as a [`SocketAddr`](https://doc.rust-lang.org/stable/core/net/socket_addr/enum.SocketAddr.html).

##### Errors

Returns an error when the operation fails.

#### ListenAddrs::validate

```rust
pub fn validate(&self) -> Result<(), String>
```

Validate every entry parses as a socket address.

##### Errors

Returns an error when the operation fails.

#### ListenAddrs::ui_port

```rust
pub fn ui_port(&self) -> u16
```

Port used for tray `http://localhost:<port>` (first entry's port).

#### ListenAddrs::ports

```rust
pub fn ports(&self) -> Vec<u16>
```

Distinct TCP ports from configured bind addresses.

#### ListenAddrs::has_loopback

```rust
pub fn has_loopback(&self) -> bool
```

Whether any configured bind is loopback.

#### ListenAddrs::has_unspecified

```rust
pub fn has_unspecified(&self) -> bool
```

Whether any configured bind is unspecified (`0.0.0.0` / `::`).

#### ListenAddrs::tray_base_url

```rust
pub fn tray_base_url(&self) -> String
```

Prefer `http://localhost:<port>` when any loopback or unspecified bind is
configured (wildcard listeners still accept loopback); else the first
concrete bind (bracket IPv6 as needed).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Display`, `Eq`, `From<&str>`, `FromStr`, `PartialEq`, `Serialize`

### LoggingHandle

```rust
pub struct LoggingHandle {
    pub diagnostics: DiagnosticsHandle,
    pub journald: bool,
    pub os_facility: Option<OsLogFacility>,
    /* private fields */
}
```

Result of installing the global subscriber.

Keep this value alive for the process lifetime so the stderr worker thread
stays running ([`WorkerGuard`](https://docs.rs/tracing_appender/latest/tracing_appender/non_blocking/struct.WorkerGuard.html)).

**Fields**

- `diagnostics`: Diagnostics ring buffer / upload handle.
- `journald`: Whether an OS facility sink was attached.
- `os_facility`: Which facility was attached (if any).

### TracingOptions

```rust
pub struct TracingOptions {
    pub format: LogFormat,
    pub default_level: String,
    pub syslog_identifier: String,
    pub diagnostics: DiagnosticsConfig,
    pub version: String,
    pub enable_journald: bool,
}
```

Options for [`init_tracing_with`](#init_tracing_with).

**Fields**

- `format`: Stderr format (text vs JSON).
- `default_level`: Default `EnvFilter` directive when `BOOKCLERK_LOG` / `RUST_LOG` are unset.
  Affects stderr + OS facility only — the diagnostics ring always keeps TRACE.
- `syslog_identifier`: Identifier for journald / Event Log source / os_log category.
- `diagnostics`: Opt-in crash / error-burst upload settings.
- `version`: Package version embedded in diagnostics uploads.
- `enable_journald`: Attempt to attach an OS log facility (no-op when unavailable).

Implements: `Clone`, `Debug`, `Default`

### MediaConfig

```rust
pub struct MediaConfig {
    pub workers: usize,
    pub isolation: Isolation,
    pub worker_bin: Option<PathBuf>,
}
```

`[media]` — decode, encode, and packaging worker pool.

**Fields**

- `workers`: Maximum codec jobs running at once. `0` derives one worker per core,
  capped at 8, unless this process's cgroup reports `memory.max` at or
  below 1 GiB or a `cpu.max` quota of at most one core — then the
  automatic choice is 1. A value above 0 is an explicit operator cap.
- `isolation`: How strictly workers must be confined.
- `worker_bin`: Explicit path to `bookclerk-media-worker`. Normally left unset, in which
  case the worker is found beside the running executable.

#### MediaConfig::apply_env_overrides

```rust
pub fn apply_env_overrides(&mut self)
```

Apply `BOOKCLERK_MEDIA_*` environment overrides.

Environment wins over TOML, matching every other section.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NamingProfileTemplates

```rust
pub struct NamingProfileTemplates {
    pub folder: &'static str,
    pub file: &'static str,
    pub chapter_file: &'static str,
}
```

Concrete templates contributed by a [`NamingProfile`](#namingprofile).

**Fields**

- `folder`: Default folder-path template for this profile (Libation tag syntax).
- `file`: Default file-stem template for this profile (no extension).
- `chapter_file`: Default per-chapter file-stem template when splitting by chapter.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ResolvedNamingTemplates

```rust
pub struct ResolvedNamingTemplates {
    pub folder: String,
    pub file: String,
    pub chapter_file: String,
}
```

Resolved folder / file / chapter-file templates after applying profile
defaults and optional per-field overrides.

**Fields**

- `folder`: Folder-path template after profile defaults and optional overrides.
- `file`: File-stem template after profile defaults and optional overrides.
- `chapter_file`: Chapter file-stem template after profile defaults and optional overrides.

#### ResolvedNamingTemplates::resolve

```rust
pub fn resolve(
    profile: NamingProfile,
    folder: Option<&str>,
    file: Option<&str>,
    chapter_file: Option<&str>,
) -> Self
```

Resolve templates: each explicit override wins over the profile default.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### OidcBrokerConfig

```rust
pub struct OidcBrokerConfig {
    pub enabled: bool,
    pub allowed_email_domains: Vec<String>,
    pub providers: Vec<OidcProviderConfig>,
    pub secret_generation: u64,
}
```

`[auth.oidc]` — optional identity broker.

**Fields**

- `enabled`: When false, RP routes report disabled and login buttons are hidden.
- `allowed_email_domains`: Global email-domain allowlist applied after the provider policy.
- `providers`: Upstream providers (enterprise and/or social).
- `secret_generation`: Durable `encrypted_secrets` generation for this broker revision.
  
  Runtime and `config.toml` must agree before rows for this generation
  are used. `0` is the unversioned legacy name.

#### OidcBrokerConfig::is_unset

```rust
pub fn is_unset(&self) -> bool
```

Whether this section is the serde default (disabled, no providers).

#### OidcBrokerConfig::enabled_providers

```rust
pub fn enabled_providers(&self) -> Vec<&OidcProviderConfig>
```

Enabled providers with a non-empty id.

#### OidcBrokerConfig::provider

```rust
pub fn provider(&self, id: &str) -> Option<&OidcProviderConfig>
```

Look up a provider by id.

#### OidcBrokerConfig::validate

```rust
pub fn validate(&self) -> Result<()>
```

Reject operator maps, unknown roles, duplicate ids, and empty ids when enabled.

##### Errors

Returns an error when the operation fails.

#### OidcBrokerConfig::validate_providers

```rust
pub fn validate_providers(&self) -> Result<()>
```

Validate provider rows even when the broker is disabled.

Used by the Settings API so a disabled-but-invalid draft cannot be
written and then fail [`crate::Config::load`](#configload) when later enabled.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OidcProviderConfig

```rust
pub struct OidcProviderConfig {
    pub id: String,
    pub name: String,
    pub preset: Option<String>,
    pub issuer: Option<String>,
    pub client_id: String,
    pub client_secret: Option<String>,
    pub scopes: Vec<String>,
    pub provision: OidcProvisionMode,
    pub default_role: String,
    pub role_claim: String,
    pub role_map: BTreeMap<String, String>,
    pub link_by_email: bool,
    pub apple_team_id: Option<String>,
    pub apple_key_id: Option<String>,
    pub apple_private_key: Option<String>,
    pub allowed_email_domains: Vec<String>,
    pub allowed_emails: Vec<String>,
    pub allowed_subjects: Vec<String>,
}
```

One upstream OIDC or OAuth 2.0 provider.

**Fields**

- `id`: Stable id used in URLs and `portal_identities.provider` (`oidc:{id}`).
- `name`: Login-button label.
- `preset`: Optional well-known preset (`google`, `github`, `apple`, `discord`).
- `issuer`: OIDC issuer URL (discovery). Ignored when `preset` fills endpoints.
- `client_id`: OAuth client id.
- `client_secret`: Optional client secret (prefer env / `encrypted_secrets`).
- `scopes`: Scopes requested at authorize.
- `provision`: Provisioning policy for this provider.
- `default_role`: Role used for [`OidcProvisionMode::Any`](#oidcprovisionmode) (and allowlist without maps).
- `role_claim`: Claim name holding groups/roles (typically `groups` or `roles`).
- `role_map`: Upstream group/role value → Bookclerk role (`owner`/`administrator`/`member`).
- `link_by_email`: When true, a new `sub` may attach to the unique matching *verified* email User.
  
  Defaults off in TOML for generic issuers: an unverified upstream email
  must not attach to an existing Owner. Settings defaults this **on** for
  Google / GitHub / Apple / Discord presets.
- `apple_team_id`: Sign in with Apple team ID (`iss` of the client-secret JWT).
- `apple_key_id`: Sign in with Apple key ID (`kid` header of the client-secret JWT).
- `apple_private_key`: PEM-encoded Apple `.p8` private key (prefer env / `encrypted_secrets`).
- `allowed_email_domains`: Email domains allowed for this provider (empty = no extra filter).
- `allowed_emails`: Exact emails allowed when `provision = allowlist`.
- `allowed_subjects`: Explicit upstream `sub` values allowed when `provision = allowlist`.

#### OidcProviderConfig::portal_provider

```rust
pub fn portal_provider(&self) -> String
```

Portal identity provider key (`oidc:{id}`).

#### OidcProviderConfig::display_name

```rust
pub fn display_name(&self) -> &str
```

Display name, falling back to id.

#### OidcProviderConfig::social_preset

```rust
pub fn social_preset(&self) -> Option<&'static str>
```

Canonical social preset (`google`, `github`, `apple`, `discord`).

Matching is ASCII case-insensitive so `GitHub` and `apple` select the
same endpoints, scopes, and profile adapter.

#### OidcProviderConfig::effective_scopes

```rust
pub fn effective_scopes(&self) -> Vec<String>
```

Scopes sent at authorize, with social-preset defaults when unset.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DestinationNaming

```rust
pub struct DestinationNaming {
    pub naming_profile: Option<NamingProfile>,
    pub folder_template: Option<String>,
    pub file_template: Option<String>,
    pub chapter_file_template: Option<String>,
}
```

Optional naming overrides for one destination plugin.

**Fields**

- `naming_profile`: Override `[output].naming_profile` for this destination when set.
- `folder_template`: Optional folder-path template override (Libation tag syntax).
- `file_template`: Optional file-stem template override (no extension).
- `chapter_file_template`: Optional per-chapter file-stem template when splitting by chapter.

#### DestinationNaming::resolve_over

```rust
pub fn resolve_over(&self, global: &OutputConfig) -> ResolvedNamingTemplates
```

Merge destination overrides over global `[output]` naming.

#### DestinationNaming::effective_profile

```rust
pub fn effective_profile(&self, global: &OutputConfig) -> NamingProfile
```

Profile after destination override (else global).

#### DestinationNaming::effective_folder_template

```rust
pub fn effective_folder_template(&self, global: &OutputConfig) -> Option<String>
```

Destination folder template, falling back to global `[output]`.

#### DestinationNaming::effective_file_template

```rust
pub fn effective_file_template(&self, global: &OutputConfig) -> Option<String>
```

Destination file-stem template, falling back to global `[output]`.

#### DestinationNaming::effective_chapter_file_template

```rust
pub fn effective_chapter_file_template(&self, global: &OutputConfig) -> Option<String>
```

Destination chapter-file template, falling back to global `[output]`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OutputConfig

```rust
pub struct OutputConfig {
    pub format: OutputFormat,
    pub widevine: bool,
    pub xhe_aac: bool,
    pub widevine_cdm: Option<PathBuf>,
    pub widevine_cdm_provider: Option<String>,
    pub naming_profile: NamingProfile,
    pub folder_template: Option<String>,
    pub file_template: Option<String>,
    pub download_cover: bool,
    pub download_pdf: bool,
    pub create_cue: bool,
    pub fixup_metadata: bool,
    pub chapter_json: ChapterJsonMode,
    pub save_chapter_json: Option<bool>,
    pub save_metadata_json: bool,
    pub cover_size: String,
    pub chapter_layout: String,
    pub overwrite_existing: bool,
    pub multi_destination: MultiDestinationMode,
    pub in_progress: Option<PathBuf>,
    pub bad_book_action: BadBookAction,
    pub split_mp3_max_mb: u32,
    pub chapter_file_template: Option<String>,
    pub chapter_title_template: Option<String>,
    pub minimum_file_duration_minutes: u32,
    pub combine_nested_chapter_titles: bool,
    pub merge_opening_and_end_credits: bool,
    pub strip_unabridged: bool,
    pub strip_audible_brand_audio: bool,
    pub download_clips_bookmarks: bool,
    pub retain_aax_file: bool,
    pub download_speed_limit_kbps: u32,
    pub lame: LameConfig,
    pub max_sample_rate: Option<u32>,
    pub creation_time: FileTimestampMode,
    pub last_write_time: FileTimestampMode,
    pub path_sanitization: PathSanitizationMode,
    pub replacement_characters: Vec<ReplacementRule>,
    pub max_filename_length: u32,
}
```

Acquire packaging (`[output]`).

Destination plugins (`local`, `s3`, …) declare their own `[[settings]]`
(root, bucket, prefix, per-destination naming overrides) and are
configured on their plugin instances; nothing destination-specific lives
in `config.toml`.

**Fields**

- `format`: Post-acquire packaging format (default: enriched M4B).
- `widevine`: Prefer Widevine/CENC (also enables Adrm→Widevine fallback; auto-provisions L3 CDM).
- `xhe_aac`: Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm`: Optional local Widevine `.wvd` path (absolute or relative to `BOOKCLERK_FILES_DIR`).
- `widevine_cdm_provider`: Remote L3 CDM provider. `None` uses classic Libation AudibleCdm; empty/`off` disables.
- `naming_profile`: Named path-template preset (`audiobookshelf` default, or `classic`).
- `folder_template`: Optional folder-path template override; `None` uses the naming profile default.
- `file_template`: Optional file-stem template override; `None` uses the naming profile default.
- `download_cover`: Save cover JPEG alongside audio.
- `download_pdf`: Download companion PDF when available.
- `create_cue`: Write a `.cue` sidecar from API chapters.
- `fixup_metadata`: Embed tags, cover, and chapters natively.
- `chapter_json`: Chapter JSON sidecars: `off` | `flat` | `tree` | `both`.
- `save_chapter_json`: Deprecated: use [`Self::chapter_json`](#outputconfig).
- `save_metadata_json`: Persist raw catalog API JSON (`metadata.json`).
- `cover_size`: Cover image size for download/embed (`500`, `1215`, or `native`).
- `chapter_layout`: Preferred Audible chapter API layout when fetching (`tree` or `flat`).
- `overwrite_existing`: Re-acquire when media already exists at the destination.
- `multi_destination`: When multiple destinations are enabled and only some already have the
  title: copy from a present dest, re-fetch into missing only, or re-fetch
  into every destination.
- `in_progress`: Scratch directory for in-progress work; relative to files_dir.
- `bad_book_action`: Operator action when a title fails to acquire (`ask` / `abort` / `retry` / `ignore`).
- `split_mp3_max_mb`: Max MP3 part size in MiB when format is `split_mp3_by_size`.
- `chapter_file_template`: Optional per-chapter file-stem template; `None` uses the naming profile default.
- `chapter_title_template`: Optional embedded chapter-title template when rewriting chapter names.
- `minimum_file_duration_minutes`: Drop chapter splits shorter than this many minutes (`0` = keep all).
- `combine_nested_chapter_titles`: When true, flatten nested chapter titles into a single path segment.
- `merge_opening_and_end_credits`: When true, merge opening/end credit chapters into adjacent content chapters.
- `strip_unabridged`: When true, strip an "Unabridged" suffix from titles used in naming.
- `strip_audible_brand_audio`: When true, trim Audible brand intro/outro audio from the remux window.
- `download_clips_bookmarks`: When true, download Audible clips/bookmarks sidecars when the store offers them.
- `retain_aax_file`: Keep encrypted download in storage (`RetainAaxFile`).
- `download_speed_limit_kbps`: Fetch speed cap in KB/s (`0` = unlimited).
- `lame`: LAME encoder knobs used when packaging to MP3.
- `max_sample_rate`: Optional ceiling for output sample rate in Hz (`None` = leave source rate).
- `creation_time`: How to set the file creation / birth timestamp after acquire.
- `last_write_time`: How to set the file last-write / mtime after acquire.
- `path_sanitization`: Auto / Windows / POSIX / S3 sanitisation profile when replacement rules are empty.
- `replacement_characters`: Explicit find/replace rules for path segments (wins over [`Self::path_sanitization`](#outputconfig)).
- `max_filename_length`: Max length per path segment. Default 255; `0` disables truncation.

#### OutputConfig::effective_format

```rust
pub fn effective_format(&self) -> OutputFormat
```

Packaging format used for acquire (currently the raw [`Self::format`](#outputconfig) field).

#### OutputConfig::effective_chapter_json

```rust
pub fn effective_chapter_json(&self) -> ChapterJsonMode
```

Resolved chapter JSON sidecar mode (default off).

#### OutputConfig::resolve_naming_templates

```rust
pub fn resolve_naming_templates(&self) -> ResolvedNamingTemplates
```

Resolve folder / file / chapter-file templates.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PathLimits

```rust
pub struct PathLimits {
    pub max_filename_length: usize,
    pub max_storage_key_bytes: usize,
    pub measure: PathLengthMeasure,
}
```

Limits applied when hardening acquired storage keys.

**Fields**

- `max_filename_length`: Max length per folder / file-stem segment. `0` disables per-segment caps.
- `max_storage_key_bytes`: Max UTF-8 bytes for the full relative storage key (`folder/…/file.ext`).
  `0` disables. For S3, set to [`S3_MAX_OBJECT_KEY_BYTES`](#s3_max_object_key_bytes) minus prefix len.
- `measure`: Whether segment truncation counts Unicode scalars or UTF-8 bytes.

#### PathLimits::local

```rust
pub fn local(measure: PathLengthMeasure) -> Self
```

Local-filesystem defaults: 255-length segments, no full-key cap.

#### PathLimits::s3

```rust
pub fn s3(prefix: &str) -> Self
```

S3 defaults: 255-byte segments plus a full-key budget of
[`S3_MAX_OBJECT_KEY_BYTES`](#s3_max_object_key_bytes) minus the normalized `prefix` UTF-8 length
(trailing `/` included when the prefix is non-empty).

#### PathLimits::resolve

```rust
pub fn resolve(
    max_filename_length: u32,
    storage_is_s3: bool,
    s3_prefix: &str,
    path_sanitization_is_windows: bool,
) -> Self
```

Resolve from config: Windows sanitization → char measure; S3 → key budget.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Paths

```rust
pub struct Paths {
    pub files_dir: PathBuf,
    pub config_file: PathBuf,
    pub library_db: PathBuf,
    pub cache_dir: PathBuf,
    pub log_dir: PathBuf,
    pub search_index_dir: PathBuf,
    pub models_dir: PathBuf,
}
```

Resolved filesystem locations for Bookclerk data.

**Fields**

- `files_dir`: Root data directory (accounts, DB, settings).
- `config_file`: Resolved config.toml path (may be overridden by `--config` / `BOOKCLERK_CONFIG`).
- `library_db`: SQLite library database path.
- `cache_dir`: Temporary download / decrypt scratch space.
- `log_dir`: Reserved path (legacy). Logging goes to stderr + journald; Bookclerk does
  not write or rotate files here.
- `search_index_dir`: Tantivy full-text search index (classic Lucene `SearchEngine`).
- `models_dir`: Downloaded ONNX embedding models (`discovery.embeddings_enabled`).

#### Paths::from_files_dir

```rust
pub fn from_files_dir(files_dir: PathBuf) -> Self
```

Build path set from an explicit files directory.

#### Paths::ensure_dirs

```rust
pub fn ensure_dirs(&self) -> Result<()>
```

Ensure directories that Bookclerk expects to exist are present.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### IntegrationsConfig

```rust
pub struct IntegrationsConfig {
    pub claim_ticket_ttl_hours: u64,
    pub public_origin: Option<String>,
    pub portal_session_ttl_hours: u64,
}
```

Connect-portal settings under `[integrations]`.

Integration plugins themselves (Audiobookshelf, …) are deployed plugin
instances; their settings live on the instance, not here.

**Fields**

- `claim_ticket_ttl_hours`: Hours a connect-portal claim ticket remains valid (default 72).
- `public_origin`: Public HTTPS origin for portal redirects when behind a reverse proxy
  (e.g. `https://bookclerk.example.com`). `None` accepts only a bound
  loopback `Origin` / `Host`; non-loopback hosts must set this pin.
- `portal_session_ttl_hours`: Hours a portal browser session cookie remains valid (default 12).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginRegistryEntry

```rust
pub struct PluginRegistryEntry {
    pub kind: String,
    pub url: Option<String>,
    pub name: Option<String>,
}
```

One configured plugin registry source (`[[plugins.registries]]`).

**Fields**

- `kind`: Adapter kind: `static`, `cargo`, `npm`, or `pypi`.
- `url`: Index / registry URL (required for `static`; optional overrides for others).
- `name`: Optional operator-facing name.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginsConfig

```rust
pub struct PluginsConfig {
    pub isolation: Isolation,
    pub jail_bin: Option<PathBuf>,
    pub jail: PluginsJailConfig,
    pub registries: Vec<PluginRegistryEntry>,
    pub allow_unverified_publisher: bool,
}
```

`[plugins]` — how external plugin guests are run.

Separate from `[sources.*]` / `[integrations.*]`, which say *which* plugins
to load and pass them their own knobs. This says how the host runs whichever
ones it loads.

**Fields**

- `isolation`: How strictly guest processes must be confined.
- `jail_bin`: Explicit path to `bookclerk-jail`. Normally left unset, in which case the
  launcher is found beside the running executable.
- `jail`: Optional global jail resource ceilings for all plugin guests.
- `registries`: Federated discovery sources (static indexes, cargo, npm, pypi).
  
  Search order is list order; default when empty is crates.io only.
- `allow_unverified_publisher`: When true, community packages without independent publisher authenticity
  may be installed without `--allow-unverified-publisher` (digests are still
  required). Bookclerk does not verify publisher signatures.

#### PluginsConfig::apply_env_overrides

```rust
pub fn apply_env_overrides(&mut self)
```

Apply `BOOKCLERK_PLUGIN_*` environment overrides.

Environment wins over TOML, matching every other section.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginsJailConfig

```rust
pub struct PluginsJailConfig {
    pub memory_mib: Option<u64>,
    pub cpu_rate_percent: Option<u32>,
    pub extra_processes: Option<u32>,
}
```

Optional global resource ceilings for plugin guest jails under `[plugins.jail]`.

These override label defaults when building a guest jail spec.
Guest filesystem access remains install read-only plus host-managed data/tmp —
not a free-form path widen.

`cpu_rate_percent` is percent of **one logical CPU** (100 = one core) and is a
**per-jail ceiling** only (`min(grant, this, host_max)`). Quotas are rate
limits, not reservations: if many plugins’ ceilings sum above host capacity,
the OS scheduler shares CPU among runnable guests. Bookclerk does not keep a
cumulative Σ ledger or throttle later spawns.

**Fields**

- `memory_mib`: Soft memory ceiling in mebibytes (mapped to Spec `memory_bytes`).
- `cpu_rate_percent`: Per-jail CPU rate ceiling (one-core percent units). Defaults to 80.
- `extra_processes`: Ceiling on **extra** processes/threads beyond launcher overhead. Defaults to 2.

#### PluginsJailConfig::clamp

```rust
pub fn clamp(&mut self)
```

Clamp CPU / extra-process ceilings into host-safe ranges when set.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AuthConfig

```rust
pub struct AuthConfig {
    pub password: Option<String>,
    pub oidc: OidcBrokerConfig,
}
```

Auth encryption settings (`[auth]` section).

Bookclerk always encrypts credentials via the process DEK (`master.key`).
Set `BOOKCLERK_AUTH_PASSWORD` (preferred) or `[auth].password` to wrap the
DEK with a passphrase at rest (`BCK1` → `BCK2`). A later password can be
applied via CLI (`config master-key wrap` / `config set auth.password`) or
daemon config reload — no GUI in this surface.

**Fields**

- `password`: Optional passphrase wrapping `master.key`. Prefer `BOOKCLERK_AUTH_PASSWORD`
  env (wins when both are set). Registered for log redaction on load.
- `oidc`: Optional upstream OIDC/OAuth identity broker (`[auth.oidc]`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Config

```rust
pub struct Config {
    pub paths: Option<Paths>,
    pub library: LibraryConfig,
    pub database: DatabaseConfig,
    pub output: OutputConfig,
    pub daemon: DaemonConfig,
    pub auth: AuthConfig,
    pub integrations: IntegrationsConfig,
    pub discovery: DiscoveryConfig,
    pub diagnostics: DiagnosticsConfig,
    pub media: MediaConfig,
    pub jobs: JobsConfig,
    pub events: EventsConfig,
    pub plugins: PluginsConfig,
    pub events_revision: Option<i64>,
    pub events_authority: Option<String>,
    pub output_revision: Option<i64>,
    pub output_authority: Option<String>,
}
```

Top-level Bookclerk configuration.

**Fields**

- `paths`: Filesystem layout (resolved at load time; not always present in TOML).
- `library`: Library / scan / acquire behaviour (`[library]`).
- `database`: Library database backend plugin (`[database]`).
- `output`: Acquire packaging (`[output]`). After control-plane enrollment every
  field except the host-local `in_progress` is overlaid from the
  `core.output` database document; the TOML table only seeds it once.
- `daemon`: HTTP control-plane / tray settings (`[daemon]`).
- `auth`: Master-key wrapping passphrase (`[auth]`).
- `integrations`: Connect-portal settings (`[integrations]`). Not diagnostics.
- `discovery`: Discovery / recommendations / request queue (`[discovery]`).
- `diagnostics`: Opt-in crash / error-burst report upload (`[diagnostics]`).
- `media`: Media decode / encode worker pool (`[media]`).
- `jobs`: Durable job queue (`[jobs]`).
- `events`: Durable domain-event outbox (`[events]`).
- `plugins`: How external plugin guests are run (`[plugins]`).
- `events_revision`: Revision of the DB-authoritative `[events]` document after overlay.
  
  Absent until control-plane bootstrap or reconcile loads `core.events`.
  Not serialized to `config.toml`. Compared only with
  [`Self::events_authority`](#config).
- `events_authority`: Cluster id that produced [`Self::events_revision`](#config).
  
  A revision from another cluster is not newer or older; a database swap
  replaces this authority explicitly. Not serialized to `config.toml`.
- `output_revision`: Revision of the DB-authoritative `[output]` document (`core.output`)
  after overlay. Absent until control-plane bootstrap or reconcile loads
  it; compared only with [`Self::output_authority`](#config). Not serialized.
- `output_authority`: Cluster id that produced [`Self::output_revision`](#config). Not serialized.

#### Config::load

```rust
pub fn load(cli_files_dir: Option<PathBuf>, config_path: Option<PathBuf>) -> Result<Self>
```

Load config: defaults ← optional TOML file ← environment overrides.

##### Arguments

* `cli_files_dir` - Optional override for `BOOKCLERK_FILES_DIR`.
* `config_path` - Optional explicit path to `config.toml`.

##### Errors

Returns [`crate::ConfigError`](#configerror) when the file cannot be read or parsed, or
when soft validation of destinations / database fails after merge.

#### Config::from_toml_file

```rust
pub fn from_toml_file(path: &Path) -> Result<Self>
```

Deserialize configuration from a TOML file on disk.

##### Errors

Returns [`crate::ConfigError::Read`](#configerror) or [`crate::ConfigError::Parse`](#configerror).

#### Config::from_toml_str

```rust
pub fn from_toml_str(text: &str, origin: &str) -> Result<Self>
```

Deserialize configuration from a TOML string (tests / embedded configs).

##### Errors

Returns [`crate::ConfigError::Parse`](#configerror) on invalid TOML.

#### Config::apply_env_overrides

```rust
pub fn apply_env_overrides(&mut self)
```

Apply `BOOKCLERK_*` environment overrides.

#### Config::validate

```rust
pub fn validate(&self) -> Result<()>
```

Soft validation of cross-field constraints.

##### Errors

Returns an error when the operation fails.

#### Config::auth_password

```rust
pub fn auth_password(&self) -> Option<String>
```

Resolve the passphrase that wraps `master.key`.

Prefers `BOOKCLERK_AUTH_PASSWORD` over `[auth].password`.

#### Config::register_known_secrets

```rust
pub fn register_known_secrets(&self)
```

Register config/env secrets for exact-value log redaction.

#### Config::resolve_relative_paths

```rust
pub fn resolve_relative_paths(&mut self)
```

Resolve relative file paths (Widevine CDM, scratch dir, database
files) under `files_dir`.

Absolute paths are unchanged; relative ones become
`{BOOKCLERK_FILES_DIR}/<path>` so systemd/Docker cwd does not matter.

#### Config::warn_unsupported_options

```rust
pub fn warn_unsupported_options(&self)
```

Warn / note about Widevine / auth encryption setup.

#### Config::write_toml_file

```rust
pub fn write_toml_file(&self, path: &Path) -> Result<()>
```

Write the config as TOML (skips resolved `paths`).

Stages a same-directory temp file, `fsync`s it, then atomically replaces
`path` so a crash cannot leave a truncated `config.toml`.

On Unix, `rename` replaces the destination. On Windows, `std::fs::rename`
cannot replace an existing file, so this uses `MoveFileExW` with
`MOVEFILE_REPLACE_EXISTING` instead of delete-then-rename.

##### Errors

Returns an error when the operation fails.

#### Config::to_toml_string

```rust
pub fn to_toml_string(&self) -> Result<String>
```

Serialize to a TOML string suitable for `config.toml`.

##### Errors

Returns an error when the operation fails.

#### Config::paths

```rust
pub fn paths(&self) -> &Paths
```

Resolved paths (panic only if `load` was not used — callers should use `load`).

##### Panics

Panics when an internal invariant does not hold.

#### Config::download_cache_dir

```rust
pub fn download_cache_dir(&self) -> PathBuf
```

Scratch directory for downloads / decrypt (`InProgress` or default cache).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### DaemonAuthConfig

```rust
pub struct DaemonAuthConfig {
    pub enabled: bool,
    pub session_ttl_hours: u64,
    pub login_max_failures: u32,
    pub login_lockout_secs: u64,
    pub require_second_factor: bool,
    pub tray_handoff_ttl_secs: u64,
}
```

Operator token / session settings for the daemon HTTP API.

**Fields**

- `enabled`: When true, `/api/*` (and legacy control routes) require Bearer or session cookie.
- `session_ttl_hours`: Browser session lifetime in hours after `POST /api/auth/login`.
- `login_max_failures`: Failed `POST /api/auth/login` attempts (per client IP) before a lockout.
- `login_lockout_secs`: How long a client stays locked out after exceeding [`Self::login_max_failures`](#daemonauthconfig).
- `require_second_factor`: When true, password login requires TOTP or the user must sign in with a passkey.
- `tray_handoff_ttl_secs`: How long a tray/CLI loopback `?code=` ticket stays valid (clamped to 30..=900).

#### DaemonAuthConfig::tray_handoff_ttl_secs_clamped

```rust
pub fn tray_handoff_ttl_secs_clamped(&self) -> u64
```

Ticket lifetime used when minting a loopback sign-in code.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DaemonConfig

```rust
pub struct DaemonConfig {
    pub listen: ListenAddrs,
    pub json_logs: bool,
    pub tray: bool,
    pub auth: DaemonAuthConfig,
    pub trusted_proxies: Vec<String>,
    pub node_api: NodeApiConfig,
}
```

Daemon / HTTP control plane settings.

**Fields**

- `listen`: Bind addresses for the HTTP control plane (string or array in TOML).
  
  Default binds both IPv4 and IPv6 loopback on port 8787.
- `json_logs`: Emit JSON logs on stderr when true (journald sink is always structured).
- `tray`: When true, `bookclerkd` starts an in-process system tray in a graphical
  session (opens the web UI in the system browser).
  
  Headless hosts (no `DISPLAY`/`WAYLAND_DISPLAY`, no session bus) skip the
  tray. Override with `BOOKCLERK_NO_TRAY=1` or `BOOKCLERK_DAEMON_TRAY=0`.
- `auth`: Operator authentication for the HTTP API / GUI.
- `trusted_proxies`: Peer addresses trusted to set `X-Forwarded-For` / `Forwarded` (CIDR or IP).
  
  Empty (default): always use the direct TCP peer — safe behind no proxy.
  When the peer matches an entry, login throttling uses the leftmost
  `X-Forwarded-For` client address (typical single reverse-proxy layout).
  
  In `[daemon.node_api] mode = "proxy"`, a matching peer is also the only
  one whose `X-Forwarded-Proto` the node API believes.
- `node_api`: Transport policy for the host-node API (`[daemon.node_api]`).
  Bootstrap configuration, like [`Self::listen`](#daemonconfig).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DiagnosticsConfig

```rust
pub struct DiagnosticsConfig {
    pub share_reports: bool,
    pub collector_url: String,
    pub upload_on_crash: bool,
    pub upload_on_error_burst: bool,
    pub upload_on_warn_burst: bool,
    pub error_burst_threshold: u32,
    pub error_burst_window_secs: u64,
    pub warn_burst_threshold: u32,
    pub warn_burst_window_secs: u64,
    pub ring_buffer_capacity: u32,
}
```

Opt-in sharing of recent **redacted** logs on crash or error bursts.

Defaults to disabled. Operators flip `share_reports` and set `collector_url`
(or bake `BOOKCLERK_DIAGNOSTICS_COLLECTOR_URL` at `cargo build` time).
The client POSTs to `/submit`; a GitHub Action calls `/report`.

Bookclerk never manages log-file rotation — use journald / the container
runtime (Windows Event Log / macOS unified logging are future work).

**Fields**

- `share_reports`: When true, share redacted crash/ERROR-burst reports with the collector.
- `collector_url`: Worker origin (HTTPS). When empty, uses `BOOKCLERK_DIAGNOSTICS_COLLECTOR_URL`
  from config/runtime env, else the value baked in at `cargo build` time.
- `upload_on_crash`: Upload the ring buffer from the panic hook.
- `upload_on_error_burst`: Upload when ERROR volume crosses the burst threshold.
- `upload_on_warn_burst`: Upload when WARN volume crosses the warn-burst threshold.
- `error_burst_threshold`: Number of ERROR events inside the window that trigger an upload.
- `error_burst_window_secs`: Sliding window length for error-burst detection, in seconds.
- `warn_burst_threshold`: Number of WARN events inside the window that trigger an upload.
- `warn_burst_window_secs`: Sliding window length for warn-burst detection, in seconds.
- `ring_buffer_capacity`: Max redacted events retained for upload (all levels through TRACE).

#### DiagnosticsConfig::effective_collector_url

```rust
pub fn effective_collector_url(&self) -> String
```

Resolved Worker origin: config/env `collector_url`, else baked deploy URL.

#### DiagnosticsConfig::effective_submit_url

```rust
pub fn effective_submit_url(&self) -> String
```

HTTPS endpoint clients POST redacted JSON to (`…/submit`).

#### DiagnosticsConfig::upload_ready

```rust
pub fn upload_ready(&self) -> bool
```

True when report sharing is enabled and a collector URL can be resolved.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DiscoveryConfig

```rust
pub struct DiscoveryConfig {
    pub embeddings_enabled: bool,
    pub embedding_model: String,
    pub embed_intra_threads: usize,
    pub openlibrary_enabled: bool,
    pub openlibrary_contact_email: Option<String>,
    pub openlibrary_max_requests_per_run: usize,
    pub worldcat_enabled: bool,
    pub storefront_candidates: bool,
    pub storefront_seed_limit: usize,
    pub storefront_max_remote_calls: usize,
    pub exclude_graphicaudio_series_sets: bool,
    pub listen_sync_interval_minutes: u64,
    pub recommend_limit: usize,
}
```

Discovery / recommendations / embeddings settings.

**Fields**

- `embeddings_enabled`: Run local embeddings for similarity scoring of storefront candidates.
- `embedding_model`: Model id (`all-minilm-l6-v2-q` preferred; runtime may fall back to `local-hash-v1`).
- `embed_intra_threads`: ONNX intra-op threads (keep at 1 on small VPSes).
- `openlibrary_enabled`: Fill metadata gaps via Open Library (low-volume, cached; not for bulk).
- `openlibrary_contact_email`: Contact email for Open Library User-Agent (API guidelines).
- `openlibrary_max_requests_per_run`: Max Open Library HTTP calls per enrich run.
- `worldcat_enabled`: Reserved for a future WorldCat provider (requires API key).
- `storefront_candidates`: Expand recommendations from storefront catalogs (unowned titles).
- `storefront_seed_limit`: Max local taste seeds used for storefront expansion.
- `storefront_max_remote_calls`: Cap remote storefront HTTP calls per recommend run.
- `exclude_graphicaudio_series_sets`: When true, drop GraphicAudio Magento series-set SKUs from candidates.
  Default false — series sets are included.
- `listen_sync_interval_minutes`: How often `bookclerkd` syncs ABS listening progress (0 = disabled).
- `recommend_limit`: Default number of titles returned by a recommendations run (default 20).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### LibraryConfig

```rust
pub struct LibraryConfig {
    pub auto_acquire: bool,
    pub scan_interval_minutes: u64,
    pub import_episodes: bool,
    pub import_plus_titles: bool,
    pub download_episodes: bool,
    pub save_podcasts_to_parent_folder: bool,
    pub enrich_from_audible: bool,
    pub enrich_min_confidence: u8,
    pub fix_storage_layout: bool,
}
```

Library / scan related settings.

**Fields**

- `auto_acquire`: Automatically acquire newly scanned titles (daemon).
- `scan_interval_minutes`: Scan interval in minutes for `bookclerkd` (0 = disabled).
- `import_episodes`: Import podcast episodes during scan (`ImportEpisodes`).
- `import_plus_titles`: Import Audible Plus titles during scan (`ImportPlusTitles`).
- `download_episodes`: Acquire podcast episodes (`DownloadEpisodes`; distinct from `ImportEpisodes`).
- `save_podcasts_to_parent_folder`: Save podcast episodes into the parent show's folder (`SavePodcastsToParentFolder`).
- `enrich_from_audible`: After scan, enrich non-Audible rows (e.g. Libro.fm) from public Audible/Audnexus metadata.
- `enrich_min_confidence`: Minimum match confidence (0–100) to accept an Audible ASIN enrichment.
  Uses AudioBookshelf-style duration/title/author scoring (default 90).
- `fix_storage_layout`: When matching storage to the library, relocate matched audio (and
  accompanying sidecars) onto the configured naming-profile layout.
  Default false — match in place without moving files.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NodeApiConfig

```rust
pub struct NodeApiConfig {
    pub mode: NodeApiMode,
    pub acme_contact: Option<String>,
    pub tls_listen: Option<String>,
    pub acme_directory: Option<String>,
    pub acme_directory_ca: Option<String>,
}
```

`[daemon.node_api]`: transport policy for host nodes.

**Fields**

- `mode`: Which connections may reach the node API.
- `acme_contact`: ACME account contact (`mailto:ops@example.com`). Required for `acme`.
- `tls_listen`: Address of the built-in TLS listener (`0.0.0.0:8443`). Required for
  `acme` and `self_signed`. ACME's TLS-ALPN-01 challenge reaches it on
  port 443 of the public origin's host, so forward 443 here.
- `acme_directory`: ACME directory URL. Defaults to Let's Encrypt production
  ([`LETS_ENCRYPT_DIRECTORY`](#lets_encrypt_directory)); set it for staging or a private ACME CA.
- `acme_directory_ca`: PEM file of extra root certificates trusted for the ACME directory's
  own HTTPS (a private ACME CA). The webpki roots are always trusted.

#### NodeApiConfig::is_default

```rust
pub fn is_default(&self) -> bool
```

True for the default table (skipped when writing `config.toml`).

#### NodeApiConfig::validate

```rust
pub fn validate(
    &self,
    trusted_proxies: &[String],
    public_origin: Option<&str>,
) -> Result<(), String>
```

Checks that the mode has what it needs. An invalid table makes the
node API refuse every connection.

##### Errors

Returns a description of the first missing or invalid setting.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ProcessEnvGuard

```rust
pub struct ProcessEnvGuard {
    /* private fields */
}
```

Process-wide lock around `BOOKCLERK_*` environment reads and test writes.

`std::env::set_var` races with `std::env::var`. [`Config::load`](#configload) holds this
lock while it reads overrides. A test that publishes an override holds the
same lock for the whole mutation, including the later load on that thread.

#### ProcessEnvGuard::enter

```rust
pub fn enter() -> Self
```

Blocks until this thread owns the process environment lock.

Implements: `Drop`

## Enums

### DatabasePluginKind

```rust
pub enum DatabasePluginKind {
    Sqlite,
    D1,
    Postgres,
}
```

Which first-party database plugin is active.

**Variants**

- `Sqlite`: Local SQLite file under the files directory (default).
- `D1`: Cloudflare D1 via the HTTP API.
- `Postgres`: PostgreSQL via a connection URL.

#### DatabasePluginKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stable TOML / CLI id (`sqlite`, `d1`, `postgres`).

#### DatabasePluginKind::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parse a first-party backend id or kind token (`sqlite`, `postgres`, `pg`, `d1`).

Does **not** inspect PluginKey text. A provenance-qualified occupancy
string such as `cargo:https://evil.example#postgres` is not a kind.
First-party classification belongs in plugin-host after resolving a
discovered install (PluginKey + verified provenance).

##### Returns

`Some` when recognised; `None` for unknown values.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ConfigError

```rust
pub enum ConfigError {
    Read { path: String, source: Error },
    Parse { path: String, source: Error },
    Invalid(String),
    Io(Error),
}
```

Errors produced while loading or validating configuration.

**Variants**

- `Read`: Failed to open or read a config file from disk.
  - `path`: Absolute or display path of the config file that could not be read.
  - `source`: Underlying I/O failure.
- `Parse`: TOML syntax or type error while deserializing a config file.
  - `path`: Absolute or display path of the config file that failed to parse.
  - `source`: Underlying TOML deserialize error.
- `Invalid`: Semantic validation failure after a successful parse (missing required
  fields, mutually exclusive options, unknown plugin ids, …).
- `Io`: Generic I/O failure unrelated to a specific config path context.

Implements: `Debug`, `Display`, `Error`, `From<Error>`

### FileTimestampMode

```rust
pub enum FileTimestampMode {
    Now,
    Purchased,
    Published,
}
```

Which timestamp to apply after acquire (`CreationTime` / `LastWriteTime`).

**Variants**

- `Now`: Use the wall-clock time at acquire completion (default).
- `Purchased`: Use the account purchase / added-to-library timestamp when known.
- `Published`: Use the publisher release date when known.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PathSanitizationMode

```rust
pub enum PathSanitizationMode {
    Auto,
    Windows,
    Posix,
    S3,
    None,
}
```

How to sanitize storage path segments when [`crate::OutputConfig::replacement_characters`](#outputconfig)
is empty (explicit rules always win).

**Variants**

- `Auto`: Pick a profile from the storage backend and host OS.
  
  * `output.s3.enabled = true` → [`s3_replacement_characters`](#s3_replacement_characters)
  * Windows host + local storage → [`windows_replacement_characters`](#windows_replacement_characters)
  * otherwise → [`posix_replacement_characters`](#posix_replacement_characters)
- `Windows`: NTFS / classic Libation Windows map (`: * ? " < > | / \`).
- `Posix`: Path-separator only (`/` `\`) — suitable for Linux/macOS local disks.
- `S3`: AWS S3 “characters to avoid” plus path separators.
- `None`: Disable platform-specific sanitization; path separators (`/` `\`) are
  still stripped so metadata cannot inject extra directories.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Isolation

```rust
pub enum Isolation {
    Required,
    BestEffort,
    Off,
}
```

What to do when a child process cannot be confined.

Shared by every tier that runs untrusted work in a child: `[media]` for
codec workers and `[plugins]` for storefront guests. The tiers differ in what
they reach for, not in how an operator wants a missing jail handled.

**Variants**

- `Required`: Refuse to run the work when the jail does not engage. The default: a
  host that cannot sandbox should not process untrusted input in reach of
  the master key.
- `BestEffort`: Confine where the platform allows and log what did not engage. Use on
  kernels without Landlock, accepting that the work runs unconfined there.
- `Off`: Run with no jail at all. Development only.

#### Isolation::as_str

```rust
pub fn as_str(self) -> &'static str
```

Canonical spelling, as written in `config.toml`.

#### Isolation::parse

```rust
pub fn parse(value: &str) -> Option<Self>
```

Parse a config or environment value.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OsLogFacility

```rust
pub enum OsLogFacility {
    Journald,
    OsLog,
    EventLog,
}
```

Which OS facility was attached (if any).

**Variants**

- `Journald`: Linux systemd journal (`/run/systemd/journal/socket`).
- `OsLog`: Apple Unified Logging (`os_log`).
- `EventLog`: Windows Event Log Application channel.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### LogFormat

```rust
pub enum LogFormat {
    Text,
    Json,
}
```

Output format for the stderr sink.

**Variants**

- `Text`: Human-oriented text (CLI default).
- `Json`: Structured JSON (daemon default / log aggregators).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### NamingProfile

```rust
pub enum NamingProfile {
    Audiobookshelf,
    Classic,
}
```

Preset path-template profile.

Select via `output.naming_profile`. Individual `folder_template` /
`file_template` / `chapter_file_template` values override the profile when
set.

**Variants**

- `Audiobookshelf`: [Audiobookshelf](https://audiobookshelf.org/docs/documentation/libraries/book-library/directory-structure/)
  recommended layout: `{Author}/{Series}/{Book}` or `{Author}/{Book}`,
  with series sequence, publish year, short title, and narrator braces in
  the book folder when available; file is full title with ASIN/ISBN.
- `Classic`: Previous Libation desktop defaults (`Templates.*.DefaultTemplate`).

#### NamingProfile::all

```rust
pub fn all() -> &'static [NamingProfile]
```

All known profiles (for CLI listing).

#### NamingProfile::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stable config / CLI id (`audiobookshelf`, `classic`).

#### NamingProfile::description

```rust
pub fn description(self) -> &'static str
```

Short human description of the profile's layout intent.

#### NamingProfile::parse

```rust
pub fn parse(value: &str) -> Option<Self>
```

Parse a profile id (case-insensitive). Accepts common aliases.

#### NamingProfile::templates

```rust
pub fn templates(self) -> NamingProfileTemplates
```

Built-in folder / file / chapter-file templates for this profile.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OidcProvisionMode

```rust
pub enum OidcProvisionMode {
    MappedRole,
    Any,
    Allowlist,
    InviteOnly,
}
```

How an upstream provider decides who may become a first-party User.

**Variants**

- `MappedRole`: Require a mapped group/role claim (`owner` / `administrator` / `member`).
- `Any`: Any authenticated account; JIT as [`OidcProviderConfig::default_role`](#oidcproviderconfig).
- `Allowlist`: Authenticated and allowlisted (email domain and/or `sub`).
- `InviteOnly`: No JIT; SSO only links a pre-created User.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ResolveOperatorTokenEnv

```rust
pub enum ResolveOperatorTokenEnv {
    Env,
}
```

How an operator token was resolved from env (config-layer only).

**Variants**

- `Env`: Value from `BOOKCLERK_OPERATOR_TOKEN`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### BadBookAction

```rust
pub enum BadBookAction {
    Ask,
    Abort,
    Retry,
    Ignore,
}
```

How to handle acquire failures (`BadBook` setting).

**Variants**

- `Ask`: Prompt the operator (interactive CLI / GUI) for how to proceed (default).
- `Abort`: Stop the acquire batch on the first hard failure.
- `Retry`: Retry the failed title before giving up.
- `Ignore`: Skip the failed title and continue the batch.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MultiDestinationMode

```rust
pub enum MultiDestinationMode {
    SyncMissing,
    RefetchMissing,
    RefetchAll,
}
```

Behavior when some output destinations already have a title and others do not.

**Variants**

- `SyncMissing`: Copy the existing object from a present destination into missing ones
  (no store re-download). Default.
- `RefetchMissing`: Re-run acquire fetch/encode, but write only to destinations that lack
  the title.
- `RefetchAll`: Re-run acquire and write to every enabled destination.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OutputBackendKind

```rust
pub enum OutputBackendKind {
    Local,
    S3,
}
```

Which output destination plugin is active.

**Variants**

- `Local`: Filesystem destination (the `local` plugin; default primary).
- `S3`: Object-store destination (the `s3` plugin).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PathLengthMeasure

```rust
pub enum PathLengthMeasure {
    Chars,
    Utf8Bytes,
}
```

How to measure path component length when truncating.

**Variants**

- `Chars`: Unicode scalar count (classic Libation on Windows).
- `Utf8Bytes`: UTF-8 byte count (classic Libation on Unix; always used for S3 keys).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ChapterJsonMode

```rust
pub enum ChapterJsonMode {
    Off,
    Flat,
    Tree,
    Both,
}
```

Whether / which chapter JSON sidecars to write beside acquired audio.

Default is [`Self::Off`](#chapterjsonmode). Replaces the old `save_chapter_json` bool +
`chapter_layout` pair for *sidecar output* (Audible API fetch layout is
chosen separately when chapters are needed for embedding).

**Variants**

- `Off`: Do not write chapter JSON sidecars.
- `Flat`: Write `chapters.flat.json` only.
- `Tree`: Write `chapters.tree.json` (nested) only.
- `Both`: Write both flat and tree sidecars.

#### ChapterJsonMode::wants_flat

```rust
pub fn wants_flat(self) -> bool
```

True when a flat `chapters.flat.json` sidecar should be written.

#### ChapterJsonMode::wants_tree

```rust
pub fn wants_tree(self) -> bool
```

True when a nested `chapters.tree.json` sidecar should be written.

#### ChapterJsonMode::wants_any

```rust
pub fn wants_any(self) -> bool
```

True when any chapter JSON sidecar mode other than [`Self::Off`](#chapterjsonmode) is set.

#### ChapterJsonMode::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parse `off|flat|tree|both`, or legacy bool-ish values (`true`→tree, `false`→off).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OutputFormat

```rust
pub enum OutputFormat {
    EnrichedM4b,
    SingleMp3,
    SplitMp3ByChapter,
    SplitMp3BySize,
    Opus,
    None,
}
```

Post-download output formatting. Default is enriched M4B (tags + chapters).

**Variants**

- `EnrichedM4b`: Remux/transcode to M4B and embed metadata/chapters (default).
- `SingleMp3`: Single MP3 file (LAME re-encode).
- `SplitMp3ByChapter`: One MP3 per chapter (after M4B chapter split).
- `SplitMp3BySize`: Split MP3 by target file size (see `split_mp3_max_mb`).
- `Opus`: Opus output (not yet implemented).
- `None`: Leave store-delivered media as-is (no remux/transcode/re-encode).

#### OutputFormat::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Parse a TOML / CLI format id (`enriched_m4b`, `mp3`, `none`, …).

##### Returns

`Some` when `raw` matches a known alias; `None` for unknown values.

#### OutputFormat::wants_mp3

```rust
pub fn wants_mp3(self) -> bool
```

True when the format re-encodes to MP3 (single or split).

#### OutputFormat::wants_split_by_chapter

```rust
pub fn wants_split_by_chapter(self) -> bool
```

True when output is one MP3 file per chapter.

#### OutputFormat::wants_split_by_size

```rust
pub fn wants_split_by_size(self) -> bool
```

True when output is split into MP3 parts by target size.

#### OutputFormat::is_noop

```rust
pub fn is_noop(self) -> bool
```

True when store-delivered media is left as-is (no remux/transcode).

#### OutputFormat::wants_opus

```rust
pub fn wants_opus(self) -> bool
```

True when Opus packaging is selected (not yet implemented).

#### OutputFormat::prefers_m4b_container

```rust
pub fn prefers_m4b_container(self) -> bool
```

True when the pipeline prefers an M4B intermediate (enriched or chapter-split).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AudioQuality

```rust
pub enum AudioQuality {
    High,
    Normal,
}
```

Audible download bitrate tier (`high` / `normal` in TOML).

**Variants**

- `High`: Highest offered bitrate for the title (default).
- `Normal`: Standard / lower bitrate when the store offers a choice.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NodeApiMode

```rust
pub enum NodeApiMode {
    Loopback,
    Proxy,
    Acme,
    SelfSigned,
}
```

How host nodes may reach the node API (`[daemon.node_api] mode`).

Plain HTTP is accepted only on loopback, in every mode.

**Variants**

- `Loopback`: Loopback connections only (the default).
- `Proxy`: A TLS-terminating reverse proxy listed in `daemon.trusted_proxies`.
- `Acme`: The built-in TLS listener with an ACME (Let's Encrypt) certificate.
- `SelfSigned`: The built-in TLS listener with a self-signed certificate whose SPKI
  pin travels in enrollment links.

#### NodeApiMode::as_str

```rust
pub fn as_str(self) -> &'static str
```

Stable TOML spelling.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### replace_file

```rust
pub fn replace_file(from: &Path, to: &Path) -> Result<()>
```

Replace `to` with `from`, both in the same directory. `to` may already
exist; on Windows this goes through `MoveFileExW` with
`MOVEFILE_REPLACE_EXISTING | MOVEFILE_WRITE_THROUGH` (see the module
docs). The moved file keeps its own permissions, so a `0600` staging file
lands as a `0600` target.

#### Errors

Returns an I/O error when the platform replace operation fails.

### cookie_secure_suffix

```rust
pub fn cookie_secure_suffix(public_origin: Option<&str>) -> &'static str
```

Append `; Secure` when the configured public origin is HTTPS.

Local loopback development (no `public_origin`, or `http://…`) keeps cookies
usable over plain HTTP.

### oidc_transaction_cookie_flags

```rust
pub fn oidc_transaction_cookie_flags(public_origin: Option<&str>) -> String
```

Cookie flags for the OIDC login transaction (`state` binding).

Apple's `form_post` callback is a cross-site POST, so HTTPS origins use
`SameSite=None; Secure`. Local HTTP keeps `Lax` so GET callbacks still send
the cookie on loopback.

### session_cookie_flags

```rust
pub fn session_cookie_flags(public_origin: Option<&str>) -> String
```

Build common session cookie attributes (`HttpOnly; SameSite=Lax` + optional Secure).

### resolve_d1_api_token

```rust
pub fn resolve_d1_api_token() -> Result<String>
```

Resolve the D1 API token from the environment (host-mediated secret).

Uses `BOOKCLERK_D1_API_TOKEN`, falling back to `CLOUDFLARE_API_TOKEN`.

#### Errors

Returns an error when the operation fails.

### resolve_postgres_url

```rust
pub fn resolve_postgres_url(config: &Config) -> Result<String>
```

Resolve the Postgres connection URL from config (env already applied).

`url_file` takes precedence over `url`.

#### Errors

Returns an error when the operation fails.

### graphical_session_available

```rust
pub fn graphical_session_available() -> bool
```

True when it is reasonable to start the in-process tray (windowing + session bus).

Linux requires a display (`DISPLAY` or `WAYLAND_DISPLAY`) **and** a session-bus
signal (`DBUS_SESSION_BUS_ADDRESS` or `XDG_RUNTIME_DIR`). Windows and macOS
return true; headless CI/servers on Linux return false.

### diagnostics_global

```rust
pub fn diagnostics_global() -> Option<&'static DiagnosticsHandle>
```

Accessor for the process-global diagnostics handle.

### apply_replacements

```rust
pub fn apply_replacements(input: &str, rules: &[ReplacementRule]) -> String
```

Apply replacement rules to a string (classic filename sanitization).

### classic_key_aliases

```rust
pub fn classic_key_aliases() -> HashMap<&'static str, &'static str>
```

Classic Settings.json key → dotted config path for `config get` / `config set`.

### default_replacement_characters

```rust
pub fn default_replacement_characters() -> Vec<ReplacementRule>
```

Alias for [`windows_replacement_characters`](#windows_replacement_characters) (classic default name).

### key_matches_reconcile_pattern

```rust
pub fn key_matches_reconcile_pattern(pattern: &str, key: &str) -> bool
```

True when `key` matches a reconcile `pattern` built with
[`reconciliation_wildcard_rules`](#reconciliation_wildcard_rules) (wildcard = any single character).

### parse_replacement_characters

```rust
pub fn parse_replacement_characters(value: &Value) -> Vec<ReplacementRule>
```

Parse classic `ReplacementCharacters` JSON.

Supports both the classic Libation `Settings.json` shape
(`{ "Replacement": [ { "CharacterToReplace": ":", "ReplacementString": "_" }, … ] }`)
and a flat map shape (`{ ":": "_", … }`). An empty or unrecognised value falls
back to [`windows_replacement_characters`](#windows_replacement_characters) (classic migrate parity).

### posix_replacement_characters

```rust
pub fn posix_replacement_characters() -> Vec<ReplacementRule>
```

Path-separator sanitization for POSIX local filesystems.

Keeps characters such as `:` that are legal on Linux/macOS but reserved on
Windows. `/` and `\` are still replaced so tag values cannot inject path
segments.

### reconciliation_wildcard_rules

```rust
pub fn reconciliation_wildcard_rules(explicit: &[ReplacementRule]) -> Vec<ReplacementRule>
```

Replacement rules that turn every known sanitizable character into
[`RECONCILE_WILDCARD`](#reconcile_wildcard).

Used when searching for existing liberations: creation still follows the
active profile, but reconcile must match files produced under Windows,
POSIX, or S3 rules (or custom `replacement_characters`).

### resolve_replacement_characters

```rust
pub fn resolve_replacement_characters(
    explicit: &[ReplacementRule],
    mode: PathSanitizationMode,
    storage_is_s3: bool,
) -> Vec<ReplacementRule>
```

Resolve the character-replacement map used when sanitising storage path segments.

Explicit non-empty `replacement_characters` always wins. Otherwise
`path_sanitization` (with `storage_is_s3` for [`PathSanitizationMode::Auto`](#pathsanitizationmode))
selects a profile.

### s3_replacement_characters

```rust
pub fn s3_replacement_characters() -> Vec<ReplacementRule>
```

Characters AWS recommends avoiding in S3 object keys, plus path separators.

See [Object key naming guidelines](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html).
Characters that only need URL encoding (e.g. `:`) are left intact.

### windows_replacement_characters

```rust
pub fn windows_replacement_characters() -> Vec<ReplacementRule>
```

NTFS-safe replacement map (classic Libation Windows defaults subset).

### journald_available

```rust
pub fn journald_available() -> bool
```

Back-compat alias.

### os_log_available

```rust
pub fn os_log_available() -> bool
```

Probe whether an OS log facility appears available.

### init_tracing

```rust
pub fn init_tracing(format: LogFormat, default_level: &str) -> LoggingHandle
```

Convenience wrapper matching the historical CLI/daemon call site.

### init_tracing_with

```rust
pub fn init_tracing_with(opts: TracingOptions) -> LoggingHandle
```

Install a global tracing subscriber (stderr + optional OS facility + diagnostics).

Filter precedence for stderr/OS facility: `BOOKCLERK_LOG` → `RUST_LOG` →
`opts.default_level`. The diagnostics ring buffer is **not** filtered by that
directive — it always records TRACE and above so uploads retain deep context.
Widen local verbosity with e.g. `BOOKCLERK_LOG=bookclerk=debug` when investigating.

Secrets are redacted on every sink via the public `register_secret` / `redact_str` helpers.

### strip_ansi_escapes

```rust
pub fn strip_ansi_escapes(input: &str) -> Cow<'_, str>
```

Remove ECMA-48/ANSI escape sequences so piped guest logs stay plain text.

`bookclerk-workerd` and other guests may emit colored `tracing_subscriber`
lines on stderr. The host re-logs those lines into JSON; leaving CSI codes
in place produces `\u001b[…` noise in `daemon.json_logs`.

### allowlist_permits

```rust
pub fn allowlist_permits(
    email: Option<&str>,
    sub: &str,
    allowed_emails: &[String],
    allowed_domains: &[String],
    allowed_subjects: &[String],
    global_domains: &[String],
) -> bool
```

Whether an allowlist provider admits this `(email, sub)`.

Empty email, domain, and subject lists fail closed. When the provider lists
are empty, `global_domains` is used as the domain allowlist.

### email_domain_allowed

```rust
pub fn email_domain_allowed(email: Option<&str>, domains: &[String]) -> bool
```

Whether `email` matches an allowlist of domains (empty list = unrestricted).

### oidc_apple_private_key_env_key

```rust
pub fn oidc_apple_private_key_env_key(provider_id: &str) -> String
```

Environment variable holding a Sign in with Apple `.p8` private key.

### oidc_client_secret_env_key

```rust
pub fn oidc_client_secret_env_key(provider_id: &str) -> String
```

Environment variable holding a provider's client secret.

Hyphens in `provider_id` become underscores. Example: `github` →
`BOOKCLERK_OIDC_GITHUB_CLIENT_SECRET`.

### oidc_secret_store_name

```rust
pub fn oidc_secret_store_name(logical: &str, generation: u64) -> String
```

`encrypted_secrets.name` for an OIDC client secret or Apple key.

Generation `0` is the unversioned legacy name; later generations use
`name@generation` so a crashed PUT cannot make new credentials visible
under the still-published config.

### resolve_mapped_role

```rust
pub fn resolve_mapped_role(
    role_map: &BTreeMap<String, String>,
    claim_values: &[String],
) -> Option<String>
```

Pick the highest Bookclerk role from mapped claim values.

Precedence: owner > administrator > host > member.

### generate_operator_token

```rust
pub fn generate_operator_token() -> Result<String>
```

Generate a 32-byte hex operator token.

#### Errors

Returns an error when the operation fails.

### read_operator_token_env

```rust
pub fn read_operator_token_env() -> Result<Option<(String, ResolveOperatorTokenEnv)>>
```

Read `BOOKCLERK_OPERATOR_TOKEN` when set and non-empty.

#### Errors

Returns an error when the operation fails.

### validate_operator_token

```rust
pub fn validate_operator_token(token: &str, source: &str) -> Result<String>
```

Reject tokens that can break URL fragments, HTTP headers, or shell pastes.

Generated tokens are hex. Env overrides must stay printable single-line and
free of whitespace / control characters so they cannot inject into
`Authorization` headers.

#### Errors

Returns an error when the operation fails.

### normalize_storage_prefix

```rust
pub fn normalize_storage_prefix(prefix: &str) -> String
```

Normalize a storage key prefix: empty stays empty; otherwise ensure a trailing `/`.

### apply_config_updates

```rust
pub fn apply_config_updates(
    files_dir: Option<PathBuf>,
    config_path: Option<PathBuf>,
    pairs: &[(&str, &str)],
) -> Result<Config>
```

Load config, apply `pairs`, and write the result back to the resolved config path.

#### Arguments

* `files_dir` - Optional `BOOKCLERK_FILES_DIR` override.
* `config_path` - Optional explicit `config.toml` path.
* `pairs` - `(key, value)` overrides (classic or dotted).

#### Returns

The updated [`Config`](#config) after a successful write.

#### Errors

Propagates load, validation, and write failures as [`crate::ConfigError`](#configerror).

### apply_config_updates_from_path

```rust
pub fn apply_config_updates_from_path(
    path: &Path,
    pairs: &[(&str, &str)],
) -> Result<Config>
```

Load `path`, apply `pairs`, and write the updated TOML back to the same path.

#### Arguments

* `path` - Existing `config.toml` to mutate in place.
* `pairs` - `(key, value)` overrides (classic or dotted).

#### Errors

Propagates read, parse, and write failures as [`crate::ConfigError`](#configerror).

### apply_setting_overrides

```rust
pub fn apply_setting_overrides(config: &mut Config, pairs: &[(&str, &str)])
```

Apply classic-style `-o Setting=value` overrides onto `config`.

Keys may be classic PascalCase (`UseWidevine`) or dotted TOML paths
(`output.widevine`). Plugin settings are not `config.toml` keys; they are
written on the plugin instance (`bookclerk plugins config set`).

### enforce_storage_key_limits

```rust
pub fn enforce_storage_key_limits(key: &str, limits: PathLimits) -> String
```

Enforce per-segment and full-key limits on a relative storage key.

`key` uses `/` separators. The final component is treated as a filename
(extension preserved). Returns the possibly truncated key.

### path_len

```rust
pub fn path_len(s: &str, measure: PathLengthMeasure) -> usize
```

Measure `s` according to `measure`.

### truncate_filename_stem

```rust
pub fn truncate_filename_stem(
    stem: &str,
    ext: &str,
    max_filename_length: usize,
    measure: PathLengthMeasure,
) -> String
```

Truncate a filename stem so `stem + "." + ext` fits in `max_filename_length`.

When the stem ends with ` [id]` (ASIN/ISBN bracket form used by naming
profiles), the id suffix is preserved and the title prefix is shortened.

### truncate_path_component

```rust
pub fn truncate_path_component(
    s: &str,
    limit: usize,
    measure: PathLengthMeasure,
) -> String
```

Truncate `s` so its measured length is ≤ `limit`. Never splits a UTF-8 char.

### resolve_config_path

```rust
pub fn resolve_config_path(files_dir: &Path) -> PathBuf
```

Resolve config file path: `BOOKCLERK_CONFIG` → `{files_dir}/config.toml`.

### resolve_files_dir

```rust
pub fn resolve_files_dir(cli_override: Option<PathBuf>) -> PathBuf
```

Resolve the Bookclerk files directory.

Precedence:
1. Explicit `--bookclerk-files` / caller-provided path
2. `BOOKCLERK_FILES_DIR`
3. XDG data dir: `$XDG_DATA_HOME/bookclerk` (or platform equivalent)
4. Fallback: `./BookclerkFiles` (cwd-relative, Docker-friendly)

### detect_distro

```rust
pub fn detect_distro() -> Option<String>
```

Best-effort OS distribution string (e.g. `Ubuntu 24.04.1 LTS`).

### host_cpu_rate_max

```rust
pub fn host_cpu_rate_max() -> u32
```

Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`).

### node_logical_cpus

```rust
pub fn node_logical_cpus() -> u32
```

Logical CPUs visible to this process (at least 1).

### contains_registered_secret

```rust
pub fn contains_registered_secret(haystack: &str) -> bool
```

True if `haystack` still contains any registered exact secret (post-redaction check).

### is_sensitive_field

```rust
pub fn is_sensitive_field(name: &str) -> bool
```

Returns true when a tracing field name should never be logged in cleartext.

### is_upload_identifying_field

```rust
pub fn is_upload_identifying_field(name: &str) -> bool
```

Field names scrubbed from **remote** diagnostics uploads
in addition to [`is_sensitive_field`](#is_sensitive_field). Local journal/stderr may still show them.

### redact_field_value

```rust
pub fn redact_field_value(name: &str, value: &str) -> String
```

Redact a field value: sensitive names are fully replaced; others are scrubbed.

### redact_str

```rust
pub fn redact_str(input: &str) -> String
```

Scrub known secrets: exact registered values, then pattern matches.

### register_secret

```rust
pub fn register_secret(value: impl AsRef<str>)
```

Register a known secret so it is always scrubbed from logs and diagnostics.

Call this whenever a passphrase, OAuth token, AWS key, etc. enters process
memory. Values shorter than 6 characters are ignored.

Also registers common encodings (percent-encoding, `+` for spaces) so a secret
cannot slip through via URL/query embedding.

### register_secrets

```rust
pub fn register_secrets<I, S>(values: I)
where
    I: IntoIterator<Item = S>,
    S: AsRef<str>,
```

Register several secrets at once.

### register_secrets_from_env

```rust
pub fn register_secrets_from_env()
```

Register secrets commonly present in the process environment / config.

Safe to call multiple times. Does not log values.

### sanitize_for_remote_upload

```rust
pub fn sanitize_for_remote_upload(name: &str, value: &str) -> String
```

Extra sanitization for payloads that leave the machine (collector / B2).

Emails are **partially** masked (local-part truncated) so triage can still see
rough account shape without shipping a clear address. Other identifying
fields (title, path, account, …) remain fully [`REDACTED`](#redacted).

### truncate_upload_message

```rust
pub fn truncate_upload_message(message: &str) -> String
```

Cap message length used in remote diagnostics events.

### unregister_secret

```rust
pub fn unregister_secret(value: impl AsRef<str>)
```

Forget a secret [`register_secret`](#register_secret) registered, with its encoded forms,
once nothing in the process holds it any more (a short-lived bearer
token whose holder was dropped), so per-request credentials do not pile
up in the registry every log line is checked against. Lines captured
while it was registered were scrubbed then. Only for values unique to
one holder: the registry keeps no count, so a value two holders share
would stop being scrubbed for both.

### read_node_api_spki_pin

```rust
pub fn read_node_api_spki_pin(files_dir: &Path) -> Option<String>
```

The SPKI pin of the daemon's self-signed node API certificate, once the
daemon has generated it. Enrollment links carry it in
`[daemon.node_api] mode = "self_signed"`.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, ConfigError>;
```

Result alias for configuration load and validation helpers in this crate.

## Constants

### RECONCILE_WILDCARD

```rust
pub const RECONCILE_WILDCARD: char = '\u{E000}';
```

Sentinel placed in reconcile path *patterns* where a sanitizable character
appeared in metadata. Matches any single character in an on-disk key.

### DEFAULT_MAX_FILENAME_LENGTH

```rust
pub const DEFAULT_MAX_FILENAME_LENGTH: usize = 255;
```

Classic Libation `LongPath.MaxFilenameLength` — per path component.

### S3_MAX_OBJECT_KEY_BYTES

```rust
pub const S3_MAX_OBJECT_KEY_BYTES: usize = 1024;
```

AWS S3 maximum object key length (UTF-8 bytes), including any prefix.

### BOOKCLERK_CONFIG_ENV

```rust
pub const BOOKCLERK_CONFIG_ENV: &str = "BOOKCLERK_CONFIG";
```

Override for the config file path (`BOOKCLERK_CONFIG`).

### BOOKCLERK_FILES_DIR_ENV

```rust
pub const BOOKCLERK_FILES_DIR_ENV: &str = "BOOKCLERK_FILES_DIR";
```

Environment variable for the Bookclerk data directory.

### REDACTED

```rust
pub const REDACTED: &str = "[REDACTED]";
```

Replacement token used wherever a secret or sensitive value is scrubbed.

### LETS_ENCRYPT_DIRECTORY

```rust
pub const LETS_ENCRYPT_DIRECTORY: &str = "https://acme-v02.api.letsencrypt.org/directory";
```

Let's Encrypt's production ACME directory.

### NODE_API_ACME_DIR

```rust
pub const NODE_API_ACME_DIR: &str = "acme";
```

Directory under `$BOOKCLERK_FILES_DIR` holding the ACME account and
certificate cache (mode 0700 on Unix).

### NODE_API_SELF_SIGNED_DIR

```rust
pub const NODE_API_SELF_SIGNED_DIR: &str = "node-api/self-signed";
```

Directory under `$BOOKCLERK_FILES_DIR` holding the self-signed node API
certificate, its key, and its SPKI pin (mode 0700 on Unix).

### NODE_API_SPKI_PIN_FILE

```rust
pub const NODE_API_SPKI_PIN_FILE: &str = "spki-pin";
```

File in [`NODE_API_SELF_SIGNED_DIR`](#node_api_self_signed_dir) holding the SPKI pin
(base64url SHA-256 of the certificate's SubjectPublicKeyInfo).

### TRAY_HANDOFF_TTL_SECS_DEFAULT

```rust
pub const TRAY_HANDOFF_TTL_SECS_DEFAULT: u64 = 180;
```

Default tray/CLI loopback sign-in link lifetime (seconds).

### TRAY_HANDOFF_TTL_SECS_MAX

```rust
pub const TRAY_HANDOFF_TTL_SECS_MAX: u64 = 900;
```

Maximum allowed [`DaemonAuthConfig::tray_handoff_ttl_secs`](#daemonauthconfig).

### TRAY_HANDOFF_TTL_SECS_MIN

```rust
pub const TRAY_HANDOFF_TTL_SECS_MIN: u64 = 30;
```

Minimum allowed [`DaemonAuthConfig::tray_handoff_ttl_secs`](#daemonauthconfig).
