# bookclerk_source

Host-side content-source traits plus the storefront types guests share.

## Audience

Host registration code and the plugin host adapter implementing
[`ContentSource`](#contentsource). The types a storefront guest also needs (DTOs, ABI
conversions, listing / record helpers) live in the slim
[`bookclerk_storefront`](../bookclerk_storefront/README.md) crate and are re-exported here unchanged, so host
code keeps importing them from `bookclerk_source`. Guests depend on
`bookclerk-storefront` directly and never link this crate, which pulls in
the library store and host configuration.

Product narrative: `docs/plugins/README.md`. Style: `docs/contributing/code-documentation.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ContentSource`](#contentsource) | trait | Pluggable audiobook storefront (auth, scan, fetch, catalog). |
| [`FetchOptions`](#fetchoptions) | struct | Options passed to `crate::ContentSource::fetch_title`. |
| [`DownloadOptions`](#downloadoptions) | struct | Options for acquire / fetch packaging and naming. |
| [`SourceRegistry`](#sourceregistry) | struct | Maps a registry address to an installed `ContentSource`. |
| [`PortalAuthMode`](#portalauthmode) | enum | How the connect portal authenticates this source. |
| [`revoke_credentials_default`](#revoke_credentials_default) | function | Shared revoke path for plugins that seal as `.plugin.auth` (and legacy names). |
| [`source_scope`](#source_scope) | function | Opens the `SourceScope` for `source`: rows are isolated by its `ContentSource::plugin_key` and labelled with its `ContentSource::id`. |

## Re-exports

- [`bookclerk_storefront::audio_extension`](../bookclerk_storefront/README.md#audio_extension)
- [`bookclerk_storefront::default_preferred_language`](../bookclerk_storefront/README.md#default_preferred_language)
- [`bookclerk_storefront::extension_from_bytes`](../bookclerk_storefront/README.md#extension_from_bytes)
- [`bookclerk_storefront::extension_from_content_type`](../bookclerk_storefront/README.md#extension_from_content_type)
- [`bookclerk_storefront::extension_from_url`](../bookclerk_storefront/README.md#extension_from_url)
- [`bookclerk_storefront::language_rank`](../bookclerk_storefront/README.md#language_rank)
- [`bookclerk_storefront::listing`](../bookclerk_storefront/listing.md)
- [`bookclerk_storefront::normalize_language`](../bookclerk_storefront/README.md#normalize_language)
- [`bookclerk_storefront::records`](../bookclerk_storefront/records.md)
- [`bookclerk_storefront::CatalogHit`](../bookclerk_storefront/README.md#cataloghit)
- [`bookclerk_storefront::CatalogSearchField`](../bookclerk_storefront/README.md#catalogsearchfield)
- [`bookclerk_storefront::CatalogSearchOpts`](../bookclerk_storefront/README.md#catalogsearchopts)
- [`bookclerk_storefront::CatalogSearchSort`](../bookclerk_storefront/README.md#catalogsearchsort)
- [`bookclerk_storefront::CatalogSortDir`](../bookclerk_storefront/README.md#catalogsortdir)
- [`bookclerk_storefront::ConfigOptionValue`](../bookclerk_storefront/README.md#configoptionvalue)
- [`bookclerk_storefront::ExpandSeed`](../bookclerk_storefront/README.md#expandseed)
- [`bookclerk_storefront::ImportCredentialsOptions`](../bookclerk_storefront/README.md#importcredentialsoptions)
- [`bookclerk_storefront::LoginOptions`](../bookclerk_storefront/README.md#loginoptions)
- [`bookclerk_storefront::OAuthProgress`](../bookclerk_storefront/README.md#oauthprogress)
- [`bookclerk_storefront::PlainAudioPart`](../bookclerk_storefront/README.md#plainaudiopart)
- [`bookclerk_storefront::PlainFetch`](../bookclerk_storefront/README.md#plainfetch)
- [`bookclerk_storefront::PurchaseHintOpts`](../bookclerk_storefront/README.md#purchasehintopts)
- [`bookclerk_storefront::Result`](../bookclerk_storefront/README.md#result)
- [`bookclerk_storefront::ScanOptions`](../bookclerk_storefront/README.md#scanoptions)
- [`bookclerk_storefront::ScanSummary`](../bookclerk_storefront/README.md#scansummary)
- [`bookclerk_storefront::SourceAccount`](../bookclerk_storefront/README.md#sourceaccount)
- [`bookclerk_storefront::SourceBrand`](../bookclerk_storefront/README.md#sourcebrand)
- [`bookclerk_storefront::SourceConfigOption`](../bookclerk_storefront/README.md#sourceconfigoption)
- [`bookclerk_storefront::SourceError`](../bookclerk_storefront/README.md#sourceerror)
- [`bookclerk_storefront::SourceFetch`](../bookclerk_storefront/README.md#sourcefetch)
- [`bookclerk_storefront::SourcePurchaseHint`](../bookclerk_storefront/README.md#sourcepurchasehint)

## Modules

- [`abi`](abi.md): Host-side conversions between storefront types and the typed plugin ABI.

## Traits

### ContentSource

```rust
pub trait ContentSource: Send + Sync {
    fn id(&self) -> &str;
    fn plugin_key(&self) -> &str;
    fn plugin_instance_id(&self) -> Option<&str>;
    fn guest_pid(&self) -> Option<u32>;
    fn display_name(&self) -> &str;
    fn aliases(&self) -> &'static [&'static str];
    fn portal_auth_mode(&self) -> PortalAuthMode;
    fn portal_brand(&self) -> SourceBrand;
    fn sort_key(&self) -> u32;
    fn login<'life0, 'life1, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
        opts: LoginOptions,
    ) -> Pin<Box<dyn Future<Output = Result<SourceAccount>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn login_with_oauth_progress<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
        opts: LoginOptions,
        on_progress: &'life2 dyn Fn(OAuthProgress) + Send + Sync,
    ) -> Pin<Box<dyn Future<Output = Result<SourceAccount>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn import_credentials<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
        path: &'life2 Path,
        opts: ImportCredentialsOptions,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<SourceAccount>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn revoke_credentials<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
        account_id: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn list_accounts<'life0, 'life1, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<SourceAccount>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn scan<'life0, 'life1, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
        opts: ScanOptions,
    ) -> Pin<Box<dyn Future<Output = Result<ScanSummary>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn fetch_title<'life0, 'life1, 'life2, 'life3, 'life4, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
        account_id: &'life2 str,
        title_id: &'life3 str,
        opts: &'life4 FetchOptions,
    ) -> Pin<Box<dyn Future<Output = Result<SourceFetch>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,
        'life4: 'async_trait,;
    fn config_options(&self) -> &'static [SourceConfigOption];
    fn inspect_title<'life0, 'life1, 'life2, 'life3, 'life4, 'async_trait>(
        &'life0 self,
        scope: &'life1 SourceScope,
        account_id: &'life2 str,
        title_id: &'life3 str,
        opts: &'life4 FetchOptions,
    ) -> Pin<Box<dyn Future<Output = Result<Value>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,
        'life3: 'async_trait,
        'life4: 'async_trait,;
    fn search_catalog<'life0, 'life1, 'async_trait>(
        &'life0 self,
        opts: &'life1 CatalogSearchOpts,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn catalog_detail<'life0, 'life1, 'async_trait>(
        &'life0 self,
        product_id: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<CatalogHit>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn expand_candidates<'life0, 'life1, 'async_trait>(
        &'life0 self,
        seed: &'life1 ExpandSeed,
        limit: usize,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn purchase_hint<'life0, 'life1, 'async_trait>(
        &'life0 self,
        opts: &'life1 PurchaseHintOpts,
    ) -> Pin<Box<dyn Future<Output = Result<Option<SourcePurchaseHint>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list_deals<'life0, 'async_trait>(
        &'life0 self,
        limit: usize,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + Send + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Pluggable audiobook storefront (auth, scan, fetch, catalog).

Plugins own their id, brand, auth mode, and config parsing. Hosts register
concrete crates at startup and talk only through this trait.

All credential and library mutations go through [`SourceScope`](../bookclerk_library/scope.md#sourcescope), which
forces `source` / `provider` to this plugin's id. First-party in-repo
adapters and third-party JSON-RPC plugins share the same scope rules.

#### ContentSource::id

```rust
fn id(&self) -> &str
```

Display / CLI alias (`audible`, `libro`, …). Globally unique within one
installation; durable identity is [`Self::plugin_key`](#contentsourceplugin_key).

#### ContentSource::plugin_key

```rust
fn plugin_key(&self) -> &str
```

Provenance-qualified PluginKey when this adapter is an external guest.

Defaults to [`Self::id`](#contentsourceid) for in-crate test doubles.

#### ContentSource::plugin_instance_id

```rust
fn plugin_instance_id(&self) -> Option<&str>
```

Plugin instance that owns this deployed source.

`None` for transitional startup and in-crate adapters, which stay
addressable by [`Self::plugin_key`](#contentsourceplugin_key) and [`Self::id`](#contentsourceid). Two deployed
instances of one key each return their own id.

#### ContentSource::guest_pid

```rust
fn guest_pid(&self) -> Option<u32>
```

Native guest pid when this process spawned the source.

`None` for in-process adapters and sources that have not recorded a pid.

#### ContentSource::display_name

```rust
fn display_name(&self) -> &str
```

Human-facing store name for UI / logs.

#### ContentSource::aliases

```rust
fn aliases(&self) -> &'static [&'static str]
```

Alternate ids accepted by CLI / config (`libro.fm`, `ga`, …).

#### ContentSource::portal_auth_mode

```rust
fn portal_auth_mode(&self) -> PortalAuthMode
```

Connect-portal auth mode.

#### ContentSource::portal_brand

```rust
fn portal_brand(&self) -> SourceBrand
```

Portal button brand (colors + favicon).

#### ContentSource::sort_key

```rust
fn sort_key(&self) -> u32
```

Stable sort key for registry listing (lower first).

#### ContentSource::login

```rust
fn login<'life0, 'life1, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
    opts: LoginOptions,
) -> Pin<Box<dyn Future<Output = Result<SourceAccount>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Authenticate and persist credentials via [`SourceScope`](../bookclerk_library/scope.md#sourcescope).

#### ContentSource::login_with_oauth_progress

```rust
fn login_with_oauth_progress<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
    opts: LoginOptions,
    on_progress: &'life2 dyn Fn(OAuthProgress) + Send + Sync,
) -> Pin<Box<dyn Future<Output = Result<SourceAccount>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Interactive OAuth login with progress (URL / waiting).

Default: ignores progress and calls [`Self::login`](#contentsourcelogin). OAuth sources
(Audible, external guests with `login.start`) override this.

#### ContentSource::import_credentials

```rust
fn import_credentials<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
    path: &'life2 Path,
    opts: ImportCredentialsOptions,
) -> Pin<Box<dyn Future<Output = Result<Vec<SourceAccount>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Import credentials from a file (auth JSON, Libation export, …).

Default: unsupported. Audible implements auth-file / Libation / mkb79.

#### ContentSource::revoke_credentials

```rust
fn revoke_credentials<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
    account_id: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Delete stored credentials for `account_id` (books / account rows kept).

Default: removes `{account}.plugin.auth`, any other `source_auth` secrets
for the account, common legacy name patterns, and a Widevine `{id}.wvd`.

#### ContentSource::list_accounts

```rust
fn list_accounts<'life0, 'life1, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
) -> Pin<Box<dyn Future<Output = Result<Vec<SourceAccount>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

List accounts for this plugin (scope filters by source id).

#### ContentSource::scan

```rust
fn scan<'life0, 'life1, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
    opts: ScanOptions,
) -> Pin<Box<dyn Future<Output = Result<ScanSummary>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Sync library rows using scoped credentials / upserts.

#### ContentSource::fetch_title

```rust
fn fetch_title<'life0, 'life1, 'life2, 'life3, 'life4, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
    account_id: &'life2 str,
    title_id: &'life3 str,
    opts: &'life4 FetchOptions,
) -> Pin<Box<dyn Future<Output = Result<SourceFetch>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
    'life4: 'async_trait,
```

Fetch everything needed to acquire one title (no storage writes).

`title_id` is the source-native product id (Audible ASIN or Libro ISBN).
Auth is loaded through `scope`; `opts.files_dir` carries the
`BOOKCLERK_FILES_DIR` path for CDM / Widevine resolution.

#### ContentSource::config_options

```rust
fn config_options(&self) -> &'static [SourceConfigOption]
```

Source-native `[sources.<id>]` knobs for config / UI discovery.

#### ContentSource::inspect_title

```rust
fn inspect_title<'life0, 'life1, 'life2, 'life3, 'life4, 'async_trait>(
    &'life0 self,
    scope: &'life1 SourceScope,
    account_id: &'life2 str,
    title_id: &'life3 str,
    opts: &'life4 FetchOptions,
) -> Pin<Box<dyn Future<Output = Result<Value>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
    'life3: 'async_trait,
    'life4: 'async_trait,
```

Optional diagnostic / license inspect for one title (opaque JSON).

Default: unsupported. Hosts must not special-case store APIs — sources
that expose license dumps (e.g. Audible) override this.

#### ContentSource::search_catalog

```rust
fn search_catalog<'life0, 'life1, 'async_trait>(
    &'life0 self,
    opts: &'life1 CatalogSearchOpts,
) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Public catalog typeahead / search (no account required).

Default: empty. Discover aggregates hits across registered sources.

#### ContentSource::catalog_detail

```rust
fn catalog_detail<'life0, 'life1, 'async_trait>(
    &'life0 self,
    product_id: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Option<CatalogHit>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Fetch rich bibliographic metadata for one catalog product (no account).

Default: none. Used when search/expand hits are lean (e.g. HTML scrape)
and a title detail dialog needs narrator, runtime, description, etc.

#### ContentSource::expand_candidates

```rust
fn expand_candidates<'life0, 'life1, 'async_trait>(
    &'life0 self,
    seed: &'life1 ExpandSeed,
    limit: usize,
) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Expand related / series / author candidates from a taste seed.

Default: empty. `limit` caps returned hits (sources may also budget HTTP).

#### ContentSource::purchase_hint

```rust
fn purchase_hint<'life0, 'life1, 'async_trait>(
    &'life0 self,
    opts: &'life1 PurchaseHintOpts,
) -> Pin<Box<dyn Future<Output = Result<Option<SourcePurchaseHint>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Resolve a purchase / catalog URL (optionally with live price).

Default: none.

#### ContentSource::list_deals

```rust
fn list_deals<'life0, 'async_trait>(
    &'life0 self,
    limit: usize,
) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + Send + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Current deals / promos for discovery shelves.

Default: empty.

## Structs

### FetchOptions

```rust
pub struct FetchOptions {
    pub download: DownloadOptions,
    pub cache_dir: PathBuf,
    pub files_dir: PathBuf,
    pub max_cache_bytes: Option<u64>,
    pub cancel: Option<Arc<AtomicBool>>,
}
```

Options passed to [`crate::ContentSource::fetch_title`](#contentsourcefetch_title).

**Fields**

- `download`: Packaging / naming knobs for this acquire (from `[output]` + destination).
- `cache_dir`: Absolute directory for scratch downloads for this fetch.
- `files_dir`: Root files directory (`BOOKCLERK_FILES_DIR`). Used for CDM / Widevine path
  resolution by Audible. Non-auth operations only; auth is loaded from the
  [`bookclerk_library::LibraryStore`](../bookclerk_library/README.md#librarystore) passed directly to the trait method.
- `max_cache_bytes`: Optional on-disk budget for `cache_dir` (bytes). Sources should call
  [`Self::enforce_cache_budget`](#fetchoptionsenforce_cache_budget) between chunks.
- `cancel`: Cooperative cancel for mid-fetch quota or a lost job fence.

#### FetchOptions::enforce_cache_budget

```rust
pub fn enforce_cache_budget(&self) -> Result<()>
```

Fails when cancel is set or `cache_dir` exceeds [`Self::max_cache_bytes`](#fetchoptions).

##### Errors

Returns `SourceError::Other` when the fetch was cancelled or the
cache directory is over the remaining scratch quota.

Implements: `Clone`, `Debug`

### DownloadOptions

```rust
pub struct DownloadOptions {
    pub quality: AudioQuality,
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
    pub save_metadata_json: bool,
    pub cover_size: String,
    pub chapter_layout: String,
    pub overwrite_existing: bool,
    pub split_files_by_chapter: bool,
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
    pub replacement_characters: Vec<ReplacementRule>,
    pub path_limits: PathLimits,
    pub save_podcasts_to_parent_folder: bool,
}
```

Options for acquire / fetch packaging and naming.

Store-specific ingest knobs live on each [`crate::ContentSource`](#contentsource) instance
(parsed from `[sources.<id>]` at registration). `quality` remains here so
Audible’s download helpers can overlay the plugin bitrate for a single fetch.

**Fields**

- `quality`: Optional per-fetch audio quality overlay (set by plugins that need it).
- `format`: Post-download packaging format (M4B / MP3 / passthrough).
- `widevine`: Prefer Widevine/CENC download when the store offers it.
- `xhe_aac`: Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm`: Optional local Widevine `.wvd` path (absolute or under files dir).
- `widevine_cdm_provider`: Remote L3 CDM provider URL (`None` = classic Libation AudibleCdm; empty/`off` = disable).
- `naming_profile`: Path-template profile; per-field template overrides win when set.
- `folder_template`: Optional folder-path template override; `None` uses the naming profile.
- `file_template`: Optional file-stem template override; `None` uses the naming profile.
- `download_cover`: When true, download a cover JPEG alongside audio.
- `download_pdf`: When true, download a companion PDF when the store exposes one.
- `create_cue`: When true, write a `.cue` sidecar from API chapters.
- `fixup_metadata`: When true, embed tags, cover, and chapters after packaging.
- `chapter_json`: Which chapter JSON sidecars to write (`off` / `flat` / `tree` / `both`).
- `save_metadata_json`: When true, persist raw catalog API JSON as `metadata.json`.
- `cover_size`: Cover image size request (`500`, `1215`, or `native`).
- `chapter_layout`: Preferred Audible chapter API layout when fetching (`tree` or `flat`).
- `overwrite_existing`: When true, re-acquire even if media already exists at the destination.
- `split_files_by_chapter`: When true, also split packaged output into per-chapter files.
- `split_mp3_max_mb`: Max MP3 part size in MiB when format is `split_mp3_by_size`.
- `chapter_file_template`: Optional per-chapter file-stem template.
- `chapter_title_template`: Optional embedded chapter-title template.
- `minimum_file_duration_minutes`: Drop chapter splits shorter than this many minutes (`0` = keep all).
- `combine_nested_chapter_titles`: When true, flatten nested chapter titles into a single path segment.
- `merge_opening_and_end_credits`: When true, merge opening/end credit chapters into adjacent chapters.
- `strip_unabridged`: When true, strip an "Unabridged" suffix from titles used in naming.
- `strip_audible_brand_audio`: When true, trim Audible brand intro/outro from the remux window.
- `download_clips_bookmarks`: When true, download Audible clips/bookmarks sidecars when offered.
- `retain_aax_file`: When true, keep the encrypted download in storage (`RetainAaxFile`).
- `download_speed_limit_kbps`: Fetch speed cap in KB/s (`0` = unlimited).
- `lame`: LAME encoder knobs used when packaging to MP3.
- `max_sample_rate`: Optional ceiling for output sample rate in Hz (`None` = leave source rate).
- `creation_time`: How to set the file creation / birth timestamp after acquire.
- `last_write_time`: How to set the file last-write / mtime after acquire.
- `replacement_characters`: Resolved find/replace rules for path-segment sanitisation.
- `path_limits`: Filesystem / object-store path length limits for storage keys.
- `save_podcasts_to_parent_folder`: Save podcast episodes under the parent show folder.

#### DownloadOptions::from_fetch_options

```rust
pub fn from_fetch_options(fetch: &FetchOptions) -> Self
```

Guest-side reconstruction: host defaults overlaid with the fetch knobs
the host sent in `abi::FetchTitleParams::fetch`.

#### DownloadOptions::apply_fetch_options

```rust
pub fn apply_fetch_options(&mut self, fetch: &FetchOptions)
```

Overlays wire fetch knobs onto these options (packaging knobs untouched).

#### DownloadOptions::for_destination

```rust
pub fn for_destination(
    cfg: &Config,
    kind: OutputBackendKind,
    naming: &DestinationNaming,
    prefix: &str,
) -> Self
```

Build acquire options stamped for one output destination.

Packaging knobs come from global `[output]`; naming templates prefer
the destination's own overrides, which the `local` / `s3` plugin
declares as `[[settings]]` and the host reads from the deployed
instance document. `prefix` is that document's key prefix.

#### DownloadOptions::effective_output

```rust
pub fn effective_output(&self) -> OutputFormat
```

Resolved post-processing format.

#### DownloadOptions::wants_mp3

```rust
pub fn wants_mp3(&self) -> bool
```

True when the effective format re-encodes to MP3.

#### DownloadOptions::wants_split_by_chapter

```rust
pub fn wants_split_by_chapter(&self) -> bool
```

True when output should be split into per-chapter files.

#### DownloadOptions::is_noop_output

```rust
pub fn is_noop_output(&self) -> bool
```

True when store-delivered media is left as-is (no remux/transcode).

#### DownloadOptions::wants_opus

```rust
pub fn wants_opus(&self) -> bool
```

True when Opus packaging is selected (not yet implemented).

#### DownloadOptions::wants_split_by_size

```rust
pub fn wants_split_by_size(&self) -> bool
```

True when output is split into MP3 parts by target size.

#### DownloadOptions::chapter_json_flat

```rust
pub fn chapter_json_flat(&self) -> bool
```

True when a flat `chapters.flat.json` sidecar should be written.

#### DownloadOptions::chapter_json_tree

```rust
pub fn chapter_json_tree(&self) -> bool
```

True when a nested `chapters.tree.json` sidecar should be written.

#### DownloadOptions::wants_chapter_json

```rust
pub fn wants_chapter_json(&self) -> bool
```

True when any chapter JSON sidecar should be written.

#### DownloadOptions::naming_templates

```rust
pub fn naming_templates(&self) -> ResolvedNamingTemplates
```

Resolve folder / file / chapter-file templates from the naming profile
with per-field overrides.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `From<&Config>`, `From<&DownloadOptions>`, `From<&OutputConfig>`, `PartialEq`, `Serialize`

### SourceRegistry

```rust
pub struct SourceRegistry {
    /* private fields */
}
```

Maps a registry address to an installed [`ContentSource`](#contentsource).

A deployed source is addressed by [`ContentSource::plugin_instance_id`](#contentsourceplugin_instance_id).
A source with no instance id is addressed by [`ContentSource::plugin_key`](#contentsourceplugin_key).
A plugin key or display alias that several instances share routes to the
plugin's default instance (see [`Self::set_default_instance`](#sourceregistryset_default_instance)); a job may
pin another instance through [`Self::route`](#sourceregistryroute). Nothing fails closed on a
shared key: accounts belong to the plugin, so any instance may serve them.

#### SourceRegistry::new

```rust
pub fn new() -> Self
```

Empty registry with no sources registered.

#### SourceRegistry::register

```rust
pub fn register(&mut self, source: Arc<dyn ContentSource>)
```

Register (or replace) a source at its instance id, or its PluginKey.

A second instance of the same plugin key stays in the map. Re-registering
the same instance id replaces that guest only.

#### SourceRegistry::remove_instance

```rust
pub fn remove_instance(&mut self, plugin_instance_id: &str) -> bool
```

Drops the source registered under `plugin_instance_id`.

Other instances of the same plugin key stay. Returns whether a row was removed.

#### SourceRegistry::set_default_instance

```rust
pub fn set_default_instance(&mut self, plugin_key: &str, plugin_instance_id: &str)
```

Names the instance a bare plugin key or alias routes to when several
instances of `plugin_key` are registered.

#### SourceRegistry::default_instance

```rust
pub fn default_instance(&self, plugin_key: &str) -> Option<&str>
```

Default instance id recorded for `plugin_key`, if any.

#### SourceRegistry::get

```rust
pub fn get(&self, id_or_alias: &str) -> Option<Arc<dyn ContentSource>>
```

Look up a source by plugin instance id, PluginKey, or display alias.

An instance id matches that deployed source. A plugin key or alias
shared by several instances of **one** plugin routes to the default
instance, else the lowest instance id, so a bare key always resolves
while any instance is registered. An alias that two different plugins
claim is ambiguous and resolves to nothing: the caller names the
PluginKey or an instance id instead.

#### SourceRegistry::route

```rust
pub fn route(
    &self,
    id_or_alias: &str,
    preferred_instance: Option<&str>,
) -> Option<Arc<dyn ContentSource>>
```

[`Self::get`](#sourceregistryget) with an optional pinned instance.

`preferred_instance` must be a registered instance of the plugin that
`id_or_alias` names; a pin for another plugin, or an unknown id, yields
`None` rather than silently falling back.

#### SourceRegistry::require

```rust
pub fn require(&self, id_or_alias: &str) -> Result<Arc<dyn ContentSource>>
```

Look up a source or return `crate::SourceError::Api` when missing.

##### Errors

Returns an API error when `id_or_alias` is not registered.

#### SourceRegistry::require_routed

```rust
pub fn require_routed(
    &self,
    id_or_alias: &str,
    preferred_instance: Option<&str>,
) -> Result<Arc<dyn ContentSource>>
```

[`Self::require`](#sourceregistryrequire) with an optional pinned instance.

##### Errors

Returns an API error when nothing is registered under `id_or_alias`,
or when `preferred_instance` is not one of its instances.

#### SourceRegistry::resolve_id

```rust
pub fn resolve_id(&self, id_or_alias: &str) -> Option<String>
```

Resolve a needle to the canonical plugin id when registered.

#### SourceRegistry::all

```rust
pub fn all(&self) -> Vec<Arc<dyn ContentSource>>
```

All registered sources in stable plugin order.

#### SourceRegistry::routed

```rust
pub fn routed(&self) -> Vec<Arc<dyn ContentSource>>
```

One routed source per registered plugin key, in stable plugin order.

Accounts belong to the plugin, so a scan that visited every instance
of one key would read and write the same rows twice. Bare-key routing
picks the instance; a pinned job goes through [`Self::route`](#sourceregistryroute).

#### SourceRegistry::scan_all

```rust
pub async fn scan_all(
    &self,
    library: &LibraryStore,
    opts: ScanOptions,
) -> Result<ScanSummary>
```

Scan every registered plugin once (honoring per-source account filters).

When `opts.accounts` is non-empty, each source only receives the subset of
account needles that resolve to an account on that source. Sources with no
matching accounts are skipped instead of failing the whole multi-source scan.
`ScanOptions::cancel` is checked between sources.

##### Errors

Returns an error when the operation fails.

#### SourceRegistry::scan_one

```rust
pub async fn scan_one(
    &self,
    library: &LibraryStore,
    source: Arc<dyn ContentSource>,
    opts: ScanOptions,
) -> Result<ScanSummary>
```

Scan one registered source (a job pinned to a plugin instance).

Account filters apply as in [`Self::scan_all`](#sourceregistryscan_all).

##### Errors

Returns an error when the scan fails or no account matches.

Implements: `Clone`, `Default`

## Enums

### PortalAuthMode

```rust
pub enum PortalAuthMode {
    Oauth,
    Password,
}
```

How the connect portal authenticates this source.

**Variants**

- `Oauth`: Browser / QR OAuth (Audible).
- `Password`: Username + password form.

#### PortalAuthMode::as_str

```rust
pub fn as_str(self) -> &'static str
```

Wire / portal id (`oauth` or `password`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### revoke_credentials_default

```rust
pub async fn revoke_credentials_default(
    scope: &SourceScope,
    account_id: &str,
) -> Result<()>
```

Shared revoke path for plugins that seal as `.plugin.auth` (and legacy names).

#### Errors

Returns an error when the operation fails.

### source_scope

```rust
pub fn source_scope(library: &LibraryStore, source: &dyn ContentSource) -> SourceScope
```

Opens the [`SourceScope`](../bookclerk_library/scope.md#sourcescope) for `source`: rows are isolated by its
[`ContentSource::plugin_key`](#contentsourceplugin_key) and labelled with its [`ContentSource::id`](#contentsourceid).

Every instance of one plugin shares this scope; accounts are not tied
to a plugin instance.
