# bookclerk_source

Shared traits and DTOs for Bookclerk content-source plugins.

## Audience

Host registration code and first-party / third-party **source plugin**
authors implementing [`ContentSource`](#contentsource). Native guests and the plugin host
map these types onto the typed Cap'n Proto ABI through [`abi`](abi.md); workerd
guests use the generated TypeScript / Python SDK types instead.

Product narrative: `docs/plugins/README.md`. Style: `docs/contributing/code-documentation.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ContentSource`](#contentsource) | trait | Pluggable audiobook storefront (auth, scan, fetch, catalog). |
| [`SourceBrand`](#sourcebrand) | struct | Visual identity for a content source in the connect portal. |
| [`DownloadOptions`](#downloadoptions) | struct | Options for acquire / fetch packaging and naming. |
| [`SourceRegistry`](#sourceregistry) | struct | Maps a registry address to an installed `ContentSource`. |
| [`CatalogHit`](#cataloghit) | struct | Neutral catalog / candidate hit (no store crate types). |
| [`CatalogSearchOpts`](#catalogsearchopts) | struct | Options for `crate::ContentSource::search_catalog`. |
| [`ConfigOptionValue`](#configoptionvalue) | struct | One allowed value for a `SourceConfigOption`. |
| [`ExpandSeed`](#expandseed) | struct | Seed for related / series / author expansion. |
| [`FetchOptions`](#fetchoptions) | struct | Options passed to `crate::ContentSource::fetch_title`. |
| [`ImportCredentialsOptions`](#importcredentialsoptions) | struct | Options for `crate::ContentSource::import_credentials`. |
| [`LoginOptions`](#loginoptions) | struct | Options for interactive / CLI login. |
| [`PlainAudioPart`](#plainaudiopart) | struct | One DRM-free audio part (chapter file or single book). |
| [`PlainFetch`](#plainfetch) | struct | DRM-free fetch result. Sources that use DRM decrypt inside the plugin and return clear media here — the host never sees ciphertext or keys. |
| [`PurchaseHintOpts`](#purchasehintopts) | struct | Options for `crate::ContentSource::purchase_hint`. |
| [`ScanOptions`](#scanoptions) | struct | Options for a library scan. |
| [`ScanSummary`](#scansummary) | struct | Summary of a scan run. |
| [`SourceAccount`](#sourceaccount) | struct | Account discovered or created by a content source. |
| [`SourceConfigOption`](#sourceconfigoption) | struct | One source-native config knob under `[sources.<id>]`. |
| [`SourcePurchaseHint`](#sourcepurchasehint) | struct | URL + optional live price for one storefront edition. |
| [`SourceError`](#sourceerror) | enum | Errors from content-source auth, scan, or fetch. |
| [`PortalAuthMode`](#portalauthmode) | enum | How the connect portal authenticates this source. |
| [`CatalogSearchField`](#catalogsearchfield) | enum | Optional role / facet for `crate::ContentSource::search_catalog`. |
| [`CatalogSearchSort`](#catalogsearchsort) | enum | Sort mode for `crate::ContentSource::search_catalog`. |
| [`CatalogSortDir`](#catalogsortdir) | enum | Ascending / descending for catalog search host re-rank. |
| [`OAuthProgress`](#oauthprogress) | enum | Progress events for interactive OAuth login (portal / CLI). |
| [`default_preferred_language`](#default_preferred_language) | function | Preferred language for Discover when the client omits one. |
| [`language_rank`](#language_rank) | function | Rank a hit's language relative to the preferred code. |
| [`normalize_language`](#normalize_language) | function | Normalize a language tag or display name to a primary ISO-ish code. |
| [`audio_extension`](#audio_extension) | function | Prefer byte sniffing, then `Content-Type`, then URL path; `default` when all fail. |
| [`extension_from_bytes`](#extension_from_bytes) | function | Guess an audio extension from magic bytes (`infer`), with AAC/MP3 fallbacks. |
| [`extension_from_content_type`](#extension_from_content_type) | function | Map a `Content-Type` header (parameters stripped) to an audio extension. |
| [`extension_from_url`](#extension_from_url) | function | Guess an audio file extension from a URL path (query/fragment stripped). |
| [`revoke_credentials_default`](#revoke_credentials_default) | function | Shared revoke path for plugins that seal as `.plugin.auth` (and legacy names). |
| [`source_scope`](#source_scope) | function | Opens the `SourceScope` for `source`: rows are isolated by its `ContentSource::plugin_key` and labelled with its `ContentSource::id`. |
| [`Result`](#result) | type alias | Result alias for content-source operations. |
| [`SourceFetch`](#sourcefetch) | type alias | Result of fetching a title for acquire (always clear media). |

## Modules

- [`abi`](abi.md): Conversions between host storefront types and the typed Cap'n Proto plugin ABI (`bookclerk_plugin_abi`).

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

### SourceBrand

```rust
pub struct SourceBrand {
    pub id: &'static str,
    pub name: &'static str,
    pub bg: &'static str,
    pub fg: &'static str,
    pub accent: &'static str,
    pub icon_url: &'static str,
}
```

Visual identity for a content source in the connect portal.

**Fields**

- `id`: Stable plugin id matching [`crate::ContentSource::id`](#contentsourceid) (`audible`, …).
- `name`: Human-facing store name shown on the connect button.
- `bg`: CSS background color (hex) for the portal button.
- `fg`: CSS foreground / text color (hex) for the portal button.
- `accent`: CSS accent color (hex) for focus / highlights.
- `icon_url`: Remote favicon / app-icon URL (not stored in-repo).

#### SourceBrand::logo_href

```rust
pub fn logo_href(&self) -> &'static str
```

Favicon / app-icon URL used as the portal button logo (`href`).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`, `Serialize`

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

Look up a source or return [`crate::SourceError::Api`](#sourceerror) when missing.

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
[`ScanOptions::cancel`](#scanoptions) is checked between sources.

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

### CatalogHit

```rust
pub struct CatalogHit {
    pub product_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub url: Option<String>,
    pub cover_url: Option<String>,
    pub origin: String,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub rating_overall: Option<f64>,
    pub rating_count: Option<i64>,
    pub is_abridged: Option<bool>,
}
```

Neutral catalog / candidate hit (no store crate types).

Identity fields are required; bibliographic extras and list price are filled
when a storefront’s search/expand payload already includes them (avoids a
second Audnexus / purchase-hint round-trip for Discover detail).

**Fields**

- `product_id`: Store-native product / title id (ASIN, ISBN, UUID, …).
- `title`: Display title for the work.
- `authors`: Author names (`;`-separated when multiple).
- `narrators`: Narrator names (`;`-separated when multiple).
- `series`: Series name when the title belongs to a series.
- `series_index`: Position within the series (e.g. `"1"`, `"1-5"`).
- `asin`: Amazon ASIN when known (may match [`Self::product_id`](#cataloghit) for Audible).
- `isbn`: ISBN-10/13 when known.
- `url`: Canonical storefront product URL when available.
- `cover_url`: Public cover image URL when the storefront provides one.
- `origin`: How this was found (`related`, `series`, `author`, `search`, `top_deals`, …).
- `subtitle`: Subtitle / secondary title line when the storefront provides one.
- `description`: Marketing / synopsis text from the storefront.
- `publisher`: Publisher imprint name when known.
- `length_minutes`: Runtime in whole minutes when the storefront reports duration.
- `published_at`: ISO-8601 / store release date string when known.
- `categories`: Genre / subject labels (`;`-separated).
- `language`: Content language (BCP-47 or storefront display name).
- `price_cents`: List / deal price from the same catalog payload (optional).
- `currency`: ISO 4217 currency code for [`Self::price_cents`](#cataloghit) when priced.
- `price_label`: Pre-formatted price string for UI (e.g. `$14.95`).
- `rating_overall`: Community overall rating (0–5) when the storefront returned it.
- `rating_count`: Number of star ratings backing [`Self::rating_overall`](#cataloghit) when known.
- `is_abridged`: `Some(true)` when the storefront marks the title abridged; `Some(false)`
  for unabridged; `None` when unknown.

#### CatalogHit::decode_html_entities

```rust
pub fn decode_html_entities(self) -> Self
```

Decode HTML entities in human-readable metadata fields.

Storefronts (notably Libro.fm) sometimes return `Memory&#39;s Blade`-style
titles; normalize before discover merge / UI display.

No-op (no allocations) when no field contains `&`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `From<CatalogHit>`, `Serialize`

### CatalogSearchOpts

```rust
pub struct CatalogSearchOpts {
    pub query: String,
    pub region: String,
    pub limit: usize,
    pub page: u32,
    pub sort: CatalogSearchSort,
    pub field: Option<CatalogSearchField>,
    pub language: Option<String>,
}
```

Options for [`crate::ContentSource::search_catalog`](#contentsourcesearch_catalog).

**Fields**

- `query`: Free-text or facet search string.
- `region`: Marketplace / catalog region (`us`, `uk`, …); empty = plugin default.
- `limit`: Maximum hits to return (`0` = plugin default).
- `page`: 1-based page index for storefronts that support paging (default 1).
- `sort`: Preferred upstream / merge sort.
- `field`: When set, prefer storefront-native search for this facet.
- `language`: Preferred content language (BCP-47 primary or display name); soft-prioritize.

Implements: `Clone`, `Debug`, `Default`, `From<&CatalogSearchOpts>`, `From<SearchCatalogParams>`

### ConfigOptionValue

```rust
pub struct ConfigOptionValue {
    pub id: &'static str,
    pub label: &'static str,
}
```

One allowed value for a [`SourceConfigOption`](#sourceconfigoption).

**Fields**

- `id`: Wire / TOML id (`high`, `m4b`, `web`, …).
- `label`: Operator-facing display name for this choice in Settings / CLI help.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ExpandSeed

```rust
pub struct ExpandSeed {
    pub source: String,
    pub product_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_asin: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub region: String,
}
```

Seed for related / series / author expansion.

**Fields**

- `source`: Source id of the seed title (`chirp`, `audible`, …).
- `product_id`: Store-native product id of the owned / seed title.
- `title`: Display title used for related/series/author expansion queries.
- `authors`: Author names from the seed library row.
- `narrators`: Narrator names from the seed library row.
- `series`: Series name from the seed library row.
- `series_asin`: Parent Audible series ASIN when known (from library metadata).
- `asin`: Amazon ASIN when known for Audible-related expansion.
- `isbn`: ISBN when known for cross-store matching.
- `region`: Marketplace / catalog region (`us`, `uk`, …).

Implements: `Clone`, `Debug`

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
  [`bookclerk_library::LibraryStore`](../bookclerk_library/index.md#librarystore) passed directly to the trait method.
- `max_cache_bytes`: Optional on-disk budget for `cache_dir` (bytes). Sources should call
  [`Self::enforce_cache_budget`](#fetchoptionsenforce_cache_budget) between chunks.
- `cancel`: Cooperative cancel for mid-fetch quota or a lost job fence.

#### FetchOptions::enforce_cache_budget

```rust
pub fn enforce_cache_budget(&self) -> Result<()>
```

Fails when cancel is set or `cache_dir` exceeds [`Self::max_cache_bytes`](#fetchoptions).

##### Errors

Returns [`SourceError::Other`](#sourceerror) when the fetch was cancelled or the
cache directory is over the remaining scratch quota.

Implements: `Clone`, `Debug`

### ImportCredentialsOptions

```rust
pub struct ImportCredentialsOptions {
    pub libation_accounts: bool,
    pub mkb79: bool,
    pub label: Option<String>,
    pub force: bool,
}
```

Options for [`crate::ContentSource::import_credentials`](#contentsourceimport_credentials).

**Fields**

- `libation_accounts`: Treat input as classic Libation `AccountsSettings.json`.
- `mkb79`: Import mkb79 / audible-cli legacy auth JSON.
- `label`: Destination display label / filename stem.
- `force`: Overwrite an existing credential.

Implements: `Clone`, `Debug`, `Default`

### LoginOptions

```rust
pub struct LoginOptions {
    pub marketplace: String,
    pub label: Option<String>,
    pub email: Option<String>,
    pub password: Option<String>,
    pub force: bool,
    pub callback_bind: Option<String>,
    pub external: bool,
    pub response_url: Option<String>,
    pub show_qr: bool,
    pub timeout_secs: Option<u64>,
    pub extra: Value,
}
```

Options for interactive / CLI login.

**Fields**

- `marketplace`: Target marketplace / region for login (`us`, `uk`, …); empty = plugin default.
- `label`: Optional nickname stored with the new account credentials.
- `email`: Email/password sources; ignored for OAuth.
- `password`: Email/password sources; ignored for OAuth.
- `force`: When true, overwrite an existing credential blob for the same account.
- `callback_bind`: Optional OAuth callback bind (`host:port`) for portal / reverse-proxy use.
- `external`: External / paste-redirect OAuth instead of a local callback server.
- `response_url`: Pre-supplied OAuth redirect URL (with [`Self::external`](#loginoptions)).
- `show_qr`: Emit a terminal QR for the authorize URL when the source supports it.
- `timeout_secs`: Seconds to wait for OAuth callback capture (source-defined default when `None`).
- `extra`: Store-specific knobs (`audible_username`, `ascii_qr`, …). Guests may ignore unknowns.

Implements: `Clone`, `Debug`, `Default`, `From<LoginParams>`

### PlainAudioPart

```rust
pub struct PlainAudioPart {
    pub path: PathBuf,
    pub title: Option<String>,
    pub duration_ms: Option<u64>,
}
```

One DRM-free audio part (chapter file or single book).

**Fields**

- `path`: Absolute path to a clear (DRM-free) audio file on disk.
- `title`: Optional chapter / part title for packaging and metadata.
- `duration_ms`: Duration in milliseconds when known.

Implements: `Clone`, `Debug`, `From<PlainAudioPart>`, `From<PlainPart>`

### PlainFetch

```rust
pub struct PlainFetch {
    pub parts: Vec<PlainAudioPart>,
    pub m4b_path: Option<PathBuf>,
    pub cover_path: Option<PathBuf>,
    pub chapters: Vec<(String, u64)>,
    pub pdf_url: Option<String>,
}
```

DRM-free fetch result. Sources that use DRM decrypt inside the plugin and
return clear media here — the host never sees ciphertext or keys.

**Fields**

- `parts`: Ordered clear audio parts (single file or per-chapter downloads).
- `m4b_path`: Pre-built M4B from the store / plugin when available.
- `cover_path`: Absolute path to a downloaded cover image when available.
- `chapters`: Chapter titles paired with start offsets in milliseconds.
- `pdf_url`: Companion PDF download URL when the store exposes one.

Implements: `Clone`, `Debug`, `From<PlainFetch>`

### PurchaseHintOpts

```rust
pub struct PurchaseHintOpts {
    pub product_id: Option<String>,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub region: String,
    pub with_price: bool,
}
```

Options for [`crate::ContentSource::purchase_hint`](#contentsourcepurchase_hint).

**Fields**

- `product_id`: Preferred store-native product id when already known.
- `title`: Title used to disambiguate when `product_id` is absent.
- `authors`: Author names used to disambiguate catalog lookups.
- `asin`: Amazon ASIN used for Audible / cross-store hints.
- `isbn`: ISBN used for cross-store hints.
- `region`: Marketplace / catalog region (`us`, `uk`, …); empty = plugin default.
- `with_price`: When true, resolve live price if the source can.

Implements: `Clone`, `Debug`, `Default`, `From<&PurchaseHintOpts>`, `From<PurchaseHintParams>`

### ScanOptions

```rust
pub struct ScanOptions {
    pub accounts: Vec<String>,
    pub page_size: u32,
    pub import_episodes: bool,
    pub import_plus_titles: bool,
    pub cancel: Option<Arc<AtomicBool>>,
}
```

Options for a library scan.

**Fields**

- `accounts`: Limit to specific account nicknames / ids.
- `page_size`: Upstream library page size (titles per HTTP page; default 50).
- `import_episodes`: Import podcast episodes — consumed by plugins that support it.
- `import_plus_titles`: Import catalog Plus / non-owned titles — consumed by plugins that support it.
- `cancel`: Cooperative cancel checked between sources and pages.

#### ScanOptions::is_cancelled

```rust
pub fn is_cancelled(&self) -> bool
```

True when a job fence asked this scan to stop.

Implements: `Clone`, `Debug`, `Default`

### ScanSummary

```rust
pub struct ScanSummary {
    pub accounts: usize,
    pub books_upserted: usize,
    pub pages: u32,
    pub skipped_disabled: usize,
    pub warnings: Vec<String>,
}
```

Summary of a scan run.

**Fields**

- `accounts`: Number of accounts successfully scanned in this run.
- `books_upserted`: Library rows inserted or updated.
- `pages`: Upstream library pages fetched across all accounts.
- `skipped_disabled`: Accounts skipped because [`crate::SourceAccount::scan_enabled`](#sourceaccount) was false.
- `warnings`: Plugin keys skipped because more than one instance is registered.
  
  Other sources in the same scan still run. Accounts stay keyed by
  storefront id, so those keys are not scanned twice.

#### ScanSummary::merge

```rust
pub fn merge(&mut self, other: &Self)
```

Merge another source's summary into this one.

Implements: `Clone`, `Debug`, `Default`

### SourceAccount

```rust
pub struct SourceAccount {
    pub account_id: String,
    pub source: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub scan_enabled: bool,
}
```

Account discovered or created by a content source.

**Fields**

- `account_id`: Stable account id stored in the library (often email or store user id).
- `source`: Canonical plugin id (`audible`, `libro`, …).
- `marketplace`: Store marketplace / region code (`us`, `uk`, …).
- `label`: Optional operator-facing nickname for CLI / UI selection.
- `scan_enabled`: When false, scheduled / bare scans skip this account (explicit
  `--account` still includes it).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `From<SourceAccount>`, `Serialize`

### SourceConfigOption

```rust
pub struct SourceConfigOption {
    pub key: &'static str,
    pub label: &'static str,
    pub values: &'static [ConfigOptionValue],
}
```

One source-native config knob under `[sources.<id>]`.

**Fields**

- `key`: TOML key (`bitrate`, `container`, `access`).
- `label`: Operator-facing display name for this knob in Settings.
- `values`: Closed set of allowed wire values for [`Self::key`](#sourceconfigoption).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### SourcePurchaseHint

```rust
pub struct SourcePurchaseHint {
    pub product_id: String,
    pub title: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
}
```

URL + optional live price for one storefront edition.

**Fields**

- `product_id`: Store-native product id for the hinted edition.
- `title`: Display title when the storefront returns one with the hint.
- `url`: Deep link to buy / open the title on the storefront.
- `price_cents`: Primary / best known sell price in minor units (prefer member when dual).
- `currency`: ISO 4217 currency code for the primary price.
- `price_label`: Pre-formatted primary price string for UI.
- `list_price_cents`: Non-member / list / retail price when the storefront shows dual pricing.
- `list_price_label`: Pre-formatted non-member / list price for dual-price storefronts.
- `member_price_cents`: Member / subscriber price when distinct from list.
- `member_price_label`: Pre-formatted member / subscriber price when distinct from list.

#### SourcePurchaseHint::decode_html_entities

```rust
pub fn decode_html_entities(self) -> Self
```

Decode HTML entities in human-readable fields (no-op when no `&`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `From<PurchaseHint>`, `From<SourcePurchaseHint>`, `Serialize`

## Enums

### SourceError

```rust
pub enum SourceError {
    NoAccounts(String),
    Auth(String),
    Api(String),
    Io(Error),
    Library(LibraryError),
    Other(Error),
}
```

Errors from content-source auth, scan, or fetch.

**Variants**

- `NoAccounts`: No accounts are configured for this source (safe to skip in multi-source scan).
- `Auth`: Authentication or credential failure (operator-facing message).
- `Api`: Upstream store HTTP / protocol failure (operator-facing message).
- `Io`: Local filesystem failure while caching or writing downloads.
- `Library`: Library database / secret-store failure.
- `Other`: Catch-all for plugin-internal failures wrapped as [`anyhow::Error`](https://docs.rs/anyhow/latest/anyhow/struct.Error.html).

#### SourceError::no_accounts

```rust
pub fn no_accounts(msg: impl Into<String>) -> Self
```

Construct [`Self::NoAccounts`](#sourceerror) with an operator-facing message.

#### SourceError::auth

```rust
pub fn auth(msg: impl Into<String>) -> Self
```

Construct [`Self::Auth`](#sourceerror) with an operator-facing message.

#### SourceError::api

```rust
pub fn api(msg: impl Into<String>) -> Self
```

Construct [`Self::Api`](#sourceerror) with an operator-facing message.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<LibraryError>`

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

### CatalogSearchField

```rust
pub enum CatalogSearchField {
    Author,
    Narrator,
    Series,
    Genre,
}
```

Optional role / facet for [`crate::ContentSource::search_catalog`](#contentsourcesearch_catalog).

Free-text Discover typeahead leaves this unset. Meta links (author, narrator,
series, genre) set it so storefronts that support scoped params (e.g. Audible
`author=` / `narrator=`) can return the right catalog slice.

**Variants**

- `Author`: Scope search to titles by this author.
- `Narrator`: Scope search to titles narrated by this person.
- `Series`: Scope search to titles in this series.
- `Genre`: Scope search to this genre / category facet.

#### CatalogSearchField::from_wire

```rust
pub fn from_wire(raw: &str) -> Option<Self>
```

Parse a wire / query-string value (`author`, `narrators`, …).

#### CatalogSearchField::as_wire

```rust
pub fn as_wire(self) -> &'static str
```

Serialize to the portal / query-string wire id.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogSearchSort

```rust
pub enum CatalogSearchSort {
    Relevance,
    Popularity,
    Rating,
    Title,
    Author,
    Price,
    Length,
}
```

Sort mode for [`crate::ContentSource::search_catalog`](#contentsourcesearch_catalog).

Storefronts that lack a matching upstream sort ignore this (caller merges).

**Variants**

- `Relevance`: Storefront relevance ranking (default).
- `Popularity`: Audible `BestSellers` when available.
- `Rating`: Audible `AvgRating` when available.
- `Title`: Alphabetical by title (host or upstream).
- `Author`: Alphabetical by primary author (often host re-rank).
- `Price`: Catalog list/deal `price_cents` (host re-rank; sparse on some stores).
- `Length`: `length_minutes` (host re-rank).

#### CatalogSearchSort::from_wire

```rust
pub fn from_wire(raw: &str) -> Self
```

Parse a wire / query-string sort id (unknown → [`Self::Relevance`](#catalogsearchsort)).

#### CatalogSearchSort::as_wire

```rust
pub fn as_wire(self) -> &'static str
```

Serialize to the portal / query-string wire id.

#### CatalogSearchSort::audible_products_sort_by

```rust
pub fn audible_products_sort_by(self) -> &'static str
```

Audible `/1.0/catalog/products` `products_sort_by` value.

#### CatalogSearchSort::audible_catalog_search_sort

```rust
pub fn audible_catalog_search_sort(self) -> &'static str
```

Audible `/1.0/catalog/search` `sort` (`sort_option_id`) value.

Prefer this endpoint for Discover keyword browse — `/catalog/products`
keyword/title filters often omit major titles that still resolve by ASIN.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<CatalogSearchSort>`, `From<CatalogSort>`, `PartialEq`, `Serialize`

### CatalogSortDir

```rust
pub enum CatalogSortDir {
    Asc,
    Desc,
}
```

Ascending / descending for catalog search host re-rank.

**Variants**

- `Asc`: Ascending order.
- `Desc`: Descending order (default for popularity / rating / price).

#### CatalogSortDir::from_wire

```rust
pub fn from_wire(raw: &str) -> Self
```

Parse a wire / query-string direction (`asc` / `desc`; unknown → [`Self::Desc`](#catalogsortdir)).

#### CatalogSortDir::as_wire

```rust
pub fn as_wire(self) -> &'static str
```

Serialize to the portal / query-string wire id.

#### CatalogSortDir::default_for_sort

```rust
pub fn default_for_sort(sort: CatalogSearchSort) -> Self
```

Default direction when the client omits `sort_dir`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OAuthProgress

```rust
pub enum OAuthProgress {
    LoginUrl { url: String, qr: Option<String> },
    CallbackListening { addr: String },
    WaitingForCallback,
    Completed { account_id: String },
}
```

Progress events for interactive OAuth login (portal / CLI).

**Variants**

- `LoginUrl`: Browser URL the operator should open (optional pre-rendered QR text).
  - `url`: Absolute HTTPS URL for the browser / QR payload.
  - `qr`: Optional pre-rendered QR text for terminal UIs.
- `CallbackListening`: Local callback server is listening (SSH port-forward hint).
  - `addr`: Listen address (host:port) of the local redirect receiver.
- `WaitingForCallback`: Waiting for the OAuth redirect / callback.
- `Completed`: Login finished; credentials are persisted for `account_id`.
  - `account_id`: Account id stored after a successful login.

Implements: `Clone`, `Debug`

## Functions

### default_preferred_language

```rust
pub fn default_preferred_language() -> &'static str
```

Preferred language for Discover when the client omits one.

### language_rank

```rust
pub fn language_rank(hit_language: Option<&str>, preferred: &str) -> u8
```

Rank a hit's language relative to the preferred code.

- `0` — matches preferred
- `1` — unknown / missing
- `2` — other language

### normalize_language

```rust
pub fn normalize_language(raw: &str) -> Option<String>
```

Normalize a language tag or display name to a primary ISO-ish code.

Accepts BCP-47 (`en-US`), bare codes (`en`), and common Audible display
names (`english`). Returns `None` for empty / unrecognized input.

### audio_extension

```rust
pub fn audio_extension(
    url: &str,
    bytes: Option<&[u8]>,
    content_type: Option<&str>,
    default: &'static str,
) -> &'static str
```

Prefer byte sniffing, then `Content-Type`, then URL path; `default` when all fail.

### extension_from_bytes

```rust
pub fn extension_from_bytes(bytes: &[u8]) -> Option<&'static str>
```

Guess an audio extension from magic bytes (`infer`), with AAC/MP3 fallbacks.

### extension_from_content_type

```rust
pub fn extension_from_content_type(content_type: &str) -> Option<&'static str>
```

Map a `Content-Type` header (parameters stripped) to an audio extension.

### extension_from_url

```rust
pub fn extension_from_url(url: &str) -> Option<&'static str>
```

Guess an audio file extension from a URL path (query/fragment stripped).

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

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, SourceError>;
```

Result alias for content-source operations.

### SourceFetch

```rust
pub type SourceFetch = PlainFetch;
```

Result of fetching a title for acquire (always clear media).

Historically a dual Encrypted/Plain enum; DRM is plugin-owned now, so this
is an alias of [`PlainFetch`](#plainfetch). Prefer `PlainFetch` in new code.
