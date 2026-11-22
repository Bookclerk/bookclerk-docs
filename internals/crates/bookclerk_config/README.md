# bookclerk_config

Host configuration, files-dir layout, and logging for Bookclerk.

## Audience

Host binaries (`bookclerk`, `bookclerkd`) and in-process host crates — not
guest plugins. Parses `config.toml`, applies env overrides, resolves
`BOOKCLERK_FILES_DIR`, and installs tracing / diagnostics.

Product narrative: `docs/server/configuration.md`, `docs/server/diagnostics.md`. Style:
`docs/contributing/code-documentation.md`.

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
| [`CheckIn`](#checkin) | struct | Body of `POST /checkin`. |
| [`InstallId`](#installid) | struct | A random install ID: a version-4 UUID in lowercase hyphenated form. |
| [`JobsConcurrencyConfig`](#jobsconcurrencyconfig) | struct | Per-resource-class worker counts and library-wide job semaphores. |
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
| [`DestinationId`](#destinationid) | struct | Id of one output destination: the plugin instance id of the deployed storage plugin instance that serves it. |
| [`DestinationNaming`](#destinationnaming) | struct | Optional naming overrides for one destination plugin. |
| [`OutputConfig`](#outputconfig) | struct | Acquire packaging (`[output]`). |
| [`CopyError`](#copyerror) | struct | Why a copy list was refused. |
| [`CopyQuality`](#copyquality) | struct | The quality of one copy, by name. |
| [`OpusSettings`](#opussettings) | struct | The Opus encoder settings one output is written with. |
| [`OutputCopy`](#outputcopy) | struct | One copy of every acquired book: a format at a quality. |
| [`PathLimits`](#pathlimits) | struct | Limits applied when hardening acquired storage keys. |
| [`Paths`](#paths) | struct | Resolved filesystem locations for Bookclerk data. |
| [`DestinationFilter`](#destinationfilter) | struct | Which books one destination takes. |
| [`FilterError`](#filtererror) | struct | Why a `DestinationFilter` was refused. |
| [`FilterRule`](#filterrule) | struct | One rule of a destination filter: a field, one value of it, and its effect. |
| [`PlacementFacts`](#placementfacts) | struct | What a destination filter knows about one book when an acquire starts. |
| [`NodeLimits`](#nodelimits) | struct | The CPU and memory a node may use, as its cgroup limits them. |
| [`IntegrationsConfig`](#integrationsconfig) | struct | Connect-portal settings under `[integrations]`. |
| [`PluginRegistryEntry`](#pluginregistryentry) | struct | One configured plugin registry source (`[[plugins.registries]]`). |
| [`PluginsConfig`](#pluginsconfig) | struct | `[plugins]` — how external plugin guests are run. |
| [`PluginsJailConfig`](#pluginsjailconfig) | struct | Optional global resource ceilings for plugin guest jails under `[plugins.jail]`. |
| [`RelayedLine`](#relayedline) | struct | One relayed line: its level and its text without the timestamp and level. |
| [`AccessCode`](#accesscode) | struct | A report's access code: the credential that reads the report back from the collector or deletes it. Canonical form: exactly `ACCESS_CODE_LENGTH` characters of `ACCESS_CODE_ALPHABET`, no separators. `Debug` never prints it. |
| [`ReportId`](#reportid) | struct | A report ID the collector assigned: a version-4 UUID in lowercase hyphenated form. |
| [`ReportRecord`](#reportrecord) | struct | One report this install sent. |
| [`SubmitReply`](#submitreply) | struct | What the collector's `POST /submit` reply names. |
| [`LookupAnswer`](#lookupanswer) | struct | The collector's answer to a lookup. |
| [`LookupEvent`](#lookupevent) | struct | One logged event in a stored report, already redacted by the install that sent it. |
| [`LookupFix`](#lookupfix) | struct | The fix recorded for the report's tracking issue. |
| [`LookupIssue`](#lookupissue) | struct | A GitHub issue that tracks the report. |
| [`LookupReport`](#lookupreport) | struct | A stored report as the collector returns it. |
| [`AuthConfig`](#authconfig) | struct | Auth encryption settings (`[auth]` section). |
| [`Config`](#config) | struct | Top-level Bookclerk configuration. |
| [`DaemonAuthConfig`](#daemonauthconfig) | struct | Operator token / session settings for the daemon HTTP API. |
| [`DaemonConfig`](#daemonconfig) | struct | Daemon / HTTP control plane settings. |
| [`DiagnosticsConfig`](#diagnosticsconfig) | struct | Opt-in sharing of recent **redacted** logs on crash or error bursts. |
| [`DiscoveryConfig`](#discoveryconfig) | struct | Discovery / recommendations / embeddings settings. |
| [`LibraryConfig`](#libraryconfig) | struct | Library / scan related settings. |
| [`NodeApiConfig`](#nodeapiconfig) | struct | `[daemon.node_api]`: transport policy for branch nodes. |
| [`ProcessEnvGuard`](#processenvguard) | struct | Process-wide lock around `BOOKCLERK_*` environment reads and test writes. |
| [`UploadsConfig`](#uploadsconfig) | struct | `[uploads]`: per-node upload limits. |
| [`CacheSettingError`](#cachesettingerror) | enum | Why a `cache.*` value was refused. |
| [`CacheSettingKey`](#cachesettingkey) | enum | A `cache.*` settings key. |
| [`DatabasePluginKind`](#databasepluginkind) | enum | Which first-party database plugin is active. |
| [`ProcessRole`](#processrole) | enum | Which kind of Bookclerk process installed the subscriber. |
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
| [`MissingFileAction`](#missingfileaction) | enum | What happens to a stored copy whose file disappears from its destination outside Bookclerk (noticed by the destination's change feed or listing). |
| [`MultiDestinationMode`](#multidestinationmode) | enum | Behavior when some output destinations already have a title and others do not. |
| [`CopyFormat`](#copyformat) | enum | The format of one copy of a book. |
| [`PathLengthMeasure`](#pathlengthmeasure) | enum | How to measure path component length when truncating. |
| [`ChapterJsonMode`](#chapterjsonmode) | enum | Whether / which chapter JSON sidecars to write beside acquired audio. |
| [`OutputFormat`](#outputformat) | enum | Post-download output formatting. Default is enriched M4B (tags + chapters). `split_mp3_by_size` parses but is not implemented yet: the acquire pipeline rejects a job that selects it before downloading. |
| [`FilterEffect`](#filtereffect) | enum | Whether a `FilterRule` lets matching books in or keeps them out. |
| [`FilterField`](#filterfield) | enum | The book field a `FilterRule` matches one value of. |
| [`PlacementMediaKind`](#placementmediakind) | enum | What a book is, as a destination filter sees it. |
| [`WebhooksExposure`](#webhooksexposure) | enum | `plugins.webhooks`: whether the public webhook route is open. |
| [`RelayedLevel`](#relayedlevel) | enum | The level a relayed line is logged at. |
| [`SubmitReplyError`](#submitreplyerror) | enum | Why a collector reply was refused. |
| [`EventLevel`](#eventlevel) | enum | Severity of one logged event in a report. |
| [`IssueState`](#issuestate) | enum | Whether a GitHub issue is open. |
| [`LookupError`](#lookuperror) | enum | Why a lookup or delete did not succeed. |
| [`LookupReplyError`](#lookupreplyerror) | enum | Why a collector answer was refused. |
| [`AudioQuality`](#audioquality) | enum | Audible download bitrate tier (`high` / `normal` in TOML). |
| [`KeyStoreKind`](#keystorekind) | enum | Where an install keeps the library's data-encryption key. |
| [`NodeApiMode`](#nodeapimode) | enum | How branch nodes may reach the node API (`[daemon.node_api] mode`). |
| [`replace_file`](#replace_file) | function | Replace `to` with `from`, both in the same directory. `to` may already exist; on Windows this goes through `MoveFileExW` with `MOVEFILE_REPLACE_EXISTING \| MOVEFILE_WRITE_THROUGH` (see the module docs), retrying briefly while another replace of `to` settles. The moved file keeps its own permissions, so a `0600` staging file lands as a `0600` target. |
| [`parse_cache_setting`](#parse_cache_setting) | function | Parses one `cache.*` setting. |
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
| [`resolve_replacement_characters`](#resolve_replacement_characters) | function | Resolve the character-replacement map the host applies to tag values when it fills a naming template. |
| [`s3_replacement_characters`](#s3_replacement_characters) | function | Characters AWS recommends avoiding in S3 object keys, plus path separators. |
| [`windows_replacement_characters`](#windows_replacement_characters) | function | NTFS-safe replacement map (classic Libation Windows defaults subset). |
| [`checkin_due`](#checkin_due) | function | Whether a check-in is due. |
| [`record_checkin`](#record_checkin) | function | Records an accepted check-in. |
| [`send_checkin`](#send_checkin) | function | Sends one check-in to the collector. |
| [`journald_available`](#journald_available) | function | Back-compat alias. |
| [`os_log_available`](#os_log_available) | function | Probe whether an OS log facility appears available. |
| [`init_tracing`](#init_tracing) | function | Convenience wrapper matching the historical CLI/daemon call site. |
| [`init_tracing_with`](#init_tracing_with) | function | Install a global tracing subscriber (stderr + optional OS facility + diagnostics). |
| [`ring_filter`](#ring_filter) | function | The diagnostics ring's per-layer filter: TRACE for `RING_TRACE_TARGETS` (matched as prefixes, so `bookclerkd` and `bookclerk_library` count), `RING_DEPENDENCY_LEVEL` for the rest. |
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
| [`copies_to_make`](#copies_to_make) | function | Every copy an acquire makes, in order: the main output's copy first (when the pipeline can make it), then each valid extra copy that differs from every earlier one, at most `MAX_EXTRA_COPIES` of them whatever the list holds. |
| [`mp3_cbr_kbps`](#mp3_cbr_kbps) | function | The constant bitrate, in kbit/s, the MP3 encoder writes for a requested `kbps`: the smallest of `MP3_CBR_KBPS` at or above it, else 320. |
| [`validate_copies`](#validate_copies) | function | Checks a list of extra copies: at most `MAX_EXTRA_COPIES`, each valid (`OutputCopy::validate`), none given twice and none the main output's copy `main`. |
| [`apply_config_updates`](#apply_config_updates) | function | Load config, apply `pairs`, and write the result back to the resolved config path. |
| [`apply_config_updates_from_path`](#apply_config_updates_from_path) | function | Load `path`, apply `pairs`, and write the updated TOML back to the same path. |
| [`apply_setting_overrides`](#apply_setting_overrides) | function | Apply classic-style `-o Setting=value` overrides onto `config`. |
| [`enforce_storage_key_limits`](#enforce_storage_key_limits) | function | Enforce per-segment and full-key limits on a relative storage key. |
| [`path_len`](#path_len) | function | Measure `s` according to `measure`. |
| [`truncate_filename_stem`](#truncate_filename_stem) | function | Truncate a filename stem so `stem + "." + ext` fits in `max_filename_length`. |
| [`truncate_path_component`](#truncate_path_component) | function | Truncate `s` so its measured length is ≤ `limit`. Never splits a UTF-8 char. |
| [`resolve_config_path`](#resolve_config_path) | function | Resolve config file path: `BOOKCLERK_CONFIG` → `{files_dir}/config.toml`. |
| [`resolve_files_dir`](#resolve_files_dir) | function | Resolve the Bookclerk files directory. |
| [`parse_destination_filter`](#parse_destination_filter) | function | Parses a destination filter from its raw JSON bytes, sans IO, the way the web API reads it: the strict JSON shape, then trimmed (`DestinationFilter::normalized`) and checked (`DestinationFilter::validate`). |
| [`detect_distro`](#detect_distro) | function | Best-effort OS distribution string (e.g. `Ubuntu 24.04.1 LTS`). |
| [`node_limits`](#node_limits) | function | This process's cgroup v2 CPU and memory limits, or `None` when it has neither (or the platform has no cgroups). |
| [`host_cpu_rate_max`](#host_cpu_rate_max) | function | Host CPU rate ceiling in one-core percent units (`logical_cpus × 100`). |
| [`node_logical_cpus`](#node_logical_cpus) | function | Logical CPUs visible to this process (at least 1). |
| [`contains_registered_secret`](#contains_registered_secret) | function | True if `haystack` still contains any registered exact secret (post-redaction check). |
| [`is_sensitive_field`](#is_sensitive_field) | function | Returns true when a tracing field name should never be logged in cleartext. |
| [`is_upload_identifying_field`](#is_upload_identifying_field) | function | Field names scrubbed from **remote** diagnostics uploads in addition to `is_sensitive_field`. Local journal/stderr may still show them. |
| [`redact_field_value`](#redact_field_value) | function | Redact a field value: sensitive names are fully replaced; others are scrubbed. |
| [`redact_str`](#redact_str) | function | Scrub known secrets: exact registered values, then pattern matches. |
| [`register_secret`](#register_secret) | function | Register a known secret so it is always scrubbed from logs and diagnostics. |
| [`register_secret_bytes`](#register_secret_bytes) | function | Registers one secret value given as bytes, when it is UTF-8 text. |
| [`register_secrets`](#register_secrets) | function | Register several secrets at once. |
| [`register_secrets_from_env`](#register_secrets_from_env) | function | Register secrets commonly present in the process environment / config. |
| [`sanitize_for_remote_upload`](#sanitize_for_remote_upload) | function | Extra sanitization for payloads that leave the machine (collector / B2). |
| [`truncate_upload_message`](#truncate_upload_message) | function | Cap message length used in remote diagnostics events. |
| [`unregister_secret`](#unregister_secret) | function | Forget a secret `register_secret` registered, with its encoded forms, once nothing in the process holds it any more (a short-lived bearer token whose holder was dropped), so per-request credentials do not pile up in the registry every log line is checked against. Lines captured while it was registered were scrubbed then. Only for values unique to one holder: the registry keeps no count, so a value two holders share would stop being scrubbed for both. |
| [`parse_relayed_line`](#parse_relayed_line) | function | Splits a tracing-format prefix (optional timestamp, then a level word) off a relayed stderr line. |
| [`find_report`](#find_report) | function | The report with this ID in the history, if it is there. |
| [`load_report_history`](#load_report_history) | function | Reads the report history, oldest first. |
| [`mark_report_deleted`](#mark_report_deleted) | function | Notes in the history that the collector's copy of a report was deleted. |
| [`parse_submit_reply`](#parse_submit_reply) | function | Reads the report ID and access code from the collector's `POST /submit` reply. |
| [`record_report`](#record_report) | function | Adds a report to the history, keeping the newest `MAX_REPORT_HISTORY`. |
| [`delete_report`](#delete_report) | function | Deletes the collector's stored copy of a sent report. Its one-line index entry and issue links stay on the collector. |
| [`fetch_report`](#fetch_report) | function | Reads a sent report back from the collector. |
| [`parse_delete_reply`](#parse_delete_reply) | function | Reads the collector's answer to `POST /lookup/delete`. |
| [`parse_lookup_reply`](#parse_lookup_reply) | function | Reads the collector's answer to `POST /lookup`. |
| [`read_node_api_spki_pin`](#read_node_api_spki_pin) | function | The SPKI pin of the daemon's self-signed node API certificate, once the daemon has generated it. Enrollment links carry it in `[daemon.node_api] mode = "self_signed"`. |
| [`Result`](#result) | type alias | Result alias for configuration load and validation helpers in this crate. |
| [`DEFAULT_CACHE_BUDGET_BYTES`](#default_cache_budget_bytes) | constant | Default disk budget for one node's image caches: 1 GiB. |
| [`DEFAULT_CACHE_UNVIEWED_DAYS`](#default_cache_unviewed_days) | constant | Default days an entry may go unviewed before it is evicted. |
| [`MAX_CACHE_BUDGET_BYTES`](#max_cache_budget_bytes) | constant | Largest accepted cache budget: 1 TiB. |
| [`MAX_CACHE_UNVIEWED_DAYS`](#max_cache_unviewed_days) | constant | Largest accepted `unviewed_days`, about ten years. |
| [`MIN_CACHE_BUDGET_BYTES`](#min_cache_budget_bytes) | constant | Smallest accepted cache budget: 16 MiB, room for a few hundred covers. |
| [`SLOW_TARGET`](#slow_target) | constant | Target of the events that mark a slow response. |
| [`RECONCILE_WILDCARD`](#reconcile_wildcard) | constant | Sentinel placed in reconcile path *patterns* where a sanitizable character appeared in metadata. Matches any single character in an on-disk key. |
| [`CHECKIN_INTERVAL_MS`](#checkin_interval_ms) | constant | Time between check-ins: one week, in milliseconds. |
| [`RING_DEPENDENCY_LEVEL`](#ring_dependency_level) | constant | Level the diagnostics ring keeps for targets outside `RING_TRACE_TARGETS`. |
| [`RING_TRACE_TARGETS`](#ring_trace_targets) | constant | Target prefixes the diagnostics ring keeps through TRACE. Every other target (dependencies such as `h2`, `hyper`, `rustls`, `reqwest`, and `log`-crate records) is kept from INFO up, so their per-frame TRACE events do not crowd Bookclerk's own events out of the ring. |
| [`MAX_EXTRA_COPIES`](#max_extra_copies) | constant | Most copies a library may keep beside its main output. |
| [`MAX_QUALITY_BYTES`](#max_quality_bytes) | constant | Longest quality name, in bytes (`320k-mono` is nine). |
| [`MP3_CBR_KBPS`](#mp3_cbr_kbps-1) | constant | The constant bitrates, in kbit/s, the MP3 encoder writes. |
| [`OPUS_KBPS`](#opus_kbps) | constant | The bitrates, in kbit/s, an Opus copy is named by. Opus encodes in variable bitrate around the target, so these are targets, not caps. |
| [`OPUS_MAIN_KBPS`](#opus_main_kbps) | constant | The quality of the main output when it is Opus (`format = "opus"`): 64 kbit/s, keeping the source's channels up to stereo. Another bitrate takes an Opus extra copy beside an M4B main output. |
| [`DEFAULT_MAX_FILENAME_LENGTH`](#default_max_filename_length) | constant | Classic Libation `LongPath.MaxFilenameLength` — per path component. |
| [`MAX_HOST_KEY_BYTES`](#max_host_key_bytes) | constant | Largest host key, in UTF-8 bytes, including the destination's prefix: the S3 object-key limit, which file systems also accept. |
| [`BOOKCLERK_CONFIG_ENV`](#bookclerk_config_env) | constant | Override for the config file path (`BOOKCLERK_CONFIG`). |
| [`BOOKCLERK_FILES_DIR_ENV`](#bookclerk_files_dir_env) | constant | Environment variable for the Bookclerk data directory. |
| [`MAX_FILTER_RULES`](#max_filter_rules) | constant | Most rules one destination filter may hold. |
| [`MAX_FILTER_VALUE_BYTES`](#max_filter_value_bytes) | constant | Longest rule value, in UTF-8 bytes. |
| [`REDACTED`](#redacted) | constant | Replacement token used wherever a secret or sensitive value is scrubbed. |
| [`ACCESS_CODE_ALPHABET`](#access_code_alphabet) | constant | Crockford's base32 alphabet, which access codes are written in: digits and letters without I, L, O and U. |
| [`ACCESS_CODE_LENGTH`](#access_code_length) | constant | Characters in an access code, without the group separators. |
| [`MAX_REPORT_HISTORY`](#max_report_history) | constant | How many reports the history keeps; older ones are dropped first. |
| [`MAX_SUBMIT_REPLY_BYTES`](#max_submit_reply_bytes) | constant | Longest collector reply, in bytes, that `parse_submit_reply` reads. The real reply is about 200 bytes. |
| [`REPORT_HISTORY_FILE`](#report_history_file) | constant | File, relative to the files directory, that holds the report history. |
| [`MAX_LOOKUP_REPLY_BYTES`](#max_lookup_reply_bytes) | constant | Longest collector answer, in bytes, that `parse_lookup_reply` and `parse_delete_reply` read. A stored report is at most 256 KiB, and the answer caps every event, so a real answer is well under this. |
| [`DEFAULT_UPGRADE_DAILY_LIMIT`](#default_upgrade_daily_limit) | constant | Default for `LibraryConfig::upgrade_daily_limit`. |
| [`LETS_ENCRYPT_DIRECTORY`](#lets_encrypt_directory) | constant | Let's Encrypt's production ACME directory. |
| [`MAX_UPGRADE_DAILY_LIMIT`](#max_upgrade_daily_limit) | constant | Highest `LibraryConfig::upgrade_daily_limit` the settings accept. |
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

#### DiagnosticsHandle::new

```rust
pub fn new(
    config: DiagnosticsConfig,
    version: impl Into<String>,
    role: ProcessRole,
) -> Self
```

A new handle with ring capacity from config (at least 1). The
process installs one with `install_global`; tests build their own.

#### DiagnosticsHandle::upload_enabled

```rust
pub fn upload_enabled(&self) -> bool
```

Whether automatic upload is configured and ready (token/url present).

#### DiagnosticsHandle::share_usage

```rust
pub fn share_usage(&self) -> bool
```

Whether the weekly usage check-in and the install ID are on right now
(`diagnostics.share_usage`, following Settings changes).

#### DiagnosticsHandle::set_sharing

```rust
pub fn set_sharing(&self, share_reports: bool, share_usage: bool)
```

Applies new `diagnostics.share_reports` / `share_usage` values without
a restart (the daemon calls it after a config reload).

##### Arguments

* `share_reports` - Whether reports are uploaded from now on.
* `share_usage` - Whether uploads carry the install ID and the
  check-in runs from now on.

#### DiagnosticsHandle::set_files_dir

```rust
pub fn set_files_dir(&self, files_dir: &Path)
```

Tells the handle where the files directory is, so accepted reports
are recorded in the install's report history. The first call wins.

##### Arguments

* `files_dir` - The Bookclerk files directory.

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

#### DiagnosticsHandle::set_install_id

```rust
pub fn set_install_id(&self, id: &InstallId)
```

Attaches the install's random ID to later uploads. Does nothing unless
`diagnostics.share_usage` is on; the first ID set stays.

##### Arguments

* `id` - The install ID ([`crate::InstallId`](#installid)).

#### DiagnosticsHandle::deployment

```rust
pub fn deployment(&self) -> &'static str
```

What uploads report as `deployment`: `demo` for a copy of the hosted
demo, otherwise the process role.

#### DiagnosticsHandle::uploads_attempted

```rust
pub fn uploads_attempted(&self) -> u64
```

Number of upload attempts since init (successful or failed HTTP tries).

#### DiagnosticsHandle::record_event

```rust
pub fn record_event(&self, event: &Event<'_>)
```

Record a redacted event into the ring buffer and maybe trigger a burst upload.

#### DiagnosticsHandle::record_event_with_spans

```rust
pub fn record_event_with_spans(
    &self,
    event: &Event<'_>,
    span_fields: Vec<(String, String)>,
)
```

Record a redacted event with fields copied from the spans it happened
in, and maybe trigger a burst upload.

`log`-crate records take their own target (`log.target`) and lose the
fields that describe the record itself (module path, file, line).

##### Arguments

* `event` - The tracing event.
* `span_fields` - Redacted `(span.field, value)` pairs, innermost span
  first (see `DiagnosticsLayer`).

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
    pub deployment: String,
    pub install_id: Option<String>,
    pub version: String,
    pub os: String,
    pub arch: String,
    pub distro: Option<String>,
    pub build_commit: String,
    pub limits: Option<NodeLimits>,
    pub rustc_release: String,
    pub rustc_channel: String,
    pub archived_at_unix_ms: u64,
    pub events: Vec<BufferedEvent>,
}
```

JSON body POSTed to the diagnostics collector `/submit` endpoint.

**Fields**

- `trigger`: Why this upload was produced (`crash`, `error_burst`, `slow_burst`,
  `cli_error`, …).
- `deployment`: What sent it: `daemon`, `desktop`, `cli`, or `demo` for a copy of the
  hosted demo.
- `install_id`: The install's random ID, sent only when `diagnostics.share_usage` is
  on. The collector keeps only a monthly-salted hash of it.
- `version`: Bookclerk version string recorded at subscriber install.
- `os`: Host OS (`linux`, `macos`, `windows`, …).
- `arch`: CPU architecture (`x86_64`, `aarch64`, …).
- `distro`: Best-effort Linux distro id when detectable.
- `build_commit`: The source commit this build was made from (or `"unknown"`).
- `limits`: The CPU and memory limits of the process's cgroup when it has any
  (a container or a systemd slice), so a slow report from a small
  node says so.
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

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### CheckIn

```rust
pub struct CheckIn {
    pub install_id: String,
    pub version: String,
    pub os: String,
    pub arch: String,
    pub kind: String,
}
```

Body of `POST /checkin`.

**Fields**

- `install_id`: The install's random ID.
- `version`: Bookclerk version.
- `os`: Operating system (`linux`, `macos`, `windows`, …).
- `arch`: CPU architecture (`x86_64`, `aarch64`, …).
- `kind`: Kind of process: `daemon` or `desktop`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### InstallId

```rust
pub struct InstallId(/* private */);
```

A random install ID: a version-4 UUID in lowercase hyphenated form.

#### InstallId::generate

```rust
pub fn generate() -> Result<Self>
```

Generates a new random ID from the operating system's random source.

##### Errors

Returns an error when the random source is unavailable.

#### InstallId::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses an ID as [`InstallId::generate`](#installidgenerate) writes it.

##### Arguments

* `text` - Candidate text; surrounding whitespace is ignored.

##### Returns

The ID, or `None` unless the text is a lowercase hyphenated version-4
UUID.

#### InstallId::as_str

```rust
pub fn as_str(&self) -> &str
```

The ID as text.

#### InstallId::load_or_create

```rust
pub fn load_or_create(files_dir: &Path) -> Result<Self>
```

Reads the ID from the files directory, creating it on first use (or
when the file does not hold a valid ID).

##### Arguments

* `files_dir` - The Bookclerk files directory.

##### Errors

Returns an error when the file cannot be read or written.

#### InstallId::reset

```rust
pub fn reset(files_dir: &Path) -> Result<Self>
```

Replaces the ID with a new random one and forgets the last check-in,
so the new ID checks in on the next pass.

##### Arguments

* `files_dir` - The Bookclerk files directory.

##### Errors

Returns an error when the file cannot be written.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### JobsConcurrencyConfig

```rust
pub struct JobsConcurrencyConfig {
    pub network: u32,
    pub plugin_instance: u32,
}
```

Per-resource-class worker counts and library-wide job semaphores.

**Fields**

- `network`: Scan / acquire / listening-sync workers on this node (default 1),
  and the library-wide number of running network jobs (the
  `jobs:network` semaphore).
- `plugin_instance`: Library-wide running jobs that require one plugin instance (the
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
    pub role: ProcessRole,
}
```

Options for [`init_tracing_with`](#init_tracing_with).

**Fields**

- `format`: Stderr format (text vs JSON).
- `default_level`: Default `EnvFilter` directive when `BOOKCLERK_LOG` / `RUST_LOG` are unset.
  Affects stderr + OS facility only — the diagnostics ring has its own
  filter ([`ring_filter`](#ring_filter)).
- `syslog_identifier`: Identifier for journald / Event Log source / os_log category.
- `diagnostics`: Opt-in crash / error-burst upload settings.
- `version`: Package version embedded in diagnostics uploads.
- `enable_journald`: Attempt to attach an OS log facility (no-op when unavailable).
- `role`: What kind of process this is, reported in diagnostics uploads.

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

### DestinationId

```rust
pub struct DestinationId(/* private */);
```

Id of one output destination: the plugin instance id of the deployed
storage plugin instance that serves it.

Plans, upload manifests and `edition_files` name destinations by it.
Several instances of one storage plugin are separate destinations
(`docs/adr/storage-destinations.md`).

#### DestinationId::MAX_LEN

```rust
pub const MAX_LEN: usize = 64
```

Longest destination id in bytes; a hyphenated UUID is 36.

#### DestinationId::new

```rust
pub fn new(id: impl Into<String>) -> Result<Self, String>
```

Checks `id`: 1 to [`Self::MAX_LEN`](#destinationidmax_len) lowercase ASCII letters, digits,
`_` or `-`, the characters a plugin instance id uses.

##### Errors

Returns a message naming what is wrong with `id`.

#### DestinationId::as_str

```rust
pub fn as_str(&self) -> &str
```

The id as text.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `From<DestinationId>`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `TryFrom<String>`

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
    pub copies: Vec<OutputCopy>,
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
    pub keep_codec_in_m4b: bool,
    pub chapter_json: ChapterJsonMode,
    pub save_chapter_json: Option<bool>,
    pub save_metadata_json: bool,
    pub cover_size: String,
    pub chapter_layout: String,
    pub overwrite_existing: bool,
    pub multi_destination: MultiDestinationMode,
    pub missing_files: MissingFileAction,
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
- `copies`: Further copies of every acquired book, beside the main output
  ([`Self::format`](#outputconfig) with [`Self::lame`](#outputconfig)), at most
  [`crate::MAX_EXTRA_COPIES`](#max_extra_copies). Empty (the default) keeps one copy.
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
- `keep_codec_in_m4b`: Keep Opus, FLAC, ALAC, E-AC-3 and AC-4 audio unchanged inside an
  M4B copy instead of encoding it to AAC (an advanced setting: fewer
  players open those codecs in MP4). Off by default.
- `chapter_json`: Chapter JSON sidecars: `off` | `flat` | `tree` | `both`.
- `save_chapter_json`: Deprecated: use [`Self::chapter_json`](#outputconfig).
- `save_metadata_json`: Persist raw catalog API JSON (`metadata.json`).
- `cover_size`: Cover image size for download/embed (`500`, `1215`, or `native`).
- `chapter_layout`: Preferred Audible chapter API layout when fetching (`tree` or `flat`).
- `overwrite_existing`: Re-acquire when media already exists at the destination.
- `multi_destination`: When multiple destinations are enabled and only some already have the
  title: copy from a present dest, re-fetch into missing only, or re-fetch
  into every destination.
- `missing_files`: What happens to a stored copy whose file disappears from its
  destination outside Bookclerk.
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

#### OutputConfig::copies_to_make

```rust
pub fn copies_to_make(&self) -> Vec<OutputCopy>
```

Every copy an acquire makes, main output first
([`crate::copies_to_make`](#copies_to_make)).

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

### CopyError

```rust
pub struct CopyError(/* private */);
```

Why a copy list was refused.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### CopyQuality

```rust
pub struct CopyQuality {
    /* private fields */
}
```

The quality of one copy, by name.

The names are `original` (the store's audio, re-packaged without
re-encoding where the format allows), a bitrate such as `64k` (one of
the MP3 encoder's constant rates, [`MP3_CBR_KBPS`](#mp3_cbr_kbps-1), or an Opus target,
[`OPUS_KBPS`](#opus_kbps)) or a LAME VBR quality `v0` to `v9`; either of the last
two may end in `-mono` for a copy mixed down to one channel. Which names
suit which format is [`OutputCopy::validate`](#outputcopyvalidate)'s to say. Names are stored
in lower case.

#### CopyQuality::ORIGINAL

```rust
pub const ORIGINAL: Self = _
```

The `original` quality.

#### CopyQuality::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Reads a quality name, without case; `None` for any text that is not
one (see the type's documentation).

#### CopyQuality::is_original

```rust
pub fn is_original(&self) -> bool
```

True for `original`.

#### CopyQuality::is_mono

```rust
pub fn is_mono(&self) -> bool
```

True for a copy mixed down to one channel (`-mono`).

#### CopyQuality::kbps

```rust
pub fn kbps(&self) -> Option<u16>
```

The bitrate the name gives, in kbit/s; `None` for `original` and
the VBR levels.

#### CopyQuality::of_lame

```rust
pub fn of_lame(lame: &LameConfig) -> Self
```

The quality the library's LAME settings give an MP3.

#### CopyQuality::lame

```rust
pub fn lame(&self, base: &LameConfig) -> LameConfig
```

`base` with this quality's encoder settings: a bitrate selects a
constant bitrate, a VBR level a VBR quality, and `-mono` mixes down.
`original` leaves `base` as it is.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `PartialSchema`, `Serialize`, `ToSchema`

### OpusSettings

```rust
pub struct OpusSettings {
    pub kbps: u16,
    pub mono: bool,
}
```

The Opus encoder settings one output is written with.

The main output uses [`OPUS_MAIN_KBPS`](#opus_main_kbps), keeping the source's channels
up to stereo; an Opus copy takes the
bitrate and channel choice its quality names ([`OutputCopy::opus_kbps`](#outputcopyopus_kbps)).

**Fields**

- `kbps`: Target bitrate in kbit/s; Opus encodes in variable bitrate around it.
- `mono`: When true, the encoder downmixes to one channel.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OutputCopy

```rust
pub struct OutputCopy {
    pub format: CopyFormat,
    pub quality: CopyQuality,
}
```

One copy of every acquired book: a format at a quality.

**Fields**

- `format`: The copy's format.
- `quality`: The copy's quality.

#### OutputCopy::main

```rust
pub fn main(format: OutputFormat, lame: &LameConfig) -> Option<Self>
```

The copy the library's main output makes: its format, with
`original` quality for M4B and as-delivered output, the LAME
settings' quality for MP3 and [`OPUS_MAIN_KBPS`](#opus_main_kbps) for Opus. `None`
for an output format the acquire pipeline refuses.

#### OutputCopy::validate

```rust
pub fn validate(&self) -> Result<(), CopyError>
```

Checks that the quality suits the format: M4B and as-delivered
copies are `original` only (the M4B is re-packaged, not
re-encoded), MP3 copies need one of the MP3 encoder's constant
rates or a VBR quality, and Opus copies one of the Opus targets.

##### Errors

Returns a [`CopyError`](#copyerror) naming the copy when they do not suit.

#### OutputCopy::opus_kbps

```rust
pub fn opus_kbps(&self) -> Option<(u16, bool)>
```

The Opus encoder settings for an Opus copy at this quality; `None`
for any other format or a quality without a bitrate.

#### OutputCopy::label

```rust
pub fn label(&self) -> String
```

The copy's label in a storage key, for a destination that keeps
more than one copy of a book: `mp3 64k-mono`.

#### OutputCopy::parse

```rust
pub fn parse(raw: &str) -> Result<Self, CopyError>
```

Reads one copy written `format:quality` (`mp3:64k-mono`), as the
`output.copies` setting lists them.

##### Errors

Returns a [`CopyError`](#copyerror) for text that is not that shape, an unknown
format or quality, or a quality that does not suit the format.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

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
  `0` disables. Host keys use [`MAX_HOST_KEY_BYTES`](#max_host_key_bytes) minus the prefix length.
- `measure`: Whether segment truncation counts Unicode scalars or UTF-8 bytes.

#### PathLimits::for_host_keys

```rust
pub fn for_host_keys(max_filename_length: u32, prefix: &str) -> Self
```

Host-key limits for a destination whose keys sit under `prefix`:
segments of at most `max_filename_length` UTF-8 bytes, and a whole
key of [`MAX_HOST_KEY_BYTES`](#max_host_key_bytes) minus the normalized `prefix` length
(its trailing `/` included when the prefix is non-empty).

Bytes are counted for every destination: a name that fits in bytes
fits in characters and UTF-16 units too.

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
- `search_index_dir`: The node's Tantivy works search index (`search_index_works/`).

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

### DestinationFilter

```rust
pub struct DestinationFilter {
    pub rules: Vec<FilterRule>,
}
```

Which books one destination takes.

**Fields**

- `rules`: Filter rules, at most 64. Empty means the destination takes every
  book.

#### DestinationFilter::takes_everything

```rust
pub fn takes_everything(&self) -> bool
```

A filter that takes every book: no rules.

#### DestinationFilter::accepts

```rust
pub fn accepts(&self, facts: &PlacementFacts) -> bool
```

True when the destination takes the book described by `facts`:
among the book rules (every rule but `format` and `quality`), no
exclude rule matches, and either there is no include rule or one of
them matches. Copy rules are not considered.

#### DestinationFilter::accepts_copy

```rust
pub fn accepts_copy(&self, copy: &OutputCopy) -> bool
```

True when the destination receives `copy` of a book it takes: among
the copy rules (`format` and `quality`), no exclude rule matches,
and either there is no include rule or one of them matches. Book
rules are not considered; a filter with no copy rules receives every
copy.

#### DestinationFilter::has_copy_rules

```rust
pub fn has_copy_rules(&self) -> bool
```

True when the filter has a `format` or `quality` rule.

#### DestinationFilter::validate

```rust
pub fn validate(&self) -> Result<(), FilterError>
```

Checks the filter's bounds.

##### Errors

Returns a [`FilterError`](#filtererror) for more than [`MAX_FILTER_RULES`](#max_filter_rules) rules,
an empty or over-long value, a value with a NUL, a `patron` value
that is not a positive user id, a `media_kind` value other than
`book` or `podcast`, a `format` value that is not a copy format, a
`quality` value that is not a copy quality, or a rule given twice
(the same field, value and effect, compared without case).

#### DestinationFilter::normalized

```rust
pub fn normalized(self) -> Self
```

The filter with each value trimmed, as it is stored.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### FilterError

```rust
pub struct FilterError(/* private */);
```

Why a [`DestinationFilter`](#destinationfilter) was refused.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### FilterRule

```rust
pub struct FilterRule {
    pub field: FilterField,
    pub value: String,
    pub effect: FilterEffect,
}
```

One rule of a destination filter: a field, one value of it, and its
effect.

**Fields**

- `field`: The field the rule matches.
- `value`: The value: a user id for `patron`, `book` or `podcast` for
  `media_kind`, a copy format for `format`, a copy quality for
  `quality`, otherwise 1 to 128 bytes of text.
- `effect`: Include or exclude.

#### FilterRule::matches

```rust
pub fn matches(&self, facts: &PlacementFacts) -> bool
```

True when the book described by `facts` has this rule's value in
the rule's field. The effect is not considered, and a copy rule
(`format`, `quality`) never matches a book.

#### FilterRule::matches_copy

```rust
pub fn matches_copy(&self, copy: &OutputCopy) -> bool
```

True when `copy` has this rule's value in the rule's field. The
effect is not considered, and a book rule never matches a copy.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### PlacementFacts

```rust
pub struct PlacementFacts {
    pub patrons: Vec<i64>,
    pub storefront: String,
    pub account: String,
    pub media_kind: PlacementMediaKind,
}
```

What a destination filter knows about one book when an acquire starts.

**Fields**

- `patrons`: Users linked to the store account that holds the book, ascending.
  Empty when the account is linked to nobody.
- `storefront`: The storefront plugin's alias (`audible`, `libro`, …).
- `account`: The holding account's id at that store.
- `media_kind`: What the book is.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NodeLimits

```rust
pub struct NodeLimits {
    pub cpu_millis: Option<u64>,
    pub memory_bytes: Option<u64>,
    pub cpus: u64,
}
```

The CPU and memory a node may use, as its cgroup limits them.

**Fields**

- `cpu_millis`: CPU limit in thousandths of a CPU (`250` is a quarter of one), from
  cgroup v2 `cpu.max`; `None` when unlimited or unknown.
- `memory_bytes`: Memory limit in bytes, from cgroup v2 `memory.max`; `None` when
  unlimited or unknown.
- `cpus`: CPUs the process may run on (`available_parallelism`), at least 1.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### IntegrationsConfig

```rust
pub struct IntegrationsConfig {
    pub claim_ticket_ttl_hours: u64,
    pub public_origin: Option<String>,
    pub portal_session_ttl_hours: u64,
    pub ingress_origin: Option<String>,
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
- `ingress_origin`: Public origin the library's active ingress plugin reported, loaded
  from the database at run time.
  
  Never read from or written to `config.toml`: handlers write the
  in-memory configuration back to the file, and a plugin-reported name
  must not become the operator's override. Read the origin through
  [`IntegrationsConfig::effective_public_origin`](#integrationsconfigeffective_public_origin).

#### IntegrationsConfig::effective_public_origin

```rust
pub fn effective_public_origin(&self) -> Option<&str>
```

The library's public origin: the operator's
`integrations.public_origin` when set, otherwise the origin the
active ingress plugin reported.

##### Returns

The origin as written (callers trim and normalize it), or `None` when
neither is set.

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
    pub webhooks: WebhooksExposure,
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
- `webhooks`: Whether plugins' `[[webhooks]]` take requests from the Internet on
  `POST /hooks/<token>/<name>`.

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

### RelayedLine

```rust
pub struct RelayedLine<'a> {
    pub level: RelayedLevel,
    pub text: &'a str,
}
```

One relayed line: its level and its text without the timestamp and level.

**Fields**

- `level`: The level to log it at.
- `text`: The text after the prefix (usually `<target>: <message> <fields>`),
  or the whole line, trimmed, when it had no prefix.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### AccessCode

```rust
pub struct AccessCode(/* private */);
```

A report's access code: the credential that reads the report back from
the collector or deletes it. Canonical form: exactly
[`ACCESS_CODE_LENGTH`](#access_code_length) characters of [`ACCESS_CODE_ALPHABET`](#access_code_alphabet), no
separators. `Debug` never prints it.

#### AccessCode::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a canonical access code, as the collector sends it.

##### Arguments

* `text` - Candidate text.

##### Returns

The code, or `None` unless the text is exactly
[`ACCESS_CODE_LENGTH`](#access_code_length) alphabet characters.

#### AccessCode::as_str

```rust
pub fn as_str(&self) -> &str
```

The canonical code.

#### AccessCode::display

```rust
pub fn display(&self) -> String
```

The code as people read it, in hyphenated groups of four
(`K7QM-3XRD-PW9T-H2CF`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ReportId

```rust
pub struct ReportId(/* private */);
```

A report ID the collector assigned: a version-4 UUID in lowercase
hyphenated form.

#### ReportId::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Parses a report ID.

##### Arguments

* `text` - Candidate text. Unlike the install ID file, no
  surrounding whitespace is allowed.

##### Returns

The ID, or `None` unless the text is a lowercase hyphenated
version-4 UUID.

#### ReportId::as_str

```rust
pub fn as_str(&self) -> &str
```

The ID as text.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ReportRecord

```rust
pub struct ReportRecord {
    pub report_id: ReportId,
    pub trigger: String,
    pub version: String,
    pub sent_at_unix_ms: u64,
    pub access_code: Option<AccessCode>,
    pub deleted_at_unix_ms: Option<u64>,
}
```

One report this install sent.

**Fields**

- `report_id`: The collector's ID for the report.
- `trigger`: Why it was sent (`crash`, `error_burst`, `cli_error`, …).
- `version`: The Bookclerk version that sent it.
- `sent_at_unix_ms`: When the collector accepted it, Unix milliseconds.
- `access_code`: The code that reads it back or deletes it, when the collector
  issued one.
- `deleted_at_unix_ms`: When this install deleted the collector's stored copy, Unix
  milliseconds, or `None` while it is stored.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SubmitReply

```rust
pub struct SubmitReply {
    pub report_id: ReportId,
    pub access_code: Option<AccessCode>,
}
```

What the collector's `POST /submit` reply names.

**Fields**

- `report_id`: The report's ID.
- `access_code`: Its access code, or `None` when the collector issued none (no
  dashboard database, its daily cap reached, or an older collector) or
  sent one that is not a canonical code.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LookupAnswer

```rust
pub struct LookupAnswer {
    pub report_id: ReportId,
    pub received_at_unix_ms: u64,
    pub deleted_at_unix_ms: Option<u64>,
    pub report: Option<LookupReport>,
    pub issues: Vec<LookupIssue>,
    pub fix: Option<LookupFix>,
}
```

The collector's answer to a lookup.

**Fields**

- `report_id`: The report's ID; always the one that was asked for.
- `received_at_unix_ms`: When the collector received it, Unix milliseconds.
- `deleted_at_unix_ms`: When its sender deleted the stored copy, Unix milliseconds, or
  `None` while it is stored.
- `report`: The stored report, or `None` once deleted (or when the collector
  could not read it).
- `issues`: Issues tracking it, the tracking issue first.
- `fix`: The tracking issue's fix, if one is recorded.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### LookupEvent

```rust
pub struct LookupEvent {
    pub ts_unix_ms: u64,
    pub level: EventLevel,
    pub target: String,
    pub message: String,
    pub fields: Vec<(String, String)>,
}
```

One logged event in a stored report, already redacted by the install
that sent it.

**Fields**

- `ts_unix_ms`: When it happened, Unix milliseconds, or `0` when the report had none.
- `level`: Its severity.
- `target`: Where it was logged (the tracing target).
- `message`: The message.
- `fields`: `(name, value)` pairs recorded with it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### LookupFix

```rust
pub struct LookupFix {
    pub pr_number: Option<u64>,
    pub pr_url: Option<String>,
    pub fixed_in: Option<String>,
}
```

The fix recorded for the report's tracking issue.

**Fields**

- `pr_number`: The pull request that fixed it, when known.
- `pr_url`: That pull request's page (`https://`), when known.
- `fixed_in`: The first release that contains the fix, when one has shipped.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### LookupIssue

```rust
pub struct LookupIssue {
    pub number: u64,
    pub title: String,
    pub state: IssueState,
    pub url: String,
}
```

A GitHub issue that tracks the report.

**Fields**

- `number`: The issue number.
- `title`: Its title.
- `state`: Whether it is open.
- `url`: Its page; always an `https://` link.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### LookupReport

```rust
pub struct LookupReport {
    pub version: String,
    pub trigger: String,
    pub os: String,
    pub arch: String,
    pub distro: String,
    pub events: Vec<LookupEvent>,
}
```

A stored report as the collector returns it.

**Fields**

- `version`: The Bookclerk version that sent it.
- `trigger`: Why it was sent.
- `os`: The sender's operating system.
- `arch`: The sender's CPU architecture.
- `distro`: The sender's Linux distribution, or empty.
- `events`: The events it carried, oldest first.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### AuthConfig

```rust
pub struct AuthConfig {
    pub password: Option<String>,
    pub key_store: Option<KeyStoreKind>,
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
- `key_store`: Where this install keeps the library's data-encryption key
  (`[auth].key_store`). Unset means the host's default: the OS keychain
  for the desktop app, the `master.key` file for the daemon and CLI.
  Set to `keychain`, it is required: a keychain that can't be reached
  is an error rather than a fallback to the file. `systemd-creds`
  always falls back to the file when the key can't be sealed.
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
    pub uploads: UploadsConfig,
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
- `uploads`: Upload limits on this node (`[uploads]`).
- `events_revision`: Revision of the DB-authoritative `[events]` document after overlay.
  
  Absent until control-plane bootstrap or reconcile loads `core.events`.
  Not serialized to `config.toml`. Compared only with
  [`Self::events_authority`](#config).
- `events_authority`: Library id that produced [`Self::events_revision`](#config).
  
  A revision from another library is not newer or older; a database swap
  replaces this authority explicitly. Not serialized to `config.toml`.
- `output_revision`: Revision of the DB-authoritative `[output]` document (`core.output`)
  after overlay. Absent until control-plane bootstrap or reconcile loads
  it; compared only with [`Self::output_authority`](#config). Not serialized.
- `output_authority`: Library id that produced [`Self::output_revision`](#config). Not serialized.

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
- `login_max_failures`: Failed sign-in attempts before a lockout, counted per client IP across
  the sign-in routes, and per account for password re-checks by a
  signed-in user (elevation, password change, `current_password`).
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
- `node_api`: Transport policy for the branch-node API (`[daemon.node_api]`).
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
    pub share_usage: bool,
    pub demo: bool,
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
- `ring_buffer_capacity`: Max redacted events retained for upload: Bookclerk's own events
  through TRACE, other crates' from INFO up.
- `share_usage`: When true, send a weekly anonymous check-in (a random install ID,
  version, platform and kind) so the project can count active installs.
  Separate from `share_reports`; reports carry the install ID only when
  this is on.
- `demo`: Marks this install as a copy of the hosted demo: reports say
  `deployment = "demo"` and no check-in is sent. The demo image sets it.

#### DiagnosticsConfig::effective_collector_url

```rust
pub fn effective_collector_url(&self) -> String
```

Resolved Worker origin: config/env `collector_url`, else baked deploy URL.

#### DiagnosticsConfig::effective_lookup_url

```rust
pub fn effective_lookup_url(&self) -> String
```

HTTPS endpoint the install reads its own reports back from
(`…/lookup`; deletion is `…/lookup/delete`), or empty when no
collector is configured.

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
    pub upgrade_quality: bool,
    pub upgrade_daily_limit: u32,
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
- `enrich_from_audible`: After scan, match non-Audible rows (e.g. Libro.fm) to Audible ASINs via the public
  Audible catalog and the deployed metadata providers (the Audnexus plugin).
- `enrich_min_confidence`: Minimum match confidence (0–100) to accept an Audible ASIN enrichment.
  Uses AudioBookshelf-style duration/title/author scoring (default 90).
- `fix_storage_layout`: When matching storage to the library, relocate matched audio (and
  accompanying sidecars) onto the configured naming-profile layout.
  Default false — match in place without moving files.
- `upgrade_quality`: Once a day, ask each store that answers quality offers whether an
  account would now download a better file than the one the library
  holds, and download it again when it would (`bookclerkd`). Off by
  default.
- `upgrade_daily_limit`: Most quality upgrades queued per store account per day (at least
  1); each download also waits for the account's pace.

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

`[daemon.node_api]`: transport policy for branch nodes.

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

### UploadsConfig

```rust
pub struct UploadsConfig {
    pub max_file_bytes: u64,
    pub max_staged_bytes: u64,
    pub free_disk_floor_percent: u8,
    pub idle_ttl_hours: u64,
}
```

`[uploads]`: per-node upload limits.

**Fields**

- `max_file_bytes`: Largest single file, in bytes.
- `max_staged_bytes`: Most bytes one person may have staged in unfinished batches.
- `free_disk_floor_percent`: Percent of the node's disk (0 to 90) that must stay free; a batch
  or chunk that would leave less is refused.
- `idle_ttl_hours`: Hours an open batch may sit without a chunk before it is pruned.

#### UploadsConfig::free_disk_floor

```rust
pub fn free_disk_floor(&self) -> u8
```

The free-disk floor as a share, capped at 90 percent so a typo
cannot close staging for good.

#### UploadsConfig::idle_ttl

```rust
pub fn idle_ttl(&self) -> Duration
```

The idle time, at least one hour.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### CacheSettingError

```rust
pub enum CacheSettingError {
    UnknownKey(String),
    NotANumber(&'static str),
    OutOfRange { key: &'static str, min: u64, max: u64 },
}
```

Why a `cache.*` value was refused.

**Variants**

- `UnknownKey`: The key is not a `cache.*` key.
- `NotANumber`: The value is not a whole number without sign or separators.
- `OutOfRange`: The value is a whole number outside the key's range.
  - `key`: The dotted key.
  - `min`: Smallest accepted value.
  - `max`: Largest accepted value.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### CacheSettingKey

```rust
pub enum CacheSettingKey {
    BudgetBytes,
    UnviewedDays,
}
```

A `cache.*` settings key.

**Variants**

- `BudgetBytes`: `cache.budget_bytes`: bytes one node's image caches may hold.
- `UnviewedDays`: `cache.unviewed_days`: days an entry may go unviewed.

#### CacheSettingKey::ALL

```rust
pub const ALL: [Self; 2] = _
```

Every key, in display order.

#### CacheSettingKey::as_str

```rust
pub const fn as_str(self) -> &'static str
```

The dotted key, such as `cache.budget_bytes`.

#### CacheSettingKey::parse

```rust
pub fn parse(key: &str) -> Option<Self>
```

The key `key` names, matched exactly.

##### Arguments

* `key` - A dotted settings key.

##### Returns

`None` for anything but a `cache.*` key.

#### CacheSettingKey::range

```rust
pub const fn range(self) -> (u64, u64)
```

The smallest and largest value the key accepts.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

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

### ProcessRole

```rust
pub enum ProcessRole {
    Daemon,
    Desktop,
    Cli,
}
```

Which kind of Bookclerk process installed the subscriber.

**Variants**

- `Daemon`: The long-running daemon (`bookclerkd`).
- `Desktop`: The desktop window, which runs the host in-process.
- `Cli`: The command-line tool (`bookclerk`).

#### ProcessRole::as_str

```rust
pub fn as_str(self) -> &'static str
```

Name reported in uploads and check-ins (`daemon`, `desktop`, `cli`).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

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

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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

- `Auto`: Leave each destination's character rules to its storage plugin:
  the host only strips path separators
  ([`posix_replacement_characters`](#posix_replacement_characters)), and the plugin translates the
  key into what it can store (a Windows-safe path for `local` on
  Windows, an S3-safe key for an object store).
- `Windows`: NTFS / classic Libation Windows map (`: * ? " < > | / \`).
- `Posix`: Path-separator only (`/` `\`) — suitable for Linux/macOS local disks.
- `S3`: AWS S3 “characters to avoid” plus path separators.
- `None`: Disable platform-specific sanitization; path separators (`/` `\`) are
  still stripped so metadata cannot inject extra directories.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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
  layout: `{Author}/{Series}/{N} - {Book}` or `{Author}/{Book}`; the
  file is the full title with the edition's call number.
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

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### MissingFileAction

```rust
pub enum MissingFileAction {
    MarkMissing,
    Restore,
}
```

What happens to a stored copy whose file disappears from its destination
outside Bookclerk (noticed by the destination's change feed or listing).

**Variants**

- `MarkMissing`: The copy is marked missing and left alone. When no destination holds
  the edition any more, it is no longer acquired, so a member can ask
  for it again. Default.
- `Restore`: The copy turns pending and the background retry writes it again.

#### MissingFileAction::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### MissingFileAction::parse

```rust
pub fn parse(value: &str) -> Option<Self>
```

The action a spelling names, ignoring ASCII case and taking `-` for
`_`; `None` for any other value.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### CopyFormat

```rust
pub enum CopyFormat {
    M4b,
    Mp3,
    Mp3Chapters,
    Original,
    Opus,
}
```

The format of one copy of a book.

**Variants**

- `M4b`: One M4B with tags and chapters (output format `enriched_m4b`).
- `Mp3`: One MP3 (output format `single_mp3`).
- `Mp3Chapters`: One MP3 per chapter (output format `split_mp3_by_chapter`).
- `Original`: The audio as the store delivered it (output format `none`).
- `Opus`: One Ogg Opus file with tags and chapters (output format `opus`), an
  advanced format.

#### CopyFormat::ALL

```rust
pub const ALL: [Self; 5] = _
```

Every format, in the order the names are documented.

#### CopyFormat::as_str

```rust
pub fn as_str(self) -> &'static str
```

The format's name in settings, filter rules and file rows.

#### CopyFormat::is_advanced

```rust
pub fn is_advanced(self) -> bool
```

True for the advanced formats: ones players and servers read less
widely than M4B and MP3, which settings screens list apart from the
common ones.

#### CopyFormat::parse

```rust
pub fn parse(raw: &str) -> Option<Self>
```

Reads a format name, without case; `None` for any other text.

#### CopyFormat::of_output

```rust
pub fn of_output(format: OutputFormat) -> Option<Self>
```

The copy format an output format produces; `None` for the format
the acquire pipeline refuses (`split_mp3_by_size`).

#### CopyFormat::output

```rust
pub fn output(self) -> OutputFormat
```

The output format the acquire pipeline packages this copy with.

#### CopyFormat::is_encoded

```rust
pub fn is_encoded(self) -> bool
```

True for the formats an encoder writes (MP3, Opus), whose quality
is an encoder setting rather than `original`.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

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

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

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
`split_mp3_by_size` parses but is not implemented yet: the acquire
pipeline rejects a job that selects it before downloading.

**Variants**

- `EnrichedM4b`: Remux/transcode to M4B and embed metadata/chapters (default).
- `SingleMp3`: Single MP3 file (LAME re-encode).
- `SplitMp3ByChapter`: One MP3 per chapter (after M4B chapter split).
- `SplitMp3BySize`: Split MP3 by target file size (see `split_mp3_max_mb`). Not yet
  implemented: the acquire pipeline rejects it before downloading.
- `Opus`: One Ogg Opus file (libopus, 64 kbit/s, the source's channels up to
  stereo) with Vorbis comment tags, chapters and cover.
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

True when the format encodes to Ogg Opus.

#### OutputFormat::prefers_m4b_container

```rust
pub fn prefers_m4b_container(self) -> bool
```

True when the pipeline prefers an M4B intermediate (enriched or chapter-split).

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### FilterEffect

```rust
pub enum FilterEffect {
    Include,
    Exclude,
}
```

Whether a [`FilterRule`](#filterrule) lets matching books in or keeps them out.

**Variants**

- `Include`: Matching books are taken (once any include rule exists, only books
  that match one are).
- `Exclude`: Matching books are refused, whatever the include rules say.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### FilterField

```rust
pub enum FilterField {
    Patron,
    Storefront,
    Account,
    MediaKind,
    Format,
    Quality,
}
```

The book field a [`FilterRule`](#filterrule) matches one value of.

**Variants**

- `Patron`: A patron whose store account holds the book, by user id (`3`).
- `Storefront`: The storefront the book came from, by its plugin's alias
  (`audible`, `libro`), compared without case.
- `Account`: The store account that holds the book, by its id at the store.
- `MediaKind`: What the book is: `book` or `podcast` (a podcast episode is a
  podcast).
- `Format`: A copy's format ([`CopyFormat`]: `m4b`, `mp3`, `mp3_chapters` or
  `original`), compared without case.
- `Quality`: A copy's quality ([`CopyQuality`]: `original`, `64k`, `v2-mono` and
  so on), compared without case.

#### FilterField::ALL

```rust
pub const ALL: [Self; 6] = _
```

Every field, in the order the names are documented.

#### FilterField::parse

```rust
pub fn parse(name: &str) -> Option<Self>
```

Reads a field by its wire name ([`Self::as_str`](#filterfieldas_str)), exactly; `None`
for any other text.

#### FilterField::as_str

```rust
pub fn as_str(self) -> &'static str
```

The field's name on the wire.

#### FilterField::is_copy_field

```rust
pub fn is_copy_field(self) -> bool
```

True for the fields that describe a copy of the book (`format`,
`quality`) rather than the book.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `ToSchema`

### PlacementMediaKind

```rust
pub enum PlacementMediaKind {
    Book,
    Podcast,
}
```

What a book is, as a destination filter sees it.

**Variants**

- `Book`: A book (any form: narration, dramatisation, ebook and so on).
- `Podcast`: A podcast show or one of its episodes.

#### PlacementMediaKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

The kind's name on the wire and in rule values.

#### PlacementMediaKind::parse

```rust
pub fn parse(value: &str) -> Option<Self>
```

Reads a rule value (`book` or `podcast`, without case).

#### PlacementMediaKind::from_content_kind

```rust
pub fn from_content_kind(content_kind: &str) -> Self
```

The kind of a library title from its content kind label (`book`,
`podcast` or `episode`): episodes and shows are podcasts, everything
else is a book.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Eq`, `PartialEq`, `Serialize`, `ToSchema`

### WebhooksExposure

```rust
pub enum WebhooksExposure {
    Auto,
    Off,
}
```

`plugins.webhooks`: whether the public webhook route is open.

**Variants**

- `Auto`: Hand out webhook URLs and take requests when
  `integrations.public_origin` is an `https` origin on a globally
  reachable host. The default.
- `Off`: Hand out no webhook URLs and answer every request on the route with
  503; stored requests stay until they expire.

#### WebhooksExposure::as_str

```rust
pub fn as_str(self) -> &'static str
```

Canonical spelling, as written in `config.toml`.

#### WebhooksExposure::parse

```rust
pub fn parse(value: &str) -> Option<Self>
```

Parses a config or settings value.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RelayedLevel

```rust
pub enum RelayedLevel {
    Trace,
    Debug,
    Info,
    Warn,
}
```

The level a relayed line is logged at.

**Variants**

- `Trace`: `TRACE`.
- `Debug`: `DEBUG`.
- `Info`: `INFO`, and every line without a level prefix.
- `Warn`: `WARN`, and also a line that said `ERROR` (relayed lines are capped
  here).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SubmitReplyError

```rust
pub enum SubmitReplyError {
    TooLong,
    NotJson,
    BadReportId,
}
```

Why a collector reply was refused.

**Variants**

- `TooLong`: The reply is longer than [`MAX_SUBMIT_REPLY_BYTES`](#max_submit_reply_bytes).
- `NotJson`: The reply is not a JSON object with a string `report_id`.
- `BadReportId`: `report_id` is present but not a version-4 UUID.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### EventLevel

```rust
pub enum EventLevel {
    Error,
    Warn,
    Info,
    Debug,
    Trace,
}
```

Severity of one logged event in a report.

**Variants**

- `Error`: `ERROR`.
- `Warn`: `WARN`.
- `Info`: `INFO`.
- `Debug`: `DEBUG`.
- `Trace`: `TRACE`.

#### EventLevel::parse

```rust
pub fn parse(text: &str) -> Option<Self>
```

Reads the collector's spelling.

##### Arguments

* `text` - `ERROR`, `WARN`, `INFO`, `DEBUG` or `TRACE`.

##### Returns

The level, or `None` for anything else.

#### EventLevel::as_str

```rust
pub fn as_str(self) -> &'static str
```

The level as the collector and the CLI spell it.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### IssueState

```rust
pub enum IssueState {
    Open,
    Closed,
}
```

Whether a GitHub issue is open.

**Variants**

- `Open`: Still open.
- `Closed`: Closed.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`, `Serialize`

### LookupError

```rust
pub enum LookupError {
    NotConfigured,
    NotFound,
    RateLimited,
    Unavailable,
    Status(u16),
    Transport(String),
    BadReply(LookupReplyError),
}
```

Why a lookup or delete did not succeed.

**Variants**

- `NotConfigured`: No collector URL is configured.
- `NotFound`: The collector knows no report with this ID and code (`404`); the two
  cases are not told apart.
- `RateLimited`: Too many attempts for this report lately (`429`); try again in a
  minute.
- `Unavailable`: The collector cannot serve lookups (`503`: no dashboard database).
- `Status`: The collector answered with another error status.
- `Transport`: The request did not complete.
- `BadReply`: The collector's answer was refused.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### LookupReplyError

```rust
pub enum LookupReplyError {
    TooLong,
    NotJson,
    BadShape,
    OtherReport,
}
```

Why a collector answer was refused.

**Variants**

- `TooLong`: Longer than [`MAX_LOOKUP_REPLY_BYTES`](#max_lookup_reply_bytes).
- `NotJson`: Not a JSON object.
- `BadShape`: A field is missing, of the wrong type, or a link is not `https://`.
- `OtherReport`: The answer names a report other than the one asked for.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

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

### KeyStoreKind

```rust
pub enum KeyStoreKind {
    File,
    Keychain,
    SystemdCreds,
}
```

Where an install keeps the library's data-encryption key.

**Variants**

- `File`: `{files_dir}/master.key` holds the key itself (`BCK1`, or `BCK2`
  under the auth password).
- `Keychain`: The OS keychain (Keychain Services, Credential Manager or Secret
  Service) holds the key; `master.key` only names the item (`BCK3`).
- `SystemdCreds`: `systemd-creds` seals the key to this machine (its TPM where there
  is one) and the daemon's user, or the service unit hands it in;
  `master.key` only names the item (`BCK4`). For headless Linux.

#### KeyStoreKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

The setting's spelling: `file`, `keychain` or `systemd-creds`.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `FromStr`, `PartialEq`, `Serialize`

### NodeApiMode

```rust
pub enum NodeApiMode {
    Loopback,
    Proxy,
    Acme,
    SelfSigned,
    Tunnel,
}
```

How branch nodes may reach the node API (`[daemon.node_api] mode`).

Plain HTTP is accepted only on loopback, in every mode.

**Variants**

- `Loopback`: Loopback connections only (the default).
- `Proxy`: A TLS-terminating reverse proxy listed in `daemon.trusted_proxies`.
- `Acme`: The built-in TLS listener with an ACME (Let's Encrypt) certificate.
- `SelfSigned`: The built-in TLS listener with a self-signed certificate whose SPKI
  pin travels in enrollment links.
- `Tunnel`: The library's active relay ingress plugin carries public traffic
  through its `ORIGIN` binding; branches verify the relay's edge
  certificate against the webpki roots.

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
docs), retrying briefly while another replace of `to` settles. The moved
file keeps its own permissions, so a `0600` staging file
lands as a `0600` target.

#### Errors

Returns an I/O error when the platform replace operation fails.

### parse_cache_setting

```rust
pub fn parse_cache_setting(
    key: &str,
    value: &str,
) -> Result<(CacheSettingKey, u64), CacheSettingError>
```

Parses one `cache.*` setting.

The value may have surrounding ASCII whitespace; inside, it must be one
or more ASCII digits, no sign, and within the key's [`CacheSettingKey::range`](#cachesettingkeyrange).

#### Arguments

* `key` - The dotted key.
* `value` - The value as text.

#### Returns

The key and the value.

#### Errors

[`CacheSettingError`](#cachesettingerror) for an unknown key, a value that is not a whole
number, or one out of range.

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
) -> Vec<ReplacementRule>
```

Resolve the character-replacement map the host applies to tag values
when it fills a naming template.

Explicit non-empty `replacement_characters` always wins. Otherwise
`path_sanitization` selects a profile; [`PathSanitizationMode::Auto`](#pathsanitizationmode)
strips only path separators and leaves the rest to the destination's
storage plugin, which translates the host key into the path or object
key it stores.

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

### checkin_due

```rust
pub fn checkin_due(files_dir: &Path, now_ms: u64) -> bool
```

Whether a check-in is due.

#### Arguments

* `files_dir` - The Bookclerk files directory.
* `now_ms` - Current Unix time in milliseconds.

#### Returns

True when no check-in was recorded, the record is unreadable, or the last
one is at least [`CHECKIN_INTERVAL_MS`](#checkin_interval_ms) old (or in the future, after a
clock change).

### record_checkin

```rust
pub fn record_checkin(files_dir: &Path, now_ms: u64) -> Result<()>
```

Records an accepted check-in.

#### Arguments

* `files_dir` - The Bookclerk files directory.
* `now_ms` - When it was accepted, Unix milliseconds.

#### Errors

Returns an error when the record cannot be written.

### send_checkin

```rust
pub fn send_checkin(config: &DiagnosticsConfig, checkin: &CheckIn) -> Result<(), String>
```

Sends one check-in to the collector.

#### Arguments

* `config` - Diagnostics settings (collector URL).
* `checkin` - The body.

#### Errors

Returns an error when no collector URL is configured, the body cannot be
serialized, or the request fails.

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
directive — it records Bookclerk's own TRACE and above (other crates' INFO
and above, see [`ring_filter`](#ring_filter)) so uploads retain deep context.
Widen local verbosity with e.g. `BOOKCLERK_LOG=bookclerk=debug` when investigating.

Secrets are redacted on every sink via the public `register_secret` / `redact_str` helpers.

### ring_filter

```rust
pub fn ring_filter() -> Targets
```

The diagnostics ring's per-layer filter: TRACE for
[`RING_TRACE_TARGETS`](#ring_trace_targets) (matched as prefixes, so `bookclerkd` and
`bookclerk_library` count), [`RING_DEPENDENCY_LEVEL`](#ring_dependency_level) for the rest.

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

Precedence: owner > administrator > member.

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

### copies_to_make

```rust
pub fn copies_to_make(main: Option<OutputCopy>, extra: &[OutputCopy]) -> Vec<OutputCopy>
```

Every copy an acquire makes, in order: the main output's copy first
(when the pipeline can make it), then each valid extra copy that
differs from every earlier one, at most [`MAX_EXTRA_COPIES`](#max_extra_copies) of them
whatever the list holds.

### mp3_cbr_kbps

```rust
pub fn mp3_cbr_kbps(kbps: u32) -> u16
```

The constant bitrate, in kbit/s, the MP3 encoder writes for a requested
`kbps`: the smallest of [`MP3_CBR_KBPS`](#mp3_cbr_kbps-1) at or above it, else 320.

### validate_copies

```rust
pub fn validate_copies(
    main: Option<&OutputCopy>,
    copies: &[OutputCopy],
) -> Result<(), CopyError>
```

Checks a list of extra copies: at most [`MAX_EXTRA_COPIES`](#max_extra_copies), each valid
([`OutputCopy::validate`](#outputcopyvalidate)), none given twice and none the main output's
copy `main`.

#### Errors

Returns a [`CopyError`](#copyerror) for the first copy that breaks a rule.

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

### parse_destination_filter

```rust
pub fn parse_destination_filter(body: &[u8]) -> Result<DestinationFilter, FilterError>
```

Parses a destination filter from its raw JSON bytes, sans IO, the way
the web API reads it: the strict JSON shape, then trimmed
([`DestinationFilter::normalized`](#destinationfilternormalized)) and checked
([`DestinationFilter::validate`](#destinationfiltervalidate)).

#### Errors

Returns a [`FilterError`](#filtererror) for bytes that are not the filter's JSON shape
(unknown or repeated keys, wrong types, a missing rule key, an unknown
field or effect name) or a filter that fails
[`DestinationFilter::validate`](#destinationfiltervalidate).

### detect_distro

```rust
pub fn detect_distro() -> Option<String>
```

Best-effort OS distribution string (e.g. `Ubuntu 24.04.1 LTS`).

### node_limits

```rust
pub fn node_limits() -> Option<NodeLimits>
```

This process's cgroup v2 CPU and memory limits, or `None` when it has
neither (or the platform has no cgroups).

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

### register_secret_bytes

```rust
pub fn register_secret_bytes(value: &[u8])
```

Registers one secret value given as bytes, when it is UTF-8 text.

A JSON credential (a token bundle) also registers each string inside
it, at any depth, since a log quotes the token field rather than the
whole document. Bytes that are not UTF-8 register nothing: the text
scrubber can never see them in a log line.

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

### parse_relayed_line

```rust
pub fn parse_relayed_line(line: &str) -> RelayedLine<'_>
```

Splits a tracing-format prefix (optional timestamp, then a level word)
off a relayed stderr line.

A line whose first word (after an optional timestamp, which is a word
starting with a digit and ending in `Z`) is not exactly one of `TRACE`,
`DEBUG`, `INFO`, `WARN` or `ERROR` is returned whole, at
[`RelayedLevel::Info`](#relayedlevel).

#### Arguments

* `line` - One stderr line, ANSI escapes already stripped.

#### Returns

The level and the remaining text, which is always a suffix of `line`
with its trailing whitespace removed.

### find_report

```rust
pub fn find_report(files_dir: &Path, report_id: &ReportId) -> Option<ReportRecord>
```

The report with this ID in the history, if it is there.

#### Arguments

* `files_dir` - The Bookclerk files directory.
* `report_id` - The report.

#### Returns

The newest record with that ID, or `None`.

### load_report_history

```rust
pub fn load_report_history(files_dir: &Path) -> Vec<ReportRecord>
```

Reads the report history, oldest first.

#### Arguments

* `files_dir` - The Bookclerk files directory.

#### Returns

The recorded reports. A missing file is an empty history; an unreadable
or damaged one is too, because the history is a convenience and the
next report rewrites it. Every access code read is registered with log
redaction.

### mark_report_deleted

```rust
pub fn mark_report_deleted(
    files_dir: &Path,
    report_id: &ReportId,
    deleted_at_unix_ms: u64,
) -> Result<()>
```

Notes in the history that the collector's copy of a report was deleted.

#### Arguments

* `files_dir` - The Bookclerk files directory.
* `report_id` - The report.
* `deleted_at_unix_ms` - When the collector deleted it.

#### Errors

Returns an error when the lock cannot be taken or the history cannot be
written. A report not in the history is not an error.

### parse_submit_reply

```rust
pub fn parse_submit_reply(body: &[u8]) -> Result<SubmitReply, SubmitReplyError>
```

Reads the report ID and access code from the collector's `POST /submit`
reply.

#### Arguments

* `body` - The reply body as received.

#### Returns

The report ID, and the access code when `access_code` is a string
holding a canonical code (anything else there is ignored, since the
report is stored either way).

#### Errors

Returns [`SubmitReplyError`](#submitreplyerror) when the body is too long, is not a JSON
object with a string `report_id`, or that value is not a version-4 UUID.

### record_report

```rust
pub fn record_report(files_dir: &Path, record: ReportRecord) -> Result<()>
```

Adds a report to the history, keeping the newest [`MAX_REPORT_HISTORY`](#max_report_history).

Its access code, if any, is registered with log redaction first.

#### Arguments

* `files_dir` - The Bookclerk files directory.
* `record` - The report the collector just accepted.

#### Errors

Returns an error when the lock cannot be taken or the history cannot be
written.

### delete_report

```rust
pub fn delete_report(
    config: &DiagnosticsConfig,
    report_id: &ReportId,
    code: &AccessCode,
) -> Result<u64, LookupError>
```

Deletes the collector's stored copy of a sent report. Its one-line index
entry and issue links stay on the collector.

Blocking: call it off an async runtime's worker threads.

#### Arguments

* `config` - Diagnostics settings (for the collector URL).
* `report_id` - The report.
* `code` - Its access code.

#### Returns

When the copy was deleted, Unix milliseconds.

#### Errors

[`LookupError`](#lookuperror) when no collector is configured, the request fails, the
collector refuses it, or its answer is malformed.

### fetch_report

```rust
pub fn fetch_report(
    config: &DiagnosticsConfig,
    report_id: &ReportId,
    code: &AccessCode,
) -> Result<LookupAnswer, LookupError>
```

Reads a sent report back from the collector.

Blocking: call it off an async runtime's worker threads.

#### Arguments

* `config` - Diagnostics settings (for the collector URL).
* `report_id` - The report.
* `code` - Its access code.

#### Returns

The collector's answer.

#### Errors

[`LookupError`](#lookuperror) when no collector is configured, the request fails, the
collector refuses it, or its answer is malformed.

### parse_delete_reply

```rust
pub fn parse_delete_reply(
    body: &[u8],
    expected: &ReportId,
) -> Result<u64, LookupReplyError>
```

Reads the collector's answer to `POST /lookup/delete`.

#### Arguments

* `body` - The answer body as received.
* `expected` - The report whose deletion was asked for.

#### Returns

When the stored copy was deleted, Unix milliseconds (the first deletion's
time when it was already gone).

#### Errors

Returns [`LookupReplyError`](#lookupreplyerror) when the body is too long, not a JSON
object, has no integer `deleted_at_unix_ms`, or names another report.

### parse_lookup_reply

```rust
pub fn parse_lookup_reply(
    body: &[u8],
    expected: &ReportId,
) -> Result<LookupAnswer, LookupReplyError>
```

Reads the collector's answer to `POST /lookup`.

Every field the collector sends is required with its type, except that
`deleted_at_unix_ms`, `report`, `fix` and the fix's fields may be absent
or `null`; other keys are ignored and, as in any JSON reader, a repeated
key keeps its last value.

#### Arguments

* `body` - The answer body as received.
* `expected` - The report that was asked for.

#### Returns

The answer.

#### Errors

Returns [`LookupReplyError`](#lookupreplyerror) when the body is too long, not a JSON
object, malformed (including an issue or pull request link that is not
`https://`), or names another report.

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

### DEFAULT_CACHE_BUDGET_BYTES

```rust
pub const DEFAULT_CACHE_BUDGET_BYTES: u64 = _;
```

Default disk budget for one node's image caches: 1 GiB.

### DEFAULT_CACHE_UNVIEWED_DAYS

```rust
pub const DEFAULT_CACHE_UNVIEWED_DAYS: u32 = 90;
```

Default days an entry may go unviewed before it is evicted.

### MAX_CACHE_BUDGET_BYTES

```rust
pub const MAX_CACHE_BUDGET_BYTES: u64 = _;
```

Largest accepted cache budget: 1 TiB.

### MAX_CACHE_UNVIEWED_DAYS

```rust
pub const MAX_CACHE_UNVIEWED_DAYS: u32 = 3650;
```

Largest accepted `unviewed_days`, about ten years.

### MIN_CACHE_BUDGET_BYTES

```rust
pub const MIN_CACHE_BUDGET_BYTES: u64 = _;
```

Smallest accepted cache budget: 16 MiB, room for a few hundred covers.

### SLOW_TARGET

```rust
pub const SLOW_TARGET: &str = "bookclerk::slow";
```

Target of the events that mark a slow response.

Events on this target count toward the `slow_burst` trigger instead of
`warn_burst`, so slowness on a starved machine and real faults reach the
collector as different kinds of report. The daemon's request log
(`bookclerkd`) is the writer today.

### RECONCILE_WILDCARD

```rust
pub const RECONCILE_WILDCARD: char = '\u{E000}';
```

Sentinel placed in reconcile path *patterns* where a sanitizable character
appeared in metadata. Matches any single character in an on-disk key.

### CHECKIN_INTERVAL_MS

```rust
pub const CHECKIN_INTERVAL_MS: u64 = _;
```

Time between check-ins: one week, in milliseconds.

### RING_DEPENDENCY_LEVEL

```rust
pub const RING_DEPENDENCY_LEVEL: LevelFilter = LevelFilter::INFO;
```

Level the diagnostics ring keeps for targets outside [`RING_TRACE_TARGETS`](#ring_trace_targets).

### RING_TRACE_TARGETS

```rust
pub const RING_TRACE_TARGETS: [&str; 1] = _;
```

Target prefixes the diagnostics ring keeps through TRACE. Every other
target (dependencies such as `h2`, `hyper`, `rustls`, `reqwest`, and
`log`-crate records) is kept from INFO up, so their per-frame TRACE events
do not crowd Bookclerk's own events out of the ring.

### MAX_EXTRA_COPIES

```rust
pub const MAX_EXTRA_COPIES: usize = 8;
```

Most copies a library may keep beside its main output.

### MAX_QUALITY_BYTES

```rust
pub const MAX_QUALITY_BYTES: usize = 16;
```

Longest quality name, in bytes (`320k-mono` is nine).

### MP3_CBR_KBPS

```rust
pub const MP3_CBR_KBPS: [u16; 7] = _;
```

The constant bitrates, in kbit/s, the MP3 encoder writes.

`bookclerk-media` rounds any other requested bitrate up to the next of
these (and anything above 256 to 320), so a quality name only offers
these rates: the name then always describes the audio written.

### OPUS_KBPS

```rust
pub const OPUS_KBPS: [u16; 7] = _;
```

The bitrates, in kbit/s, an Opus copy is named by. Opus encodes in
variable bitrate around the target, so these are targets, not caps.

### OPUS_MAIN_KBPS

```rust
pub const OPUS_MAIN_KBPS: u16 = 64;
```

The quality of the main output when it is Opus (`format = "opus"`):
64 kbit/s, keeping the source's channels up to stereo. Another bitrate takes an Opus extra copy beside an
M4B main output.

### DEFAULT_MAX_FILENAME_LENGTH

```rust
pub const DEFAULT_MAX_FILENAME_LENGTH: usize = 255;
```

Classic Libation `LongPath.MaxFilenameLength` — per path component.

### MAX_HOST_KEY_BYTES

```rust
pub const MAX_HOST_KEY_BYTES: usize = 1024;
```

Largest host key, in UTF-8 bytes, including the destination's prefix:
the S3 object-key limit, which file systems also accept.

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

### MAX_FILTER_RULES

```rust
pub const MAX_FILTER_RULES: usize = 64;
```

Most rules one destination filter may hold.

### MAX_FILTER_VALUE_BYTES

```rust
pub const MAX_FILTER_VALUE_BYTES: usize = 128;
```

Longest rule value, in UTF-8 bytes.

### REDACTED

```rust
pub const REDACTED: &str = "[REDACTED]";
```

Replacement token used wherever a secret or sensitive value is scrubbed.

### ACCESS_CODE_ALPHABET

```rust
pub const ACCESS_CODE_ALPHABET: &str = "0123456789ABCDEFGHJKMNPQRSTVWXYZ";
```

Crockford's base32 alphabet, which access codes are written in: digits
and letters without I, L, O and U.

### ACCESS_CODE_LENGTH

```rust
pub const ACCESS_CODE_LENGTH: usize = 16;
```

Characters in an access code, without the group separators.

### MAX_REPORT_HISTORY

```rust
pub const MAX_REPORT_HISTORY: usize = 200;
```

How many reports the history keeps; older ones are dropped first.

### MAX_SUBMIT_REPLY_BYTES

```rust
pub const MAX_SUBMIT_REPLY_BYTES: usize = 4096;
```

Longest collector reply, in bytes, that [`parse_submit_reply`](#parse_submit_reply) reads.
The real reply is about 200 bytes.

### REPORT_HISTORY_FILE

```rust
pub const REPORT_HISTORY_FILE: &str = "diagnostics/reports.json";
```

File, relative to the files directory, that holds the report history.

### MAX_LOOKUP_REPLY_BYTES

```rust
pub const MAX_LOOKUP_REPLY_BYTES: usize = _;
```

Longest collector answer, in bytes, that [`parse_lookup_reply`](#parse_lookup_reply) and
[`parse_delete_reply`](#parse_delete_reply) read. A stored report is at most 256 KiB, and the
answer caps every event, so a real answer is well under this.

### DEFAULT_UPGRADE_DAILY_LIMIT

```rust
pub const DEFAULT_UPGRADE_DAILY_LIMIT: u32 = 5;
```

Default for [`LibraryConfig::upgrade_daily_limit`](#libraryconfig).

### LETS_ENCRYPT_DIRECTORY

```rust
pub const LETS_ENCRYPT_DIRECTORY: &str = "https://acme-v02.api.letsencrypt.org/directory";
```

Let's Encrypt's production ACME directory.

### MAX_UPGRADE_DAILY_LIMIT

```rust
pub const MAX_UPGRADE_DAILY_LIMIT: u32 = 100;
```

Highest [`LibraryConfig::upgrade_daily_limit`](#libraryconfig) the settings accept.

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
