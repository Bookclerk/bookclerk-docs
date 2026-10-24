# bookclerk_plugin_catalog

Registry-neutral plugin catalog, manifests, and secure installer.

See `docs/plugin-registry.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`RegistryAdapter`](#registryadapter) | trait | Registry adapter trait (search / hydrate / versions). |
| [`CargoAdapter`](#cargoadapter) | struct | crates.io discovery + `.crate` metadata hydration. |
| [`NpmAdapter`](#npmadapter) | struct | npm registry adapter. |
| [`PypiAdapter`](#pypiadapter) | struct | PyPI JSON API adapter. Search is not supported by PyPI; use static registries for discovery and exact `pypi:name==version` coordinates for install. |
| [`StaticAdapter`](#staticadapter) | struct | Adapter over an in-memory or remote static index. |
| [`StaticIndex`](#staticindex) | struct | Top-level static index document. |
| [`StaticPackage`](#staticpackage) | struct | One package entry in a static registry index. |
| [`BundledSync`](#bundledsync) | struct | Outcome of `sync_bundled_platform` for one bundled platform artifact. |
| [`CatalogHit`](#cataloghit) | struct | One discovery hit from any registry adapter. |
| [`SearchQuery`](#searchquery) | struct | Search query passed to adapters. |
| [`PackageCoordinate`](#packagecoordinate) | struct | Exact package coordinate: source + name + version. |
| [`ArtifactIdentity`](#artifactidentity) | struct | Identity of one exact installed build: `PluginKey` + version + hashes. |
| [`PlatformArtifact`](#platformartifact) | struct | One Bookclerk-shipped platform package. |
| [`PluginInstallIdentity`](#plugininstallidentity) | struct | Evaluated identity for one discovered install directory. |
| [`PluginKey`](#pluginkey) | struct | Stable identity across upgrades (not version, not artifact hash). |
| [`InstallOptions`](#installoptions) | struct | Options for `Installer::install_from_manifest`. |
| [`InstallOutcome`](#installoutcome) | struct | Result of a successful (or dry-run) install. |
| [`Installer`](#installer) | struct | Installer: resolve → download → verify → extract → activate. |
| [`RuntimeIdentity`](#runtimeidentity) | struct | Runtime identity used for discovery and install collision checks. |
| [`InstallLedger`](#installledger) | struct | Host-owned map of PluginKey → expected digests for verified installs. |
| [`InstallLedgerEntry`](#installledgerentry) | struct | Host-owned record of one verified installed artifact. |
| [`ArtifactTarget`](#artifacttarget) | struct | One precompiled artifact for a Bookclerk target. |
| [`BookclerkPackageManifest`](#bookclerkpackagemanifest) | struct | Canonical install-grade package metadata. |
| [`PackageLinks`](#packagelinks) | struct | Optional documentation / support links. |
| [`PublisherIdentity`](#publisheridentity) | struct | Optional publisher signing identity (when available). |
| [`SandboxRequest`](#sandboxrequest) | struct | Publisher-requested sandbox (informational until host approves). |
| [`PluginMutationLock`](#pluginmutationlock) | struct | RAII exclusive lock covering one host plugin namespace (`$FILES_DIR`). |
| [`InstallReceipt`](#installreceipt) | struct | Record written after a successful install / update. |
| [`TrustPolicy`](#trustpolicy) | struct | Trust policy for plugin installation (digest-required artifacts). |
| [`Version`](#version) | struct | Parsed `major.minor.patch` with optional pre-release / build metadata. |
| [`BundledSyncAction`](#bundledsyncaction) | enum | What `sync_bundled_platform` did for one platform artifact. |
| [`RegistrySource`](#registrysource) | enum | Where a package was discovered / will be fetched from. |
| [`CatalogError`](#catalogerror) | enum | Failures from discovery, manifest validation, download, or installation. |
| [`PluginProvenance`](#pluginprovenance) | enum | How Bookclerk established (or lost) trust in an installed tree. |
| [`ProvenanceScheme`](#provenancescheme) | enum | Provenance scheme in a canonical `PluginKey`. |
| [`PluginKind`](#pluginkind) | enum | Which Bookclerk surface a plugin implements. |
| [`ArchiveFormat`](#archiveformat) | enum | Archive format for a target. |
| [`load_static_index`](#load_static_index) | function | Load a static index from a file path or `file://` URL. |
| [`bundled_plugins_dir_beside`](#bundled_plugins_dir_beside) | function | Bundled plugin directory for the host executable at `exe`, when it exists. |
| [`sync_bundled_platform`](#sync_bundled_platform) | function | Installs bundled platform plugins from `bundle` into `files_dir`. |
| [`evaluate_install`](#evaluate_install) | function | Evaluate identity + provenance for an install directory. |
| [`evaluate_install_in`](#evaluate_install_in) | function | Evaluate identity + provenance, consulting the host install ledger under `files_dir` when provided. |
| [`stamp_platform_receipt`](#stamp_platform_receipt) | function | Build a host-owned platform-bundled receipt **and** ledger row. |
| [`extract_archive`](#extract_archive) | function | Extract an archive into `dest`, refusing path traversal and oversized output. |
| [`require_mutable_child`](#require_mutable_child) | function | Lexical child path under `root` safe for `remove_dir_all` / `rename` / create. |
| [`require_under`](#require_under) | function | Returns `path` when it stays under `root` (canonical containment). |
| [`safe_join`](#safe_join) | function | Join `dest` / `rel` while rejecting absolute paths and `..` escapes. |
| [`sha256_bytes`](#sha256_bytes) | function | SHA-256 hex digest of bytes. |
| [`sha256_file`](#sha256_file) | function | SHA-256 hex digest of a file. |
| [`is_first_party_database_adapter`](#is_first_party_database_adapter) | function | True when `key` names a host-controlled first-party database adapter. |
| [`is_first_party_destination`](#is_first_party_destination) | function | True when `key` names a host-controlled first-party destination (`local` / `s3`). |
| [`is_first_party_integration`](#is_first_party_integration) | function | True when `key` names a host-controlled first-party integration. |
| [`is_platform_plugin_key`](#is_platform_plugin_key) | function | True when `key` is a known Bookclerk platform `PluginKey`. |
| [`platform_artifact`](#platform_artifact) | function | Looks up a known platform artifact by package name + manifest id. |
| [`fetch_artifact`](#fetch_artifact) | function | Fetches an artifact URL (`file://` or local path, or HTTPS/localhost HTTP bounded by `MAX_DOWNLOAD_BYTES` and `DOWNLOAD_TIMEOUT`) into `dest`, refusing oversized bodies. |
| [`record_install`](#record_install) | function | Records (or updates) a host-verified install in the files-dir ledger. |
| [`restore_ledger_entry`](#restore_ledger_entry) | function | Restores `previous` for `plugin_key`, or deletes the row when `previous` is `None`. |
| [`normalize_protocol`](#normalize_protocol) | function | Normalize optional `protocol` for Workers RPC packages. |
| [`parse_sha256_hex`](#parse_sha256_hex) | function | Decode hex SHA-256 into 32 bytes. |
| [`validate_sha256_hex`](#validate_sha256_hex) | function | Validate lowercase hex SHA-256 (64 chars). |
| [`acquire_plugins_lock`](#acquire_plugins_lock) | function | Acquires the host mutation lock for `plugins_root`'s parent files dir. |
| [`check_owner_artifact_url`](#check_owner_artifact_url) | function | Checks a URL an owner cluster asks this host to fetch an archive from. |
| [`fetch_owner_artifact`](#fetch_owner_artifact) | function | Fetches an owner-chosen `https:` archive into `dest`, bounded by `MAX_DOWNLOAD_BYTES` and `DOWNLOAD_TIMEOUT` per request. Redirects are followed (at most `MAX_REDIRECTS`) only when each hop passes `check_owner_artifact_url`, and every hop connects only to addresses that pass `owner_address_allowed` (`PublicOnlyResolver`); no proxy is used. The caller checks the archive digest. |
| [`manifest_sha256`](#manifest_sha256) | function | SHA-256 of `plugin.toml` at `plugin_root`. |
| [`payload_root_sha256`](#payload_root_sha256) | function | Deterministic payload root: sorted relative path + type + content hash. |
| [`host_bookclerk_target`](#host_bookclerk_target) | function | Host Bookclerk target for the machine compiling this code. |
| [`normalize_target`](#normalize_target) | function | Map a rustc triple (or Bookclerk target) to a Bookclerk target name. |
| [`rust_triple`](#rust_triple) | function | Rustc triple corresponding to a Bookclerk target (for packaging). |
| [`select_target`](#select_target) | function | Select an artifact entry matching the host (or override) target. |
| [`max_version`](#max_version) | function | Prefer the greatest release version; if only pre-releases exist, the greatest pre-release. Falls back to lexical max when nothing parses. |
| [`newest_newer_than`](#newest_newer_than) | function | Newest version strictly greater than `current`, if any. |
| [`federated_search`](#federated_search) | function | Federated search across configured adapters (static first, then cargo/npm). |
| [`fetch_manifest_for_coordinate`](#fetch_manifest_for_coordinate) | function | Resolve a coordinate to a manifest using the matching adapter. |
| [`list_versions_for_coordinate`](#list_versions_for_coordinate) | function | List published versions for a coordinate's package name. |
| [`resolve_newer_version`](#resolve_newer_version) | function | Resolve the newest version newer than `coord.version`, if any. |
| [`Result`](#result) | type alias | Result alias for `CatalogError`. |
| [`BUNDLED_PLUGINS_DIR`](#bundled_plugins_dir) | constant | Directory beside the host executable that holds bundled platform plugins. |
| [`CATALOG_DTO_SCHEMA_VERSION`](#catalog_dto_schema_version) | constant | Schema version for CLI JSON catalog output. |
| [`MAX_ARCHIVE_BYTES`](#max_archive_bytes) | constant | Hard cap on archive file size before extraction. |
| [`MAX_EXTRACTED_BYTES`](#max_extracted_bytes) | constant | Hard cap on total extracted archive bytes. |
| [`CRATES_IO_INDEX`](#crates_io_index) | constant | Default Cargo registry origin used in canonical `cargo:` PluginKeys. |
| [`FIRST_PARTY_DATABASE_ADAPTERS`](#first_party_database_adapters) | constant | Host-controlled first-party database adapter identities. |
| [`FIRST_PARTY_DESTINATIONS`](#first_party_destinations) | constant | Host-controlled first-party destination identities (`local`, `s3`). |
| [`PLATFORM_ARTIFACTS`](#platform_artifacts) | constant | Known installer-shipped platform packages. Host-controlled; never read from plugin-authored manifest fields other than matching `id` after provenance is already established. |
| [`PLATFORM_PRODUCT`](#platform_product) | constant | Product name recorded on Bookclerk-shipped platform artifacts. |
| [`PLUGIN_KEY_FS_ID_HEX_CHARS`](#plugin_key_fs_id_hex_chars) | constant | Hex characters of SHA-256 used in `PluginKey::fs_id` (128 bits). |
| [`DOWNLOAD_TIMEOUT`](#download_timeout) | constant | Download / install limits. |
| [`MAX_DOWNLOAD_BYTES`](#max_download_bytes) | constant | Hard cap on downloaded artifact size in bytes. |
| [`PLUGIN_HOLD_DIR`](#plugin_hold_dir) | constant | Host-owned directory under `$FILES_DIR` for held plugin-state during remove. |
| [`INSTALL_LEDGER_FILE`](#install_ledger_file) | constant | Filename under `$BOOKCLERK_FILES_DIR` for the host install ledger. |
| [`INSTALL_LEDGER_SCHEMA_VERSION`](#install_ledger_schema_version) | constant | Schema version for `InstallLedger`. |
| [`MANIFEST_SCHEMA_VERSION`](#manifest_schema_version) | constant | Current package manifest schema. |
| [`PROTOCOL_WORKERS_RPC`](#protocol_workers_rpc) | constant | Product wire protocol for Workers RPC (`api_version = 3`). |
| [`PLUGIN_MUTATION_LOCK_FILE`](#plugin_mutation_lock_file) | constant | Filename under `$BOOKCLERK_FILES_DIR` for the host plugin mutation lock. |
| [`PAYLOAD_SKIP_NAMES`](#payload_skip_names) | constant | Directory / file names excluded from the immutable payload (host-owned receipts only). Packaged `data/` and `tmp/` are hashed: mutable plugin state lives under `$FILES_DIR/plugin-state/<PluginKey fs-id>/`, not inside the install tree. |
| [`RECEIPT_FILE`](#receipt_file) | constant | Filename for the receipt beside `plugin.toml`. |
| [`TARGETS`](#targets) | constant | Canonical Bookclerk install targets (not raw rustc triples). |

## Traits

### RegistryAdapter

```rust
pub trait RegistryAdapter: Send + Sync {
    fn source_kind(&self) -> &'static str;
    fn search(&self, q: &SearchQuery) -> Result<Vec<CatalogHit>>;
    fn fetch_manifest(&self, coord: &PackageCoordinate) -> Result<BookclerkPackageManifest>;
    fn list_versions(&self, name: &str) -> Result<Vec<String>>;
}
```

Registry adapter trait (search / hydrate / versions).

#### RegistryAdapter::source_kind

```rust
fn source_kind(&self) -> &'static str
```

Returns the adapter source kind string (`cargo`, `npm`, `pypi`, `static`).

#### RegistryAdapter::search

```rust
fn search(&self, q: &SearchQuery) -> Result<Vec<CatalogHit>>
```

Searches this registry for packages matching `query`.

##### Errors

Returns an error when the operation fails.

#### RegistryAdapter::fetch_manifest

```rust
fn fetch_manifest(&self, coord: &PackageCoordinate) -> Result<BookclerkPackageManifest>
```

Fetches and parses the Bookclerk package manifest for `coordinate`.

##### Errors

Returns an error when the operation fails.

#### RegistryAdapter::list_versions

```rust
fn list_versions(&self, name: &str) -> Result<Vec<String>>
```

Lists known versions for `name` in this registry.

##### Errors

Returns an error when the operation fails.

Implemented by: `CargoAdapter`, `NpmAdapter`, `PypiAdapter`, `StaticAdapter`

## Structs

### CargoAdapter

```rust
pub struct CargoAdapter {
    pub registry_url: String,
}
```

crates.io discovery + `.crate` metadata hydration.

**Fields**

- `registry_url`: Base URL of the package registry (for example crates.io or npm).

Implements: `Default`, `RegistryAdapter`

### NpmAdapter

```rust
pub struct NpmAdapter {
    pub registry_url: String,
}
```

npm registry adapter.

**Fields**

- `registry_url`: Base URL of the package registry (for example crates.io or npm).

Implements: `Default`, `RegistryAdapter`

### PypiAdapter

```rust
pub struct PypiAdapter {
    pub base_url: String,
}
```

PyPI JSON API adapter. Search is not supported by PyPI; use static registries
for discovery and exact `pypi:name==version` coordinates for install.

**Fields**

- `base_url`: PyPI simple/API base URL for this adapter.

Implements: `Default`, `RegistryAdapter`

### StaticAdapter

```rust
pub struct StaticAdapter {
    pub index_url: String,
    /* private fields */
}
```

Adapter over an in-memory or remote static index.

**Fields**

- `index_url`: Absolute URL of the static registry index document.

#### StaticAdapter::open

```rust
pub fn open(index_url: impl Into<String>) -> Result<Self>
```

Downloads and parses a static registry index from `index_url`.

##### Errors

Returns an error when the operation fails.

#### StaticAdapter::from_index

```rust
pub fn from_index(index_url: impl Into<String>, index: StaticIndex) -> Self
```

Wraps an already-parsed static index without fetching.

Implements: `RegistryAdapter`

### StaticIndex

```rust
pub struct StaticIndex {
    pub schema_version: u32,
    pub packages: Vec<StaticPackage>,
}
```

Top-level static index document.

**Fields**

- `schema_version`: DTO schema version for CLI/UI JSON compatibility.
- `packages`: Packages listed in this static registry index.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### StaticPackage

```rust
pub struct StaticPackage {
    pub name: String,
    pub versions: BTreeMap<String, BookclerkPackageManifest>,
}
```

One package entry in a static registry index.

**Fields**

- `name`: Package name within `source`.
- `versions`: Known versions and manifests for this static package.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Serialize`

### BundledSync

```rust
pub struct BundledSync {
    pub artifact: &'static PlatformArtifact,
    pub action: BundledSyncAction,
}
```

Outcome of [`sync_bundled_platform`](#sync_bundled_platform) for one bundled platform artifact.

**Fields**

- `artifact`: The platform package this row is about.
- `action`: What was done.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CatalogHit

```rust
pub struct CatalogHit {
    pub schema_version: u32,
    pub coordinate: Option<PackageCoordinate>,
    pub source_kind: String,
    pub package_name: String,
    pub version: String,
    pub description: Option<String>,
    pub downloads: Option<u64>,
    pub repository: Option<String>,
    pub homepage: Option<String>,
    pub documentation: Option<String>,
    pub runtime: Option<RuntimeIdentity>,
    pub manifest: Option<BookclerkPackageManifest>,
}
```

One discovery hit from any registry adapter.

**Fields**

- `schema_version`: DTO schema version for CLI/UI JSON compatibility.
- `coordinate`: Source-qualified coordinate when version is known; otherwise name-only hint.
- `source_kind`: Registry adapter kind (`cargo`, `npm`, `pypi`, `static`, …).
- `package_name`: Package name in the source registry.
- `version`: Resolved or candidate package version string.
- `description`: Short package description from the registry, when present.
- `downloads`: Download count from the registry index, when known.
- `repository`: Source repository URL from package metadata, when present.
- `homepage`: Project homepage URL from package metadata, when present.
- `documentation`: Documentation URL from package metadata, when present.
- `runtime`: Bookclerk runtime identity (kind + plugin id) when known.
- `manifest`: Present after metadata hydration / static index lookup.

#### CatalogHit::from_cargo_search

```rust
pub fn from_cargo_search(
    crate_name: impl Into<String>,
    version: impl Into<String>,
    kind: Option<PluginKind>,
    id: Option<String>,
    description: Option<String>,
    downloads: u64,
    repository: Option<String>,
    homepage: Option<String>,
    documentation: Option<String>,
) -> Self
```

Build a hit from a crates.io-shaped search result.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SearchQuery

```rust
pub struct SearchQuery {
    pub text: Option<String>,
    pub limit: u32,
}
```

Search query passed to adapters.

**Fields**

- `text`: Free-text search string passed to registry adapters.
- `limit`: Maximum hits to return (clamped by the caller).

Implements: `Clone`, `Debug`, `Default`

### PackageCoordinate

```rust
pub struct PackageCoordinate {
    pub source: RegistrySource,
    pub name: String,
    pub version: String,
}
```

Exact package coordinate: source + name + version.

Display forms:
- `cargo:bookclerk-plugin-source-example@1.2.3`
- `npm:@publisher/bookclerk-plugin-source-example@1.2.3`
- `pypi:bookclerk-plugin-source-example==1.2.3`
- `registry:https://example.com/index.json#community/example@1.2.3`
- `local:/path/to/archive.tar.gz` (version may be `0.0.0` placeholder)

**Fields**

- `source`: Registry source (crates.io, npm, static index URL, …).
- `name`: Package name within `source`.
- `version`: Resolved or candidate package version string.

#### PackageCoordinate::parse

```rust
pub fn parse(raw: &str) -> Result<Self>
```

Parse a source-qualified coordinate string.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `PartialEq`, `Serialize`

### ArtifactIdentity

```rust
pub struct ArtifactIdentity {
    pub plugin_key: PluginKey,
    pub version: String,
    pub manifest_sha256: String,
    pub payload_root_sha256: String,
    pub archive_sha256: Option<String>,
}
```

Identity of one exact installed build: [`PluginKey`](#pluginkey) + version + hashes.

**Fields**

- `plugin_key`: Logical plugin (stable across upgrades).
- `version`: Package / manifest version string.
- `manifest_sha256`: SHA-256 of the installed `plugin.toml` bytes.
- `payload_root_sha256`: Deterministic Merkle-style root over the immutable installed payload.
- `archive_sha256`: SHA-256 of the downloaded archive, when the install came from a registry.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PlatformArtifact

```rust
pub struct PlatformArtifact {
    pub package_name: &'static str,
    pub manifest_id: &'static str,
    pub allowed_entrypoints: &'static [&'static str],
    pub allowed_bindings: &'static [&'static str],
}
```

One Bookclerk-shipped platform package.

**Fields**

- `package_name`: Cargo package name staged by `install-platform`.
- `manifest_id`: Manifest alias (`sqlite`, `local`).
- `allowed_entrypoints`: Entrypoints the installer envelope permits.
- `allowed_bindings`: Host bindings the installer envelope permits.

#### PlatformArtifact::plugin_key

```rust
pub fn plugin_key(self) -> PluginKey
```

[`PluginKey`](#pluginkey) for this platform package.

##### Panics

Panics if a baked-in [`PLATFORM_ARTIFACTS`](#platform_artifacts) package name or manifest id
fails the PluginKey grammar. Those rows are compile-time constants.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### PluginInstallIdentity

```rust
pub struct PluginInstallIdentity {
    pub plugin_key: PluginKey,
    pub alias: String,
    pub artifact: ArtifactIdentity,
    pub provenance: PluginProvenance,
}
```

Evaluated identity for one discovered install directory.

**Fields**

- `plugin_key`: Stable logical identity (provenance + package; not the alias).
- `alias`: Human-facing alias from `plugin.toml` `id`.
- `artifact`: Exact installed build.
- `provenance`: Host-evaluated provenance (never plugin-authored).

#### PluginInstallIdentity::alias

```rust
pub fn alias(&self) -> &str
```

Display alias (`plugin.toml` id). Never a security principal.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginKey

```rust
pub struct PluginKey {
    /* private fields */
}
```

Stable identity across upgrades (not version, not artifact hash).

Canonical text (deterministic, unambiguous). Registry index URL and
package name are `#`-separated so a registry path cannot be confused
with a package path. The human alias is **not** included (it lives on
the manifest / receipt `runtime.id`):

- `cargo:{index_url}#{package}`
- `npm:{index_url}#{package}`
- `pypi:{index_url}#{package}`
- `registry:{index_url}#{package}`
- `platform:bookclerk/{package}`
- `path:file://{normalized_absolute_path}`

#### PluginKey::parse

```rust
pub fn parse(raw: &str) -> Result<Self>
```

Builds a key from already-canonical text. Prefer the typed constructors.

##### Errors

Returns an error when the canonical form cannot be parsed.

#### PluginKey::platform

```rust
pub fn platform(package_name: &str, manifest_id: &str) -> Result<Self>
```

Platform-bundled key (`platform:bookclerk/{package}`).

`manifest_id` is validated as a plugin alias but is not part of the key.

##### Errors

Returns an error when `manifest_id` fails the plugin id grammar.

#### PluginKey::from_coordinate

```rust
pub fn from_coordinate(coordinate: &PackageCoordinate, manifest_id: &str) -> Result<Self>
```

Registry-sourced key (Cargo / npm / PyPI / static index).

Version and the human alias are **not** part of the key. `manifest_id`
is validated as a plugin alias only.

##### Errors

Returns an error when the coordinate or id is invalid.

#### PluginKey::from_install_path

```rust
pub fn from_install_path(root: &Path, manifest_id: &str) -> Result<Self>
```

Filesystem / development-tree key for an install directory.

##### Errors

Returns an error when the path cannot be made into a `file://` URL or
the id is invalid.

#### PluginKey::canonical

```rust
pub fn canonical(&self) -> &str
```

Canonical textual form.

#### PluginKey::digest_sha256

```rust
pub fn digest_sha256(&self) -> String
```

SHA-256 (hex) of the canonical UTF-8 bytes.

#### PluginKey::fs_id

```rust
pub fn fs_id(&self) -> String
```

Filesystem-safe directory leaf (`pk-` + 32 hex chars / 128 bits of SHA-256).

#### PluginKey::scheme

```rust
pub fn scheme(&self) -> ProvenanceScheme
```

Provenance scheme.

##### Panics

Panics if the in-memory canonical string is not a valid PluginKey.
That is a programming error: every [`PluginKey`](#pluginkey) is produced by
[`Self::parse`](#pluginkeyparse) or a typed constructor that goes through it.

#### PluginKey::package

```rust
pub fn package(&self) -> String
```

Package coordinate / name when present (empty for `path:` keys).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `FromStr`, `Hash`, `PartialEq`, `Serialize`

### InstallOptions

```rust
pub struct InstallOptions {
    pub plugins_root: PathBuf,
    pub target: Option<String>,
    pub dry_run: bool,
    pub replace: bool,
    pub offline: bool,
    pub trust: TrustPolicy,
    pub skip_health: bool,
    pub approve_capabilities: bool,
}
```

Options for [`Installer::install_from_manifest`](#installerinstall_from_manifest).

**Fields**

- `plugins_root`: Directory under which plugin install folders are created.
- `target`: Host target triple used to select release artifacts.
- `dry_run`: When true, resolve and verify without writing an install.
- `replace`: When true, overwrite an existing install of the **same** [`PluginKey`](#pluginkey)
  at the **same** alias.
  
  Does not allow a different PluginKey to seize an already-used alias,
  and does not rename an installed PluginKey's alias. Alias changes are
  not supported by ordinary install/update/replace.
- `offline`: When true, refuse network fetches (local/cache only).
- `trust`: Trust policy applied to community packages (digest still required).
- `skip_health`: Skip health spawn (caller runs health separately).
- `approve_capabilities`: When true, persist a covering consent grant after install.

Implements: `Clone`, `Debug`, `Default`

### InstallOutcome

```rust
pub struct InstallOutcome {
    pub plugin_root: PathBuf,
    pub receipt: InstallReceipt,
    pub dry_run: bool,
    pub previous: Option<PathBuf>,
    pub previous_ledger: Option<InstallLedgerEntry>,
    pub files_dir: Option<PathBuf>,
}
```

Result of a successful (or dry-run) install.

**Fields**

- `plugin_root`: Filesystem path of the installed plugin directory.
- `receipt`: Install receipt written under the plugin root after activation.
- `dry_run`: When true, resolve and verify without writing an install.
- `previous`: Previous install kept aside until [`Installer::commit`](#installercommit) (or restored by
  [`Installer::rollback`](#installerrollback)). Present only for replace installs.
- `previous_ledger`: Ledger row that existed for this PluginKey before activation, if any.
- `files_dir`: Files directory that owns `install-ledger.json` (`plugins_root` parent).

Implements: `Clone`, `Debug`

### Installer

```rust
pub struct Installer;
```

Installer: resolve → download → verify → extract → activate.

#### Installer::plugin_key_for

```rust
pub fn plugin_key_for(
    coordinate: &PackageCoordinate,
    runtime_id: &str,
    plugins_root: &Path,
) -> Result<PluginKey>
```

PluginKey this installer would assign for `coordinate` + runtime alias.

Host/CLI alias preflight uses the same identity the activate step will
write so a foreign occupant cannot be compared against a different key.

##### Errors

Returns when the coordinate cannot form a canonical [`PluginKey`](#pluginkey).

#### Installer::local_archive_coordinate

```rust
pub fn local_archive_coordinate(
    archive: &Path,
    manifest: &BookclerkPackageManifest,
) -> PackageCoordinate
```

Coordinate used when installing from a local archive path.

#### Installer::install_from_manifest

```rust
pub fn install_from_manifest(
    manifest: &BookclerkPackageManifest,
    coordinate: &PackageCoordinate,
    opts: &InstallOptions,
) -> Result<InstallOutcome>
```

Install from an already-validated package manifest (fixture / adapter output).

##### Arguments

* `manifest` - Validated Bookclerk package manifest for this version.
* `coordinate` - Source-qualified coordinate being installed.
* `opts` - Plugins root, target, dry-run, replace, trust, and consent flags.

##### Returns

[`InstallOutcome`](#installoutcome) with plugin root, receipt, and optional previous install.

##### Errors

Returns [`CatalogError`](#catalogerror) when validation, download, digest, extract, or
activation fails (or when offline/trust policy refuses the package).

#### Installer::install_from_manifest_with_lock

```rust
pub fn install_from_manifest_with_lock(
    lock: &PluginMutationLock,
    manifest: &BookclerkPackageManifest,
    coordinate: &PackageCoordinate,
    opts: &InstallOptions,
) -> Result<InstallOutcome>
```

Install while already holding the host-local plugin mutation lock.

Production callers that also run configured-discovery preflight, health,
and commit/rollback must acquire [`PluginMutationLock`](#pluginmutationlock) once for
`$FILES_DIR` and pass it here so the lock covers the full local
transaction. Do not acquire a second lock around health.

##### Errors

Returns when `lock` does not cover `opts.plugins_root`, or when
validation, download, digest, extract, or activation fails.

#### Installer::commit

```rust
pub fn commit(outcome: &InstallOutcome) -> Result<()>
```

Discard a replace backup after a successful health check (or when none).

The new plugin tree and ledger row are already active before this runs.
Failure here is an operational cleanup error: the installation succeeded,
but a previous-version backup may remain under `plugins/.staging`. Callers
must surface that and must not claim a fully clean success.

Retry-safe: a missing backup is treated as already cleaned up.

##### Errors

Returns when the leftover backup directory cannot be deleted.

#### Installer::rollback

```rust
pub fn rollback(outcome: &InstallOutcome) -> Result<()>
```

Restore the previous install (or delete a failed first install) after a
failed health check.

Rollback is **monotonic and retry-safe**:

* Never delete `plugin_root` unless a usable previous-version backup
  exists that can replace it.
* If the backup was already consumed and the destination contains the old
  tree, treat the tree phase as done and retry remaining ledger repair.
* First-install rollback may delete the new tree; a missing destination
  on retry is success for the tree phase.

##### Errors

Returns when the tree cannot be restored/removed or the ledger cannot
be repaired. A failed call must not destroy an already-restored tree.

#### Installer::install_local_archive

```rust
pub fn install_local_archive(
    archive: &Path,
    manifest: &BookclerkPackageManifest,
    opts: &InstallOptions,
) -> Result<InstallOutcome>
```

Install from a local archive path using an explicit manifest.

##### Errors

Returns an error when the operation fails.

#### Installer::install_local_archive_with_lock

```rust
pub fn install_local_archive_with_lock(
    lock: &PluginMutationLock,
    archive: &Path,
    manifest: &BookclerkPackageManifest,
    opts: &InstallOptions,
) -> Result<InstallOutcome>
```

Local-archive install while already holding the host mutation lock.

##### Errors

Returns when `lock` does not cover `opts.plugins_root` or install fails.

#### Installer::remove

```rust
pub fn remove(plugins_root: &Path, spec: &str, purge_state: bool) -> Result<()>
```

Remove an installed plugin directory; optionally purge `plugin-state/`.

`spec` is a canonical PluginKey or an alias unique in this host plugin
namespace. Duplicate aliases on disk fail closed.

Removal binds durable identity from **host-owned evidence plus
filesystem placement** before any mutation: parseable `plugin.toml`
alias, the unique matching `install-ledger.json` row, and (for
`pk-*` directories) `basename == PluginKey.fs_id()`. A syntactically
valid `receipt.json` PluginKey is metadata and must agree; it cannot
redirect remove or `--purge-state` onto another PluginKey.

`--purge-state` **holds** `plugin-state/<fs-id>` under
`$FILES_DIR/.plugin-hold/` until commit. Rollback restores tree, state,
and ledger. Commit deletes the held tree then the held state.

##### Errors

Returns when the plugin is missing, durable identity cannot be
established, the trust ledger cannot be read or written, or the tree
or state cannot be held, restored, or finalized.

#### Installer::remove_with_lock

```rust
pub fn remove_with_lock(
    lock: &PluginMutationLock,
    plugins_root: &Path,
    spec: &str,
    purge_state: bool,
) -> Result<()>
```

Remove while already holding the host-local plugin mutation lock.

##### Errors

Returns when `lock` does not cover `plugins_root` or remove fails.

#### Installer::remove_with_lock_preparing_purge

```rust
pub fn remove_with_lock_preparing_purge<F>(
    lock: &PluginMutationLock,
    plugins_root: &Path,
    spec: &str,
    purge_state: bool,
    prep: F,
) -> Result<()>
where
    F: FnMut(&Path) -> Result<(), String>,
```

Remove, and run `prep` on an existing plugin-state directory before purge.

`prep` revokes host ACL journals. If it fails, the install tree is
restored and plugin-state is left in place.

##### Errors

Returns when `lock` does not cover `plugins_root`, `prep` fails, or
remove fails.

### RuntimeIdentity

```rust
pub struct RuntimeIdentity {
    pub kind: PluginKind,
    pub id: String,
}
```

Runtime identity used for discovery and install collision checks.

**Fields**

- `kind`: Destination backend kind (`local`, `s3`, …).
- `id`: Plugin alias (`[a-z0-9_]{2,32}`), unique within one host plugin namespace.

#### RuntimeIdentity::new

```rust
pub fn new(kind: PluginKind, id: impl Into<String>) -> Self
```

Builds a runtime identity from plugin kind and id.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### InstallLedger

```rust
pub struct InstallLedger {
    pub schema_version: u32,
    pub artifacts: Vec<InstallLedgerEntry>,
}
```

Host-owned map of PluginKey → expected digests for verified installs.

**Fields**

- `schema_version`: DTO schema version.
- `artifacts`: One row per PluginKey (last write wins).

#### InstallLedger::path

```rust
pub fn path(files_dir: &Path) -> PathBuf
```

Absolute path to `install-ledger.json` under `files_dir`.

#### InstallLedger::load

```rust
pub fn load(files_dir: &Path) -> Result<Self>
```

Loads the ledger, or an empty ledger when the file is missing.

##### Errors

Returns an error when the file exists but cannot be read or parsed,
or when `files_dir` contains `..` / cannot be contained.
A missing file (or missing files dir) is not an error.

#### InstallLedger::get

```rust
pub fn get(&self, plugin_key: &PluginKey) -> Option<&InstallLedgerEntry>
```

Lookup by canonical PluginKey.

#### InstallLedger::upsert

```rust
pub fn upsert(&mut self, entry: InstallLedgerEntry)
```

Inserts or replaces the row for `entry.plugin_key`.

#### InstallLedger::remove

```rust
pub fn remove(&mut self, plugin_key: &PluginKey) -> bool
```

Removes the row for `plugin_key` if present.

#### InstallLedger::store

```rust
pub fn store(&self, files_dir: &Path) -> Result<()>
```

Atomically writes the ledger under `files_dir`.

##### Errors

Returns when the directory cannot be created or the file cannot be written.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### InstallLedgerEntry

```rust
pub struct InstallLedgerEntry {
    pub plugin_key: String,
    pub package_name: String,
    pub manifest_id: String,
    pub manifest_sha256: String,
    pub payload_root_sha256: String,
    pub provenance: PluginProvenance,
    pub recorded_at: DateTime<Utc>,
}
```

Host-owned record of one verified installed artifact.

**Fields**

- `plugin_key`: Canonical [`PluginKey`](#pluginkey).
- `package_name`: Package coordinate name recorded at install/stage time.
- `manifest_id`: Manifest alias (`sqlite`, `local`, …).
- `manifest_sha256`: SHA-256 of the installed `plugin.toml` bytes.
- `payload_root_sha256`: Deterministic payload root over immutable packaged files.
- `provenance`: Host-evaluated provenance at the time the ledger row was written.
- `recorded_at`: RFC 3339 time when the host recorded this artifact.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ArtifactTarget

```rust
pub struct ArtifactTarget {
    pub target: String,
    pub url: String,
    pub archive_sha256: String,
    pub archive_root: String,
    pub executable: String,
    pub executable_sha256: Option<String>,
}
```

One precompiled artifact for a Bookclerk target.

**Fields**

- `target`: Bookclerk target (`linux-x64-gnu`, …) or legacy rustc triple.
- `url`: HTTPS download URL (or `file:` for fixtures).
- `archive_sha256`: Lowercase hex SHA-256 of the archive bytes (required for unattended install).
- `archive_root`: Path inside the archive to the plugin root (default `.`).
- `executable`: Relative path to the executable inside the archive root.
- `executable_sha256`: Optional SHA-256 of the extracted executable.

#### ArtifactTarget::bookclerk_target

```rust
pub fn bookclerk_target(&self) -> String
```

Bookclerk target id selected for this host.

#### ArtifactTarget::archive_format

```rust
pub fn archive_format(&self) -> ArchiveFormat
```

Archive format for this artifact (`tar.gz` or `zip`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### BookclerkPackageManifest

```rust
pub struct BookclerkPackageManifest {
    pub schema_version: u32,
    pub protocol: Option<String>,
    pub api_version: u32,
    pub api_version_max: Option<u32>,
    pub min_bookclerk: Option<String>,
    pub kind: PluginKind,
    pub id: String,
    pub display_name: Option<String>,
    pub description: Option<String>,
    pub coordinate: Option<PackageCoordinate>,
    pub artifacts: Vec<ArtifactTarget>,
    pub sandbox: SandboxRequest,
    pub links: PackageLinks,
    pub yanked: bool,
    pub released_at: Option<String>,
    pub publisher: Option<PublisherIdentity>,
}
```

Canonical install-grade package metadata.

**Fields**

- `schema_version`: DTO schema version for CLI/UI JSON compatibility.
- `protocol`: Optional wire label. Prefer absent; when present use `workers-rpc`.
- `api_version`: Plugin ABI version negotiated with the host.
- `api_version_max`: Maximum plugin ABI version this package claims to support.
- `min_bookclerk`: Optional minimum Bookclerk host version required to install/run.
- `kind`: Destination backend kind (`local`, `s3`, …).
- `id`: Plugin alias (`[a-z0-9_]{2,32}`), unique within one host plugin namespace.
- `display_name`: Optional human-readable name for catalog and UI.
- `description`: Short package description from the registry, when present.
- `coordinate`: Filled by adapters; may be omitted in static index entries that nest under name/version.
- `artifacts`: Per-target downloadable artifacts for this package version.
- `sandbox`: Publisher-requested sandbox surface (informational until approved).
- `links`: Optional documentation and support links.
- `yanked`: When true, this version was yanked and must not be installed unattended.
- `released_at`: RFC 3339 release timestamp from the registry, when known.
- `publisher`: Optional publisher identity for trust decisions.

#### BookclerkPackageManifest::runtime

```rust
pub fn runtime(&self) -> RuntimeIdentity
```

Returns the [`RuntimeIdentity`](#runtimeidentity) (kind + id) for this package.

#### BookclerkPackageManifest::effective_protocol

```rust
pub fn effective_protocol(&self) -> String
```

Effective protocol string for receipts / wire defaults.

#### BookclerkPackageManifest::validate_for_install

```rust
pub fn validate_for_install(&self) -> Result<()>
```

Validate schema + require digests for install-grade manifests.

##### Errors

Returns an error when the operation fails.

#### BookclerkPackageManifest::from_json

```rust
pub fn from_json(text: &str) -> Result<Self>
```

Parses a package manifest from a JSON document.

##### Errors

Returns an error when the operation fails.

#### BookclerkPackageManifest::from_toml

```rust
pub fn from_toml(text: &str) -> Result<Self>
```

Parse from TOML (`[package.metadata.bookclerk]` style flat table or root).

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PackageLinks

```rust
pub struct PackageLinks {
    pub repository: Option<String>,
    pub documentation: Option<String>,
    pub homepage: Option<String>,
    pub license: Option<String>,
    pub support: Option<String>,
}
```

Optional documentation / support links.

**Fields**

- `repository`: Source repository URL from package metadata, when present.
- `documentation`: Documentation URL from package metadata, when present.
- `homepage`: Project homepage URL from package metadata, when present.
- `license`: SPDX or human-readable license string from package metadata.
- `support`: Support / issue-tracker URL from package metadata, when present.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PublisherIdentity

```rust
pub struct PublisherIdentity {
    pub name: Option<String>,
    pub url: Option<String>,
    pub key_id: Option<String>,
}
```

Optional publisher signing identity (when available).

**Fields**

- `name`: Package name within `source`.
- `url`: HTTPS URL for this resource (artifact, publisher, …).
- `key_id`: Publisher signing key identifier when signatures are present.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SandboxRequest

```rust
pub struct SandboxRequest {
    pub network: String,
}
```

Publisher-requested sandbox (informational until host approves).

**Fields**

- `network`: Requested guest network mode (`deny` or `outbound`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginMutationLock

```rust
pub struct PluginMutationLock {
    /* private fields */
}
```

RAII exclusive lock covering one host plugin namespace (`$FILES_DIR`).

Hold this guard for the full local mutation transaction: configured alias
preflight, ledger read, tree/state hold, ledger write, health, and
commit or rollback. Drop (or process exit) releases the OS lock.

Binding the guard is required: `PluginMutationLock::acquire(dir)?;` without
an owning binding drops the lock immediately.

#### PluginMutationLock::path

```rust
pub fn path(files_dir: &Path) -> PathBuf
```

Absolute path to `.plugin-mutation.lock` under `files_dir`.

#### PluginMutationLock::acquire

```rust
pub fn acquire(files_dir: &Path) -> Result<Self>
```

Acquires an exclusive inter-process lock for `files_dir`.

Blocks until the lock is available. Creates `files_dir` and the lock
file when missing. Does not take a shared-database or cluster lock.

Operator-selected roots may contain `..` spelling; they are resolved via
canonicalize. Escaping or dangling leaf symlinks at the lock path are
refused via [`require_under`](#require_under) before open (not a claim that check-then-open
is race-free).

##### Errors

Returns when `files_dir` cannot be created, the lock path escapes the
root, the lock file cannot be opened, or the OS refuses the advisory lock.

#### PluginMutationLock::files_dir

```rust
pub fn files_dir(&self) -> &Path
```

Host files directory this lock owns.

#### PluginMutationLock::covers_plugins_root

```rust
pub fn covers_plugins_root(&self, plugins_root: &Path) -> bool
```

True when `plugins_root` is the `plugins/` directory under this lock.

#### PluginMutationLock::require_plugins_root

```rust
pub fn require_plugins_root(&self, plugins_root: &Path) -> Result<()>
```

Rejects a plugins root that is not this lock's host namespace.

##### Errors

Returns when `plugins_root` is not `$FILES_DIR/plugins` for this lock.

Implements: `Debug`

### InstallReceipt

```rust
pub struct InstallReceipt {
    pub schema_version: u32,
    pub plugin_key: String,
    pub provenance: PluginProvenance,
    pub coordinate: PackageCoordinate,
    pub version: String,
    pub registry_url: Option<String>,
    pub artifact_url: String,
    pub target: String,
    pub archive_sha256: String,
    pub manifest_sha256: String,
    pub payload_root_sha256: String,
    pub executable_sha256: Option<String>,
    pub protocol: String,
    pub api_version: u32,
    pub runtime: RuntimeIdentity,
    pub requested_sandbox: SandboxRequest,
    pub approved_network: String,
    pub installed_at: DateTime<Utc>,
    pub update_constraint: Option<String>,
}
```

Record written after a successful install / update.

**Fields**

- `schema_version`: DTO schema version for CLI/UI JSON compatibility.
- `plugin_key`: Provenance-qualified logical plugin identity (no version, no hash).
- `provenance`: Host-evaluated provenance at install time (re-checked at discovery).
- `coordinate`: Fully qualified package coordinate when version is known.
- `version`: Resolved or candidate package version string.
- `registry_url`: Base URL of the package registry (for example crates.io or npm).
- `artifact_url`: Resolved download URL used for this install.
- `target`: Host target triple used to select release artifacts.
- `archive_sha256`: Lowercase hex SHA-256 of the downloadable archive bytes (install-time).
- `manifest_sha256`: SHA-256 of the installed `plugin.toml` bytes.
- `payload_root_sha256`: Deterministic Merkle-style root over immutable packaged files.
- `executable_sha256`: Optional SHA-256 of the extracted executable bytes.
- `protocol`: Host↔guest wire protocol id (for example `workers-rpc`).
- `api_version`: Plugin ABI version negotiated with the host.
- `runtime`: Bookclerk runtime identity (kind + plugin id) when known.
- `requested_sandbox`: Publisher-requested sandbox snapshot recorded at install time.
- `approved_network`: Host-approved network mode (may be stricter than requested).
- `installed_at`: RFC 3339 time when this install was activated.
- `update_constraint`: Optional update constraint (for example pinned version range).

#### InstallReceipt::SCHEMA_VERSION

```rust
pub const SCHEMA_VERSION: u32 = 2
```

Receipt schema version for forward-compatible reads.

#### InstallReceipt::path_in

```rust
pub fn path_in(plugin_root: &Path) -> PathBuf
```

Path to receipt inside an installed plugin directory.

#### InstallReceipt::plugin_key

```rust
pub fn plugin_key(&self) -> Result<PluginKey>
```

Parsed [`PluginKey`](#pluginkey) from [`Self::plugin_key`](#installreceiptplugin_key).

##### Errors

Returns an error when the stored key is not canonical.

#### InstallReceipt::platform_bundled

```rust
pub fn platform_bundled(
    plugin_key: PluginKey,
    version: &str,
    manifest: &PluginManifest,
    manifest_sha256: String,
    payload_root_sha256: String,
    product: &str,
    package_name: &str,
) -> Self
```

Host-stamped receipt for a Bookclerk platform artifact.

#### InstallReceipt::load

```rust
pub fn load(plugin_root: &Path) -> Result<Self>
```

Load receipt from a plugin install directory.

Operator-selected `plugin_root` may contain `..` spelling (resolved via
canonicalize). Escaping or dangling leaf symlinks at `receipt.json` are
hard errors; a true missing file is [`CatalogError::ReceiptNotFound`](#catalogerror).

##### Errors

Returns [`CatalogError::ReceiptNotFound`](#catalogerror) when `receipt.json` is absent.
Any other I/O or JSON failure is returned as an error (fail closed).

#### InstallReceipt::store

```rust
pub fn store(&self, plugin_root: &Path) -> Result<()>
```

Atomically write receipt (unique same-dir staging + replace).

Does not follow final/backup/temp leaf symlinks: staging uses
`create_new`, and replace uses Unix `rename` / Windows `MoveFileExW` so
only the directory entry is replaced. Cleanup removes only the staging
file this call created.

##### Errors

Returns an error when the operation fails.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TrustPolicy

```rust
pub struct TrustPolicy {
    pub allow_unverified_publisher: bool,
    pub refuse_yanked: bool,
    pub warn_on_unsigned_os: bool,
}
```

Trust policy for plugin installation (digest-required artifacts).

[`Self::allow_unverified_publisher`](#trustpolicyallow_unverified_publisher) means the operator accepted a package
that has no independent publisher authenticity proof. Archive SHA-256 is
still required. Bookclerk does **not** verify publisher signatures.

**Fields**

- `allow_unverified_publisher`: Allow community packages without independent publisher authenticity
  (content digests are still required).
- `refuse_yanked`: Refuse yanked versions (always true for unattended).
- `warn_on_unsigned_os`: When true, warn instead of refuse on missing OS code signature.

#### TrustPolicy::allow_unverified_publisher

```rust
pub fn allow_unverified_publisher() -> Self
```

Interactive default: allow community packages after an explicit flag.

#### TrustPolicy::check_unverified_publisher_allowed

```rust
pub fn check_unverified_publisher_allowed(&self) -> Result<()>
```

Returns `Ok(())` when community packages without publisher authenticity
are permitted by this policy.

##### Errors

Returns an error when the operator has not opted in.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Version

```rust
pub struct Version {
    pub major: u64,
    pub minor: u64,
    pub patch: u64,
    pub pre: Option<String>,
}
```

Parsed `major.minor.patch` with optional pre-release / build metadata.

Ordering follows SemVer 2.0 for the subset we care about: a release is
greater than any pre-release with the same major/minor/patch
(`1.0.0` > `1.0.0-beta`). Pre-release identifiers are compared
lexicographically after numeric segments where both sides are integers.

**Fields**

- `major`: Semver major component.
- `minor`: Semver minor component.
- `patch`: Semver patch component.
- `pre`: `None` means a release (no `-pre` suffix). Empty vs missing is not used.

#### Version::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parse a version string (`1.2.3`, `1.2.3-beta`, `v1.2.3`).

#### Version::is_prerelease

```rust
pub fn is_prerelease(&self) -> bool
```

Returns true when this version has a prerelease suffix.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

## Enums

### BundledSyncAction

```rust
pub enum BundledSyncAction {
    Installed,
    Updated,
    Current,
    AliasOccupied { path: PathBuf },
    Failed { reason: String },
}
```

What [`sync_bundled_platform`](#sync_bundled_platform) did for one platform artifact.

**Variants**

- `Installed`: The files directory had no tree for this PluginKey; the bundled copy was installed.
- `Updated`: An untrusted or different tree was replaced by the bundled copy.
- `Current`: The installed tree is trusted and byte-identical to the bundled copy.
- `AliasOccupied`: Another directory under `$FILES_DIR/plugins` claims this alias; nothing was touched.
  - `path`: The occupying directory.
- `Failed`: This artifact could not be checked or installed; the others were still synced.
  - `reason`: Operator-facing reason.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### RegistrySource

```rust
pub enum RegistrySource {
    Cargo { registry_url: String },
    Npm { registry_url: String },
    Pypi { simple_url: String },
    Static { index_url: String },
    LocalArchive,
}
```

Where a package was discovered / will be fetched from.

**Variants**

- `Cargo`: crates.io (or alternate Cargo registry).
  - `registry_url`: Base URL of the package registry.
- `Npm`: npm registry.
  - `registry_url`: Base URL of the package registry.
- `Pypi`: PyPI (exact lookup only for search-less discovery).
  - `simple_url`: PyPI simple API base URL used for exact package lookup.
- `Static`: Static HTTPS JSON/TOML index.
  - `index_url`: Absolute URL of the static registry index document.
- `LocalArchive`: Local archive path (offline / fixture install).

#### RegistrySource::kind_name

```rust
pub fn kind_name(&self) -> &'static str
```

Returns the short source kind name (`cargo`, `npm`, `pypi`, `registry`, `local`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogError

```rust
pub enum CatalogError {
    Message(String),
    ReceiptNotFound,
    Io(Error),
    Json(Error),
    Toml(Error),
    Url(ParseError),
}
```

Failures from discovery, manifest validation, download, or installation.

**Variants**

- `Message`: Operator-facing error text with no structured code.
- `ReceiptNotFound`: `receipt.json` is genuinely absent from the install directory.
- `Io`: Filesystem I/O failure during download, extract, or activate.
- `Json`: JSON encode/decode failure (static index, receipt, …).
- `Toml`: TOML decode failure for package manifests.
- `Url`: Invalid URL in a coordinate, artifact, or index.

#### CatalogError::message

```rust
pub fn message(msg: impl Into<String>) -> Self
```

Builds a [`CatalogError::Message`](#catalogerror) from any displayable string.

##### Arguments

* `msg` - Operator-facing explanation; must not embed secrets.

##### Returns

A message-only [`CatalogError`](#catalogerror).

#### CatalogError::is_receipt_not_found

```rust
pub fn is_receipt_not_found(&self) -> bool
```

True when this error is a genuinely missing install receipt.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<ParseError>`

### PluginProvenance

```rust
pub enum PluginProvenance {
    PlatformBundled,
    VerifiedStaged,
    LocalDevelopment,
    Modified,
    Installed,
}
```

How Bookclerk established (or lost) trust in an installed tree.

**Variants**

- `PlatformBundled`: Host-staged Bookclerk platform artifact; content hashes match the receipt.
- `VerifiedStaged`: Installed from a registry/archive; receipt hashes still match.
- `LocalDevelopment`: Local/dev tree with no verified receipt (or path-only identity).
- `Modified`: Receipt exists but the immutable payload no longer matches.
- `Installed`: Materialized from a cluster install: the payload-root digest of the
  cached tree matched the `plugin_versions` row the node trusts.

#### PluginProvenance::grants_platform_defaults

```rust
pub fn grants_platform_defaults(self) -> bool
```

True when this provenance may receive installer platform default grants.

#### PluginProvenance::is_verified_artifact

```rust
pub fn is_verified_artifact(self) -> bool
```

True when discovery verified the installed payload against the host ledger.

[`Self::PlatformBundled`](#pluginprovenance), [`Self::VerifiedStaged`](#pluginprovenance), and
[`Self::Installed`](#pluginprovenance) qualify. [`Self::LocalDevelopment`](#pluginprovenance) (no receipt)
and [`Self::Modified`](#pluginprovenance) never do.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### ProvenanceScheme

```rust
pub enum ProvenanceScheme {
    Cargo,
    Npm,
    Pypi,
    Registry,
    Platform,
    Path,
}
```

Provenance scheme in a canonical [`PluginKey`](#pluginkey).

**Variants**

- `Cargo`: crates.io / Cargo registry.
- `Npm`: npm registry.
- `Pypi`: PyPI.
- `Registry`: Static HTTPS plugin index.
- `Platform`: Bookclerk-shipped installer artifact.
- `Path`: Local filesystem install / development tree.

#### ProvenanceScheme::as_str

```rust
pub fn as_str(self) -> &'static str
```

Canonical scheme token used in [`PluginKey`](#pluginkey) text.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### PluginKind

```rust
pub enum PluginKind {
    Source,
    Integration,
    Output,
    Database,
}
```

Which Bookclerk surface a plugin implements.

**Variants**

- `Source`: Content-source storefront guest (Audible, Libro.fm, …).
- `Integration`: Portal / library integration guest (not a content storefront).
- `Output`: Destination / output guest (local disk, S3, …).
- `Database`: Library database backend guest (`sqlite`, `d1`, `postgres`).

#### PluginKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Returns the canonical lowercase kind string for manifests and crate names.

#### PluginKind::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parse a kind segment from a coordinate or crate name.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### ArchiveFormat

```rust
pub enum ArchiveFormat {
    TarGz,
    Zip,
}
```

Archive format for a target.

**Variants**

- `TarGz`: Gzip-compressed tar archive (default for non-Windows targets).
- `Zip`: Zip archive (default for Windows targets).

#### ArchiveFormat::for_target

```rust
pub fn for_target(bookclerk_target: &str) -> Self
```

Returns the conventional archive format for a Bookclerk target id.

#### ArchiveFormat::extension

```rust
pub fn extension(self) -> &'static str
```

File extension without dot (for example `m4b` or `mp3`).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### load_static_index

```rust
pub fn load_static_index(path_or_url: &str) -> Result<StaticIndex>
```

Load a static index from a file path or `file://` URL.

#### Errors

Returns an error when the operation fails.

### bundled_plugins_dir_beside

```rust
pub fn bundled_plugins_dir_beside(exe: &Path) -> Option<PathBuf>
```

Bundled plugin directory for the host executable at `exe`, when it exists.

#### Arguments

* `exe` - Path of the running host binary (`std::env::current_exe()`).

#### Returns

`<dir of exe>/plugins` when that is a real directory (not a symlink);
`None` otherwise, which is the normal case for development builds.

### sync_bundled_platform

```rust
pub fn sync_bundled_platform(bundle: &Path, files_dir: &Path) -> Result<Vec<BundledSync>>
```

Installs bundled platform plugins from `bundle` into `files_dir`.

Artifacts the bundle does not carry get no row. When `bundle` is
`files_dir/plugins` itself (a release bundle used directly as the files
directory) there is nothing to copy and the result is empty. A failure for
one artifact is reported in its row and does not stop the others.

When an install is needed this blocks on the files directory's
[`PluginMutationLock`](#pluginmutationlock), so it must not be called while this process holds
that lock.

#### Arguments

* `bundle` - Directory holding `<PluginKey fs-id>/plugin.toml` trees.
* `files_dir` - Host `$BOOKCLERK_FILES_DIR` that owns `install-ledger.json`.

#### Returns

One [`BundledSync`](#bundledsync) per platform artifact found in `bundle`.

#### Errors

Returns an error only when the bundle or `files_dir/plugins` cannot be
resolved, or the mutation lock cannot be taken.

### evaluate_install

```rust
pub fn evaluate_install(
    root: &Path,
    manifest: &PluginManifest,
) -> Result<PluginInstallIdentity>
```

Evaluate identity + provenance for an install directory.

Equivalent to [`evaluate_install_in`](#evaluate_install_in) with no host files directory. Without
a files-dir ledger, platform trust cannot be established.

#### Errors

Returns an error when the payload cannot be hashed, `plugin.toml` is
missing, a receipt is malformed, or a present receipt is not recorded in
the host ledger.

### evaluate_install_in

```rust
pub fn evaluate_install_in(
    root: &Path,
    manifest: &PluginManifest,
    files_dir: Option<&Path>,
) -> Result<PluginInstallIdentity>
```

Evaluate identity + provenance, consulting the host install ledger under
`files_dir` when provided.

Receipt fields are **not** a trust anchor. [`PluginProvenance::PlatformBundled`](#pluginprovenance)
requires a matching [`InstallLedger`](#installledger) row written by host tooling, a known
platform [`PluginKey`](#pluginkey), and exact current payload digests.

A genuinely absent receipt is [`PluginProvenance::LocalDevelopment`](#pluginprovenance). Any
other receipt/hash failure fails closed.

#### Errors

Returns an error when hashing fails, a receipt is unreadable or malformed,
the stored PluginKey is invalid, or a present receipt has no matching
host ledger entry.

### stamp_platform_receipt

```rust
pub fn stamp_platform_receipt(
    root: &Path,
    files_dir: &Path,
    package_name: &str,
    manifest: &PluginManifest,
    version: &str,
) -> Result<InstallReceipt>
```

Build a host-owned platform-bundled receipt **and** ledger row.

Only [`crate::identity::PLATFORM_ARTIFACTS`](#platform_artifacts) may be stamped this way.
`files_dir` is the Bookclerk files directory that owns `install-ledger.json`.

#### Errors

Returns an error when `package_name`/`manifest.id` are not a known
platform pair, hashing fails, or the ledger cannot be written.

### extract_archive

```rust
pub fn extract_archive(archive: &Path, format: ArchiveFormat, dest: &Path) -> Result<()>
```

Extract an archive into `dest`, refusing path traversal and oversized output.

#### Errors

Returns an error when the operation fails.

### require_mutable_child

```rust
pub fn require_mutable_child(root: &Path, path: &Path) -> Result<PathBuf>
```

Lexical child path under `root` safe for `remove_dir_all` / `rename` / create.

Unlike [`require_under`](#require_under), this returns the intended directory entry (not a
symlink target) and refuses when the leaf exists as a symlink or when a
resolved existing path equals the trusted root (so cleanup cannot wipe the
root via an in-root alias). Missing leaves are validated via [`require_under`](#require_under).

#### Errors

Returns when the path escapes `root`, is a symlink leaf, or would mutate the root.

### require_under

```rust
pub fn require_under(root: &Path, path: &Path) -> Result<PathBuf>
```

Returns `path` when it stays under `root` (canonical containment).

Use before filesystem sinks that already hold an absolute path built from a
trusted root (install dest, staging, receipt). Prefer [`safe_join`](#safe_join) when the
relative segment is still separate.

When `path` does not exist yet, canonicalizes the nearest existing ancestor
and rejoins the missing suffix (same approach as bookclerk-sandbox
`require_under_root`) so a symlinked parent such as
`root/.staging -> /outside` cannot pass a lexical `starts_with` check.

#### Errors

Returns when `path` contains a `ParentDir` component, or escapes `root`
lexically or after canonicalize.

### safe_join

```rust
pub fn safe_join(dest: &Path, rel: &Path) -> Result<PathBuf>
```

Join `dest` / `rel` while rejecting absolute paths and `..` escapes.

#### Errors

Returns an error when the operation fails.

### sha256_bytes

```rust
pub fn sha256_bytes(data: &[u8]) -> String
```

SHA-256 hex digest of bytes.

### sha256_file

```rust
pub fn sha256_file(path: &Path) -> Result<String>
```

SHA-256 hex digest of a file.

Callers must pass a path already contained under a trusted root (for
example via [`safe_join`](#safe_join) / [`require_under`](#require_under)). Trusted roots may retain
intentional `..` spelling until the caller canonicalizes them; this hasher
does not re-litigate path spelling.

#### Errors

Returns an error when the file cannot be read.

### is_first_party_database_adapter

```rust
pub fn is_first_party_database_adapter(key: &PluginKey) -> bool
```

True when `key` names a host-controlled first-party database adapter.

Matches exact package name on `platform:` or the canonical crates.io
`cargo:` index. Callers that inject host-private state must also require
verified provenance and the expected alias on the discovered plugin.

### is_first_party_destination

```rust
pub fn is_first_party_destination(key: &PluginKey) -> bool
```

True when `key` names a host-controlled first-party destination (`local` / `s3`).

### is_first_party_integration

```rust
pub fn is_first_party_integration(key: &PluginKey) -> bool
```

True when `key` names a host-controlled first-party integration.

Matches exact package name on `platform:` or the canonical crates.io
`cargo:` index. Callers that grant host-implied network access must also
require verified provenance on the discovered plugin.

### is_platform_plugin_key

```rust
pub fn is_platform_plugin_key(key: &PluginKey) -> bool
```

True when `key` is a known Bookclerk platform [`PluginKey`](#pluginkey).

### platform_artifact

```rust
pub fn platform_artifact(
    package_name: &str,
    manifest_id: &str,
) -> Option<&'static PlatformArtifact>
```

Looks up a known platform artifact by package name + manifest id.

### fetch_artifact

```rust
pub fn fetch_artifact(url: &str, dest: &Path, offline: bool) -> Result<()>
```

Fetches an artifact URL (`file://` or local path, or HTTPS/localhost
HTTP bounded by [`MAX_DOWNLOAD_BYTES`](#max_download_bytes) and [`DOWNLOAD_TIMEOUT`](#download_timeout)) into
`dest`, refusing oversized bodies.

#### Errors

Returns an error when the URL is unsupported, the host is offline for a
remote URL, or the transfer fails or exceeds the cap.

### record_install

```rust
pub fn record_install(
    files_dir: &Path,
    plugin_key: &PluginKey,
    package_name: &str,
    manifest_id: &str,
    manifest_sha256: &str,
    payload_root_sha256: &str,
    provenance: PluginProvenance,
) -> Result<()>
```

Records (or updates) a host-verified install in the files-dir ledger.

#### Errors

Returns when the ledger cannot be loaded or stored, or `plugin_key` is empty.

### restore_ledger_entry

```rust
pub fn restore_ledger_entry(
    files_dir: &Path,
    plugin_key: &PluginKey,
    previous: Option<InstallLedgerEntry>,
) -> Result<()>
```

Restores `previous` for `plugin_key`, or deletes the row when `previous` is `None`.

#### Errors

Returns when the ledger cannot be loaded or stored.

### normalize_protocol

```rust
pub fn normalize_protocol(protocol: Option<&str>) -> Result<String>
```

Normalize optional `protocol` for Workers RPC packages.

Accepts absent / empty / `workers-rpc` only.

#### Errors

Returns an error when the operation fails.

### parse_sha256_hex

```rust
pub fn parse_sha256_hex(s: &str) -> Result<[u8; 32]>
```

Decode hex SHA-256 into 32 bytes.

#### Errors

Returns an error when the operation fails.

### validate_sha256_hex

```rust
pub fn validate_sha256_hex(s: &str) -> Result<()>
```

Validate lowercase hex SHA-256 (64 chars).

#### Errors

Returns an error when the operation fails.

### acquire_plugins_lock

```rust
pub fn acquire_plugins_lock(plugins_root: &Path) -> Result<Option<PluginMutationLock>>
```

Acquires the host mutation lock for `plugins_root`'s parent files dir.

#### Errors

Returns when the lock cannot be taken. `None` when `plugins_root` has no
parent (no host files dir).

### check_owner_artifact_url

```rust
pub fn check_owner_artifact_url(url: &str) -> Result<Url>
```

Checks a URL an owner cluster asks this host to fetch an archive from.

This is the textual half of the policy; the fetch enforces the address
half again on what each name resolves to (`PublicOnlyResolver`).

#### Arguments

* `url` - The owner's artifact URL.

#### Returns

The parsed URL.

#### Errors

Returns an error when the URL does not parse, is not `https:`, carries
credentials, has no host, or names `localhost` (or a `.localhost`
name, or another name that always resolves locally) or an IP literal
that is not a public Internet address.

### fetch_owner_artifact

```rust
pub fn fetch_owner_artifact(url: &str, dest: &Path) -> Result<()>
```

Fetches an owner-chosen `https:` archive into `dest`, bounded by
[`MAX_DOWNLOAD_BYTES`](#max_download_bytes) and [`DOWNLOAD_TIMEOUT`](#download_timeout) per request. Redirects
are followed (at most `MAX_REDIRECTS`) only when each hop passes
[`check_owner_artifact_url`](#check_owner_artifact_url), and every hop connects only to addresses
that pass `owner_address_allowed` (`PublicOnlyResolver`); no proxy
is used. The caller checks the archive digest.

#### Arguments

* `url` - The owner's artifact URL.
* `dest` - Where the body is written.

#### Errors

Returns an error when a URL or a redirect hop is refused, a host
resolves to a refused address, there are too many redirects, or the
transfer fails or exceeds the cap.

### manifest_sha256

```rust
pub fn manifest_sha256(plugin_root: &Path) -> Result<String>
```

SHA-256 of `plugin.toml` at `plugin_root`.

#### Errors

Returns an error when the file cannot be read.

### payload_root_sha256

```rust
pub fn payload_root_sha256(plugin_root: &Path) -> Result<String>
```

Deterministic payload root: sorted relative path + type + content hash.

Encoding (UTF-8, `\n`-terminated records, paths use `/`):

- file: `f\t{rel}\t{sha256}`
- directory: `d\t{rel}` (children listed separately)

Symlinks and path traversal (`..`) are rejected. The root directory itself
is not listed; an empty tree hashes the empty byte string.

#### Errors

Returns an error on I/O failure, traversal, or a symlink in the payload.

### host_bookclerk_target

```rust
pub fn host_bookclerk_target() -> &'static str
```

Host Bookclerk target for the machine compiling this code.

### normalize_target

```rust
pub fn normalize_target(triple_or_target: &str) -> Option<&'static str>
```

Map a rustc triple (or Bookclerk target) to a Bookclerk target name.

### rust_triple

```rust
pub fn rust_triple(bookclerk_target: &str) -> Option<&'static str>
```

Rustc triple corresponding to a Bookclerk target (for packaging).

### select_target

```rust
pub fn select_target<'a, T>(
    artifacts: &'a [T],
    target_of: impl Fn(&T) -> &str,
    prefer: Option<&str>,
) -> Result<&'a T>
```

Select an artifact entry matching the host (or override) target.

#### Errors

Returns an error when the operation fails.

### max_version

```rust
pub fn max_version<'a>(versions: impl IntoIterator<Item = &'a str>) -> Option<&'a str>
```

Prefer the greatest release version; if only pre-releases exist, the
greatest pre-release. Falls back to lexical max when nothing parses.

### newest_newer_than

```rust
pub fn newest_newer_than<'a>(
    current: &str,
    versions: impl IntoIterator<Item = &'a str>,
) -> Option<&'a str>
```

Newest version strictly greater than `current`, if any.

Prefer a newer release over a newer pre-release when both qualify.

### federated_search

```rust
pub fn federated_search(
    adapters: &[&dyn RegistryAdapter],
    query: &SearchQuery,
) -> Result<Vec<CatalogHit>>
```

Federated search across configured adapters (static first, then cargo/npm).

#### Arguments

* `adapters` - Registry adapters to query in order.
* `query` - Free-text query and hit limit.

#### Returns

Combined catalog hits truncated to `query.limit` (clamped 1..=100).

#### Errors

Returns [`CatalogError`](#catalogerror) when every adapter fails and no hits were collected.

### fetch_manifest_for_coordinate

```rust
pub fn fetch_manifest_for_coordinate(
    coord: &PackageCoordinate,
    static_indexes: &[(String, StaticIndex)],
) -> Result<BookclerkPackageManifest>
```

Resolve a coordinate to a manifest using the matching adapter.

#### Errors

Returns an error when the operation fails.

### list_versions_for_coordinate

```rust
pub fn list_versions_for_coordinate(
    coord: &PackageCoordinate,
    static_indexes: &[(String, StaticIndex)],
) -> Result<Vec<String>>
```

List published versions for a coordinate's package name.

#### Errors

Returns an error when the operation fails.

### resolve_newer_version

```rust
pub fn resolve_newer_version(
    coord: &PackageCoordinate,
    static_indexes: &[(String, StaticIndex)],
) -> Result<Option<String>>
```

Resolve the newest version newer than `coord.version`, if any.

#### Errors

Returns an error when the operation fails.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, CatalogError>;
```

Result alias for [`CatalogError`](#catalogerror).

## Constants

### BUNDLED_PLUGINS_DIR

```rust
pub const BUNDLED_PLUGINS_DIR: &str = "plugins";
```

Directory beside the host executable that holds bundled platform plugins.

### CATALOG_DTO_SCHEMA_VERSION

```rust
pub const CATALOG_DTO_SCHEMA_VERSION: u32 = 1;
```

Schema version for CLI JSON catalog output.

### MAX_ARCHIVE_BYTES

```rust
pub const MAX_ARCHIVE_BYTES: u64 = _;
```

Hard cap on archive file size before extraction.

### MAX_EXTRACTED_BYTES

```rust
pub const MAX_EXTRACTED_BYTES: u64 = _;
```

Hard cap on total extracted archive bytes.

### CRATES_IO_INDEX

```rust
pub const CRATES_IO_INDEX: &str = "https://crates.io";
```

Default Cargo registry origin used in canonical `cargo:` PluginKeys.

### FIRST_PARTY_DATABASE_ADAPTERS

```rust
pub const FIRST_PARTY_DATABASE_ADAPTERS: &[PlatformArtifact] = _;
```

Host-controlled first-party database adapter identities.

Secret injection, library-path grants, and host-private connect params
require this exact package + manifest id **and** a ledger-backed
provenance. A third-party tree that merely reuses the `sqlite` /
`postgres` / `d1` alias never matches.

### FIRST_PARTY_DESTINATIONS

```rust
pub const FIRST_PARTY_DESTINATIONS: &[PlatformArtifact] = _;
```

Host-controlled first-party destination identities (`local`, `s3`).

Typed `[output.local]` / `[output.s3]` secrets and extra jail grants require
this exact package on `platform:` or the canonical crates.io `cargo:` index
**and** a verified artifact. A third-party tree that merely reuses the
`local` / `s3` alias never matches. The live manifest alias is **not**
consulted here.

### PLATFORM_ARTIFACTS

```rust
pub const PLATFORM_ARTIFACTS: &[PlatformArtifact] = _;
```

Known installer-shipped platform packages. Host-controlled; never read from
plugin-authored manifest fields other than matching `id` after provenance
is already established.

### PLATFORM_PRODUCT

```rust
pub const PLATFORM_PRODUCT: &str = "bookclerk";
```

Product name recorded on Bookclerk-shipped platform artifacts.

### PLUGIN_KEY_FS_ID_HEX_CHARS

```rust
pub const PLUGIN_KEY_FS_ID_HEX_CHARS: usize = 32;
```

Hex characters of SHA-256 used in [`PluginKey::fs_id`](#pluginkeyfs_id) (128 bits).

### DOWNLOAD_TIMEOUT

```rust
pub const DOWNLOAD_TIMEOUT: Duration = _;
```

Download / install limits.

### MAX_DOWNLOAD_BYTES

```rust
pub const MAX_DOWNLOAD_BYTES: u64 = _;
```

Hard cap on downloaded artifact size in bytes.

### PLUGIN_HOLD_DIR

```rust
pub const PLUGIN_HOLD_DIR: &str = ".plugin-hold";
```

Host-owned directory under `$FILES_DIR` for held plugin-state during remove.

Not inside a plugin-controlled install tree. Used so `--purge-state` can
rename `plugin-state/<fs-id>` aside and restore it on rollback.

### INSTALL_LEDGER_FILE

```rust
pub const INSTALL_LEDGER_FILE: &str = "install-ledger.json";
```

Filename under `$BOOKCLERK_FILES_DIR` for the host install ledger.

### INSTALL_LEDGER_SCHEMA_VERSION

```rust
pub const INSTALL_LEDGER_SCHEMA_VERSION: u32 = 1;
```

Schema version for [`InstallLedger`](#installledger).

### MANIFEST_SCHEMA_VERSION

```rust
pub const MANIFEST_SCHEMA_VERSION: u32 = 1;
```

Current package manifest schema.

### PROTOCOL_WORKERS_RPC

```rust
pub const PROTOCOL_WORKERS_RPC: &str = "workers-rpc";
```

Product wire protocol for Workers RPC (`api_version = 3`).

### PLUGIN_MUTATION_LOCK_FILE

```rust
pub const PLUGIN_MUTATION_LOCK_FILE: &str = ".plugin-mutation.lock";
```

Filename under `$BOOKCLERK_FILES_DIR` for the host plugin mutation lock.

### PAYLOAD_SKIP_NAMES

```rust
pub const PAYLOAD_SKIP_NAMES: &[&str] = _;
```

Directory / file names excluded from the immutable payload (host-owned
receipts only). Packaged `data/` and `tmp/` are hashed: mutable plugin
state lives under `$FILES_DIR/plugin-state/<PluginKey fs-id>/`, not inside
the install tree.

### RECEIPT_FILE

```rust
pub const RECEIPT_FILE: &str = "receipt.json";
```

Filename for the receipt beside `plugin.toml`.

### TARGETS

```rust
pub const TARGETS: &[&str] = _;
```

Canonical Bookclerk install targets (not raw rustc triples).
