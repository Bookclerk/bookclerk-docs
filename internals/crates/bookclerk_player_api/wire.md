# bookclerk_player_api::wire

The response shapes player apps read.

Field names and types follow what apps written for an Audiobookshelf
server parse: camelCase keys, lengths of time in seconds as JSON
numbers ([`crate::time`](time.md)), times of day in milliseconds since the Unix
epoch, and `null` (not an absent key) for an unknown value. Books are
the only media type: no shape here carries a podcast field, because
some apps tell a book from a podcast by which fields are present.

Items carry one metadata shape whether listed or fetched alone: the
listing's flat names (`authorName`, `seriesName`) and the expanded
arrays (`authors`, `series`) together, so an app reading either finds
what it needs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Status`](#status) | struct | `GET /status`: what an app asks before it signs in. |
| [`AuthFormData`](#authformdata) | struct | Text an app shows on its sign-in screen. |
| [`Ping`](#ping) | struct | `GET /ping`: `{ "success": true }`. |
| [`LoginResponse`](#loginresponse) | struct | The answer to a sign-in, a token refresh, `POST /api/authorize` and the browser sign-in's callback. |
| [`User`](#user) | struct | A patron as apps see them. |
| [`Permissions`](#permissions) | struct | What a patron may do, as apps read it. |
| [`ServerSettings`](#serversettings) | struct | The server settings apps read. |
| [`Libraries`](#libraries) | struct | `GET /api/libraries`. |
| [`Library`](#library) | struct | One library an app can open. |
| [`LibrarySettings`](#librarysettings) | struct | A library's display settings. |
| [`LibraryWithFilterData`](#librarywithfilterdata) | struct | `GET /api/libraries/{id}?include=filterdata`. |
| [`FilterData`](#filterdata) | struct | The values an app offers as filters. |
| [`NamedRef`](#namedref) | struct | An id and a name. |
| [`SeriesRef`](#seriesref) | struct | A series an item belongs to, with the item's place in it. |
| [`ItemPage`](#itempage) | struct | A page of items: `GET /api/libraries/{id}/items` and the like. |
| [`LibraryItem`](#libraryitem) | struct | An item: an edition with a stored copy. |
| [`BookMedia`](#bookmedia) | struct | A book's media: metadata, tracks and chapters. |
| [`BookMetadata`](#bookmetadata) | struct | A book's metadata. |
| [`LibraryFile`](#libraryfile) | struct | A file of an item, in the expanded item's `libraryFiles`. |
| [`FileMetadata`](#filemetadata) | struct | A file's name, size and times. |
| [`AudioFile`](#audiofile) | struct | An audio file of a book. |
| [`AudioTrack`](#audiotrack) | struct | A track with its place in the book. |
| [`Chapter`](#chapter) | struct | A chapter of a book. |
| [`MediaProgress`](#mediaprogress) | struct | The patron's progress on one item. |
| [`Bookmark`](#bookmark) | struct | A bookmark. |
| [`SessionDevice`](#sessiondevice) | struct | The device a playback session was opened from. |
| [`PlaybackSession`](#playbacksession) | struct | A playback session, as `POST /api/items/{id}/play` answers. |
| [`SessionPage`](#sessionpage) | struct | `GET /api/me/listening-sessions`. |
| [`ListeningStats`](#listeningstats) | struct | `GET /api/me/listening-stats`. |
| [`ItemListening`](#itemlistening) | struct | Seconds listened to one item. |
| [`LibraryItems`](#libraryitems) | struct | `{ "libraryItems": [...] }`: in-progress items and batch reads. |
| [`Series`](#series) | struct | A series. |
| [`SeriesProgress`](#seriesprogress) | struct | The patron's progress through a series. |
| [`SeriesPage`](#seriespage) | struct | A page of series. |
| [`Author`](#author) | struct | An author. |
| [`AuthorSeries`](#authorseries) | struct | A series of an author's, with the author's items in it. |
| [`Authors`](#authors) | struct | `GET /api/libraries/{id}/authors` without paging. |
| [`AuthorPage`](#authorpage) | struct | A page of authors. |
| [`Narrator`](#narrator) | struct | A narrator with how many items they read. |
| [`Narrators`](#narrators) | struct | `GET /api/libraries/{id}/narrators`. |
| [`Shelf`](#shelf) | struct | A personalized shelf on the home screen. |
| [`SearchResults`](#searchresults) | struct | `GET /api/libraries/{id}/search`. |
| [`SearchBook`](#searchbook) | struct | An item that matched a search. |
| [`SearchSeries`](#searchseries) | struct | A series that matched a search. |
| [`LibraryStats`](#librarystats) | struct | `GET /api/libraries/{id}/stats`. |
| [`EmptyPage`](#emptypage) | struct | An empty page, for collections and playlists the library does not have yet. |
| [`Playlists`](#playlists) | struct | `{ "playlists": [] }`: the patron's playlists. |
| [`SyncResult`](#syncresult) | struct | One entry's outcome in a batch sync. |
| [`SyncResults`](#syncresults) | struct | `POST /api/session/local-all`. |
| [`ProgressEvent`](#progressevent) | struct | `user_item_progress_updated` on the socket: progress changed on another device. |
| [`SocketInit`](#socketinit) | struct | `init` on the socket: the socket is signed in. |
| [`SocketAuthFailed`](#socketauthfailed) | struct | `auth_failed` on the socket. |
| [`MetaTags`](#metatags) | struct | The tags read from an audio file, as apps read them: always empty until the stored media probe reads tags, so it serializes as `{}`. |
| [`ShelfEntities`](#shelfentities) | enum | A shelf's entries. |
| [`title_ignore_prefix`](#title_ignore_prefix) | function | The name an item's title sorts by: a leading `The `, `A ` or `An ` moved to the end (`Hobbit, The`), as apps show it in sorted lists. |
| [`last_first`](#last_first) | function | A name family name first (`Le Guin, Ursula K.`): the last word moved to the front. A one-word name is returned as it is. |
| [`plain_text`](#plain_text) | function | Text with markup removed: everything between `<` and `>` dropped and the common entities decoded. |
| [`MEDIA_TYPE_BOOK`](#media_type_book) | constant | The media type of every library and item the API lists. |
| [`STATUS_APP`](#status_app) | constant | The `app` the status answer names. Apps check it before they offer a sign-in, so it is the value they expect. |
| [`AUTH_METHODS`](#auth_methods) | constant | The sign-in methods an app may offer: a player password, and the browser sign-in through the library's own sign-in page. |

## Structs

### Status

```rust
pub struct Status {
    pub app: &'static str,
    pub server_version: &'static str,
    pub is_init: bool,
    pub language: &'static str,
    pub auth_methods: Vec<&'static str>,
    pub auth_form_data: AuthFormData,
}
```

`GET /status`: what an app asks before it signs in.

**Fields**

- `app`: Always [`STATUS_APP`](#status_app).
- `server_version`: The server release whose API this one answers like
  ([`crate::COMPATIBLE_SERVER_VERSION`](README.md#compatible_server_version)).
- `is_init`: Always true: the library is set up before an app can sign in.
- `language`: The interface language apps show server text in.
- `auth_methods`: [`AUTH_METHODS`](#auth_methods).
- `auth_form_data`: What the sign-in screen shows.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### AuthFormData

```rust
pub struct AuthFormData {
    pub auth_login_custom_message: Option<String>,
    pub auth_openid_button_text: String,
    pub auth_openid_auto_launch: bool,
}
```

Text an app shows on its sign-in screen.

**Fields**

- `auth_login_custom_message`: A note under the sign-in form: where to get a player password.
- `auth_openid_button_text`: The label of the browser sign-in button.
- `auth_openid_auto_launch`: Whether apps open the browser sign-in without a tap: never.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Ping

```rust
pub struct Ping {
    pub success: bool,
}
```

`GET /ping`: `{ "success": true }`.

**Fields**

- `success`: Always true.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### LoginResponse

```rust
pub struct LoginResponse {
    pub user: User,
    pub user_default_library_id: String,
    pub server_settings: ServerSettings,
    pub ereader_devices: Vec<Value>,
    pub source: &'static str,
}
```

The answer to a sign-in, a token refresh, `POST /api/authorize` and
the browser sign-in's callback.

**Fields**

- `user`: The signed-in patron, with the card's tokens.
- `user_default_library_id`: The library an app opens first: the whole library.
- `server_settings`: Settings apps read at sign-in.
- `ereader_devices`: E-reader devices: always empty, the API serves no ebooks.
- `source`: How the server was installed; apps only display it.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### User

```rust
pub struct User {
    pub id: String,
    pub username: String,
    pub email: Option<String>,
    pub kind: &'static str,
    pub token: String,
    pub access_token: Option<String>,
    pub refresh_token: Option<String>,
    pub is_old_token: bool,
    pub media_progress: Vec<MediaProgress>,
    pub series_hide_from_continue_listening: Vec<String>,
    pub bookmarks: Vec<Bookmark>,
    pub is_active: bool,
    pub is_locked: bool,
    pub last_seen: Option<i64>,
    pub created_at: i64,
    pub permissions: Permissions,
    pub libraries_accessible: Vec<String>,
    pub item_tags_selected: Vec<String>,
    pub has_openid_link: bool,
}
```

A patron as apps see them.

**Fields**

- `id`: The patron's portal identity id, as a string.
- `username`: The name the patron signs in with.
- `email`: The patron's email address, if the library has one.
- `kind`: Always `user`: no app is given administrator powers.
- `token`: The access token, for apps that read it from here (older apps
  send this one as their bearer). Empty where the answer is not a
  sign-in.
- `access_token`: The card's access token, on sign-in and refresh answers only.
- `refresh_token`: The card's refresh token, on sign-in and refresh answers only.
- `is_old_token`: Always false: tokens are never the old non-expiring kind.
- `media_progress`: Progress on every item the patron has started.
- `series_hide_from_continue_listening`: Series hidden from continue listening: none.
- `bookmarks`: Every bookmark the patron has made.
- `is_active`: Always true: a disabled patron cannot sign in.
- `is_locked`: Always false.
- `last_seen`: When the patron was last seen, in epoch milliseconds.
- `created_at`: When the patron joined, in epoch milliseconds.
- `permissions`: What the patron may do.
- `libraries_accessible`: Libraries the patron may open, by id.
- `item_tags_selected`: Tags that limit what the patron sees: none (scope is applied by
  the host, not by tags).
- `has_openid_link`: Whether the patron signed in through the browser at least once.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Permissions

```rust
pub struct Permissions {
    pub download: bool,
    pub update: bool,
    pub delete: bool,
    pub upload: bool,
    pub create_ereader: bool,
    pub access_all_libraries: bool,
    pub access_all_tags: bool,
    pub access_explicit_content: bool,
    pub selected_tags_not_accessible: bool,
}
```

What a patron may do, as apps read it.

**Fields**

- `download`: Whether the patron may download items to the device.
- `update`: Always false: apps edit nothing.
- `delete`: Always false.
- `upload`: Always false.
- `create_ereader`: Always false.
- `access_all_libraries`: Always true: the libraries listed are the ones the patron sees.
- `access_all_tags`: Always true.
- `access_explicit_content`: Always true: scope, not an explicit flag, decides visibility.
- `selected_tags_not_accessible`: Always false.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### ServerSettings

```rust
pub struct ServerSettings {
    pub id: &'static str,
    pub version: &'static str,
    pub build_number: u32,
    pub language: &'static str,
    pub date_format: &'static str,
    pub time_format: &'static str,
    pub sorting_ignore_prefix: bool,
    pub sorting_prefixes: Vec<&'static str>,
    pub chromecast_enabled: bool,
    pub home_bookshelf_view: u8,
    pub bookshelf_view: u8,
    pub auth_active_auth_methods: Vec<&'static str>,
    pub auth_openid_button_text: String,
    pub auth_openid_auto_launch: bool,
    pub auth_login_custom_message: Option<String>,
    pub time_zone: String,
}
```

The server settings apps read.

**Fields**

- `id`: Always `server-settings`.
- `version`: [`crate::COMPATIBLE_SERVER_VERSION`]: apps turn features on by it.
- `build_number`: Always 1.
- `language`: The interface language.
- `date_format`: How apps format dates.
- `time_format`: How apps format times.
- `sorting_ignore_prefix`: Whether sorting skips leading articles: no.
- `sorting_prefixes`: The articles sorting would skip.
- `chromecast_enabled`: Whether apps offer casting: no.
- `home_bookshelf_view`: The home shelf style (1: standard).
- `bookshelf_view`: The library shelf style (1: standard).
- `auth_active_auth_methods`: [`AUTH_METHODS`](#auth_methods).
- `auth_openid_button_text`: The browser sign-in button's label.
- `auth_openid_auto_launch`: Never true.
- `auth_login_custom_message`: The sign-in screen's note.
- `time_zone`: The host's time zone name, or UTC.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Libraries

```rust
pub struct Libraries {
    pub libraries: Vec<Library>,
}
```

`GET /api/libraries`.

**Fields**

- `libraries`: The libraries the patron may open.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Library

```rust
pub struct Library {
    pub id: String,
    pub name: String,
    pub folders: Vec<Value>,
    pub display_order: u32,
    pub icon: &'static str,
    pub media_type: &'static str,
    pub provider: &'static str,
    pub settings: LibrarySettings,
    pub last_scan: Option<i64>,
    pub last_scan_version: Option<String>,
    pub created_at: i64,
    pub last_update: i64,
}
```

One library an app can open.

**Fields**

- `id`: The library id ([`crate::ids::WHOLE_LIBRARY_ID`](ids.md#whole_library_id) for the whole
  library).
- `name`: The library's name.
- `folders`: Folders on disk: none are shown.
- `display_order`: The library's place in the list, from 1.
- `icon`: The icon apps show (`audiobookshelf` names a book stack).
- `media_type`: Always [`MEDIA_TYPE_BOOK`](#media_type_book).
- `provider`: The metadata provider apps show: none.
- `settings`: Display settings.
- `last_scan`: Always null.
- `last_scan_version`: Always null.
- `created_at`: When the library was made, in epoch milliseconds.
- `last_update`: When it last changed, in epoch milliseconds.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### LibrarySettings

```rust
pub struct LibrarySettings {
    pub cover_aspect_ratio: u8,
    pub disable_watcher: bool,
    pub audiobooks_only: bool,
    pub hide_single_book_series: bool,
    pub only_show_later_books_in_continue_series: bool,
    pub epubs_allow_scripted_content: bool,
    pub mark_as_finished_percent_complete: Option<f64>,
    pub mark_as_finished_time_remaining: f64,
}
```

A library's display settings.

**Fields**

- `cover_aspect_ratio`: 1 for square covers, 2 for the 1.6:1 book shape: square.
- `disable_watcher`: Always true: the library is not a watched folder.
- `audiobooks_only`: Always true: audiobooks only.
- `hide_single_book_series`: Whether one-book series are hidden from the series list: no.
- `only_show_later_books_in_continue_series`: Whether continue series skips earlier books: no.
- `epubs_allow_scripted_content`: Always false.
- `mark_as_finished_percent_complete`: An item counts as finished at this share listened, in percent.
- `mark_as_finished_time_remaining`: An item counts as finished with this many seconds left.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### LibraryWithFilterData

```rust
pub struct LibraryWithFilterData {
    pub filterdata: FilterData,
    pub issues: u64,
    pub num_user_playlists: u64,
    pub library: Library,
}
```

`GET /api/libraries/{id}?include=filterdata`.

**Fields**

- `filterdata`: The filter values.
- `issues`: Items with problems: none are reported.
- `num_user_playlists`: The patron's playlists in the library: none.
- `library`: The library.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### FilterData

```rust
pub struct FilterData {
    pub authors: Vec<NamedRef>,
    pub genres: Vec<String>,
    pub tags: Vec<String>,
    pub series: Vec<NamedRef>,
    pub narrators: Vec<String>,
    pub languages: Vec<String>,
    pub publishers: Vec<String>,
    pub published_decades: Vec<String>,
    pub book_count: u64,
    pub author_count: u64,
    pub series_count: u64,
    pub podcast_count: u64,
    pub number_episodes: u64,
    pub loaded_at: i64,
}
```

The values an app offers as filters.

**Fields**

- `authors`: Authors with at least one item.
- `genres`: Genres.
- `tags`: Tags: none are shown.
- `series`: Series with at least one item.
- `narrators`: Narrators.
- `languages`: Languages.
- `publishers`: Publishers.
- `published_decades`: Decades of publication: none are offered.
- `book_count`: Items in the library.
- `author_count`: Authors in the library.
- `series_count`: Series in the library.
- `podcast_count`: Always 0.
- `number_episodes`: Always 0.
- `loaded_at`: When the values were read, in epoch milliseconds.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### NamedRef

```rust
pub struct NamedRef {
    pub id: String,
    pub name: String,
}
```

An id and a name.

**Fields**

- `id`: The entity's id.
- `name`: Its display name.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SeriesRef

```rust
pub struct SeriesRef {
    pub id: String,
    pub name: String,
    pub sequence: Option<String>,
}
```

A series an item belongs to, with the item's place in it.

**Fields**

- `id`: The series id.
- `name`: The series name.
- `sequence`: The item's place in the series as the library writes it (`1`,
  `2.5`), if known.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### ItemPage

```rust
pub struct ItemPage {
    pub results: Vec<LibraryItem>,
    pub total: u64,
    pub limit: u64,
    pub page: u64,
    pub sort_by: Option<String>,
    pub sort_desc: bool,
    pub filter_by: Option<String>,
    pub media_type: &'static str,
    pub minified: bool,
    pub collapseseries: bool,
    pub include: String,
    pub offset: u64,
}
```

A page of items: `GET /api/libraries/{id}/items` and the like.

**Fields**

- `results`: The page's items.
- `total`: Items matching the query in all.
- `limit`: Items per page (0: no paging).
- `page`: The page number, from 0.
- `sort_by`: The sort key the app sent, echoed.
- `sort_desc`: Whether the order is reversed.
- `filter_by`: The filter the app sent, echoed.
- `media_type`: Always [`MEDIA_TYPE_BOOK`](#media_type_book).
- `minified`: Whether items are minified.
- `collapseseries`: Whether series were collapsed: never.
- `include`: The include list the app sent, echoed.
- `offset`: The offset of the first result.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### LibraryItem

```rust
pub struct LibraryItem {
    pub id: String,
    pub ino: String,
    pub old_library_item_id: Option<String>,
    pub library_id: String,
    pub folder_id: String,
    pub path: String,
    pub rel_path: String,
    pub is_file: bool,
    pub mtime_ms: i64,
    pub ctime_ms: i64,
    pub birthtime_ms: i64,
    pub added_at: i64,
    pub updated_at: i64,
    pub is_missing: bool,
    pub is_invalid: bool,
    pub media_type: &'static str,
    pub media: BookMedia,
    pub num_files: u64,
    pub size: u64,
    pub library_files: Option<Vec<LibraryFile>>,
    pub user_media_progress: Option<MediaProgress>,
    pub series_sequence: Option<String>,
}
```

An item: an edition with a stored copy.

**Fields**

- `id`: The item id: the edition id.
- `ino`: The item's inode as apps expect one: the edition id again.
- `old_library_item_id`: Always null.
- `library_id`: The library the item is listed in.
- `folder_id`: The folder id: the library id.
- `path`: A display path: the title.
- `rel_path`: A display path: the title.
- `is_file`: Always false: an item is a folder of tracks.
- `mtime_ms`: When the copy last changed, in epoch milliseconds.
- `ctime_ms`: When the copy was stored, in epoch milliseconds.
- `birthtime_ms`: When the copy was stored, in epoch milliseconds.
- `added_at`: When the item joined the library, in epoch milliseconds.
- `updated_at`: When its record last changed, in epoch milliseconds.
- `is_missing`: Always false.
- `is_invalid`: Always false.
- `media_type`: Always [`MEDIA_TYPE_BOOK`](#media_type_book).
- `media`: The book.
- `num_files`: Files in the item.
- `size`: Bytes in all its files.
- `library_files`: The item's files, when the item is fetched expanded.
- `user_media_progress`: The patron's progress, when the app asked for it.
- `series_sequence`: The item's place in a series, on a series' book list.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### BookMedia

```rust
pub struct BookMedia {
    pub id: String,
    pub library_item_id: Option<String>,
    pub metadata: BookMetadata,
    pub cover_path: Option<String>,
    pub tags: Vec<String>,
    pub num_tracks: u64,
    pub num_audio_files: u64,
    pub num_chapters: u64,
    pub duration: f64,
    pub size: u64,
    pub audio_files: Option<Vec<AudioFile>>,
    pub chapters: Option<Vec<Chapter>>,
    pub tracks: Option<Vec<AudioTrack>>,
}
```

A book's media: metadata, tracks and chapters.

**Fields**

- `id`: The media id: the edition id.
- `library_item_id`: The item id, when fetched expanded.
- `metadata`: The book's metadata.
- `cover_path`: A cover path apps only test for presence; covers are fetched from
  `/api/items/{id}/cover`.
- `tags`: Tags: none.
- `num_tracks`: Tracks in play order.
- `num_audio_files`: Audio files.
- `num_chapters`: Chapters.
- `duration`: The book's length in seconds.
- `size`: Bytes in its audio files.
- `audio_files`: The audio files, when fetched expanded.
- `chapters`: The chapters, when fetched expanded.
- `tracks`: The tracks with their place in the book, when fetched expanded.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### BookMetadata

```rust
pub struct BookMetadata {
    pub title: String,
    pub title_ignore_prefix: String,
    pub subtitle: Option<String>,
    pub authors: Vec<NamedRef>,
    pub author_name: String,
    pub author_name_lf: String,
    pub narrators: Vec<String>,
    pub narrator_name: String,
    pub series: Vec<SeriesRef>,
    pub series_name: String,
    pub genres: Vec<String>,
    pub published_year: Option<String>,
    pub published_date: Option<String>,
    pub publisher: Option<String>,
    pub description: Option<String>,
    pub description_plain: Option<String>,
    pub isbn: Option<String>,
    pub asin: Option<String>,
    pub language: Option<String>,
    pub explicit: bool,
    pub abridged: bool,
}
```

A book's metadata.

**Fields**

- `title`: The title.
- `title_ignore_prefix`: The title for sorting.
- `subtitle`: The subtitle, if any.
- `authors`: The authors.
- `author_name`: The authors' names joined with `, `.
- `author_name_lf`: The authors' names, family name first, joined with `, `.
- `narrators`: The narrators' names.
- `narrator_name`: The narrators' names joined with `, `.
- `series`: The series the book is in.
- `series_name`: The series names with the book's place (`Name #2`), joined.
- `genres`: Genres.
- `published_year`: The year of publication, as a string (`"2019"`).
- `published_date`: The date of publication, if known.
- `publisher`: The publisher.
- `description`: The description, as the library stores it.
- `description_plain`: The description with markup removed.
- `isbn`: The ISBN, if known.
- `asin`: The ASIN, if known.
- `language`: The language.
- `explicit`: Always false.
- `abridged`: Whether the edition is abridged.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### LibraryFile

```rust
pub struct LibraryFile {
    pub ino: String,
    pub metadata: FileMetadata,
    pub added_at: i64,
    pub updated_at: i64,
    pub file_type: &'static str,
}
```

A file of an item, in the expanded item's `libraryFiles`.

**Fields**

- `ino`: The file's `ino` ([`crate::ids::file_ino`](ids.md#file_ino)).
- `metadata`: The file's name, size and times.
- `added_at`: When the file was stored, in epoch milliseconds.
- `updated_at`: When the file was stored, in epoch milliseconds.
- `file_type`: Always `audio`.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### FileMetadata

```rust
pub struct FileMetadata {
    pub filename: String,
    pub ext: String,
    pub path: String,
    pub rel_path: String,
    pub size: u64,
    pub mtime_ms: i64,
    pub ctime_ms: i64,
    pub birthtime_ms: i64,
}
```

A file's name, size and times.

**Fields**

- `filename`: The file name.
- `ext`: The extension with its dot (`.m4b`).
- `path`: A display path: the file name.
- `rel_path`: A display path: the file name.
- `size`: The file's size in bytes.
- `mtime_ms`: When the file was stored, in epoch milliseconds.
- `ctime_ms`: When the file was stored, in epoch milliseconds.
- `birthtime_ms`: When the file was stored, in epoch milliseconds.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### AudioFile

```rust
pub struct AudioFile {
    pub index: u64,
    pub ino: String,
    pub metadata: FileMetadata,
    pub added_at: i64,
    pub updated_at: i64,
    pub track_num_from_meta: Option<u64>,
    pub disc_num_from_meta: Option<u64>,
    pub track_num_from_filename: Option<u64>,
    pub disc_num_from_filename: Option<u64>,
    pub manually_verified: bool,
    pub exclude: bool,
    pub error: Option<String>,
    pub format: Option<String>,
    pub duration: f64,
    pub bit_rate: Option<u64>,
    pub language: Option<String>,
    pub codec: Option<String>,
    pub time_base: Option<String>,
    pub channels: Option<u64>,
    pub channel_layout: Option<String>,
    pub chapters: Vec<Chapter>,
    pub embedded_cover_art: Option<String>,
    pub meta_tags: MetaTags,
    pub mime_type: String,
}
```

An audio file of a book.

**Fields**

- `index`: The file's place in play order, from 1.
- `ino`: The file's `ino` ([`crate::ids::file_ino`](ids.md#file_ino)).
- `metadata`: The file's name, size and times.
- `added_at`: When the file was stored, in epoch milliseconds.
- `updated_at`: When the file was stored, in epoch milliseconds.
- `track_num_from_meta`: The track number the file's tags state: not read.
- `disc_num_from_meta`: The disc number the file's tags state: not read.
- `track_num_from_filename`: The track number in the file name: not read.
- `disc_num_from_filename`: The disc number in the file name: not read.
- `manually_verified`: Always false.
- `exclude`: Always false.
- `error`: Always null.
- `format`: The container format, such as `mp4` or `mp3`.
- `duration`: The file's length in seconds.
- `bit_rate`: Bits per second, if known.
- `language`: The file's language, if known.
- `codec`: The codec, if known.
- `time_base`: The time base, if known.
- `channels`: Audio channels, if known.
- `channel_layout`: The channel layout, if known.
- `chapters`: Chapters embedded in the file: none are read.
- `embedded_cover_art`: Always null.
- `meta_tags`: Tags read from the file: none until the stored media probe reads
  them.
- `mime_type`: The file's media type, such as `audio/mp4`.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### AudioTrack

```rust
pub struct AudioTrack {
    pub index: u64,
    pub start_offset: f64,
    pub duration: f64,
    pub title: String,
    pub content_url: String,
    pub mime_type: String,
    pub codec: Option<String>,
    pub metadata: Option<FileMetadata>,
}
```

A track with its place in the book.

**Fields**

- `index`: The track's place in play order, from 1.
- `start_offset`: Where the track starts in the book, in seconds.
- `duration`: The track's length in seconds.
- `title`: The track's title: its file name.
- `content_url`: The path the track streams from, under the API's base address
  (`/api/items/{id}/file/{ino}`); apps add `?token=`.
- `mime_type`: The track's media type.
- `codec`: The codec, if known.
- `metadata`: The file's name, size and times.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Chapter

```rust
pub struct Chapter {
    pub id: u64,
    pub start: f64,
    pub end: f64,
    pub title: String,
}
```

A chapter of a book.

**Fields**

- `id`: The chapter's place, from 0.
- `start`: Where it starts, in seconds.
- `end`: Where it ends, in seconds.
- `title`: Its title.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### MediaProgress

```rust
pub struct MediaProgress {
    pub id: String,
    pub user_id: String,
    pub library_item_id: String,
    pub episode_id: Option<String>,
    pub media_item_id: String,
    pub media_item_type: &'static str,
    pub duration: f64,
    pub progress: f64,
    pub current_time: f64,
    pub is_finished: bool,
    pub hide_from_continue_listening: bool,
    pub ebook_location: Option<String>,
    pub ebook_progress: Option<f64>,
    pub last_update: i64,
    pub started_at: i64,
    pub finished_at: Option<i64>,
}
```

The patron's progress on one item.

**Fields**

- `id`: The progress id: the item id (one record per patron and item).
- `user_id`: The patron's id.
- `library_item_id`: The item id.
- `episode_id`: Always null.
- `media_item_id`: The media id: the item id.
- `media_item_type`: Always `book`.
- `duration`: The book's length in seconds, as last reported.
- `progress`: The share listened, from 0 to 1.
- `current_time`: Where the patron is, in seconds.
- `is_finished`: Whether the patron finished it.
- `hide_from_continue_listening`: Whether it is hidden from continue listening.
- `ebook_location`: Always null.
- `ebook_progress`: Always null.
- `last_update`: When the progress last changed, in epoch milliseconds.
- `started_at`: When the patron started, in epoch milliseconds.
- `finished_at`: When the patron finished, in epoch milliseconds.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Bookmark

```rust
pub struct Bookmark {
    pub library_item_id: String,
    pub title: String,
    pub time: f64,
    pub created_at: i64,
}
```

A bookmark.

**Fields**

- `library_item_id`: The item it is in.
- `title`: Its title.
- `time`: Where it is, in seconds.
- `created_at`: When it was made, in epoch milliseconds.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SessionDevice

```rust
pub struct SessionDevice {
    pub device_id: String,
    pub device_name: Option<String>,
    pub client_name: Option<String>,
}
```

The device a playback session was opened from.

**Fields**

- `device_id`: The device's id: the card's session id.
- `device_name`: The device name the app reported, if any.
- `client_name`: The app's name, if reported.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### PlaybackSession

```rust
pub struct PlaybackSession {
    pub id: String,
    pub user_id: String,
    pub library_id: String,
    pub library_item_id: String,
    pub book_id: String,
    pub episode_id: Option<String>,
    pub media_type: &'static str,
    pub media_metadata: BookMetadata,
    pub chapters: Vec<Chapter>,
    pub display_title: String,
    pub display_author: String,
    pub cover_path: Option<String>,
    pub duration: f64,
    pub play_method: u8,
    pub media_player: String,
    pub device_info: SessionDevice,
    pub server_version: &'static str,
    pub date: String,
    pub day_of_week: String,
    pub time_listening: f64,
    pub start_time: f64,
    pub current_time: f64,
    pub started_at: i64,
    pub updated_at: i64,
    pub audio_tracks: Vec<AudioTrack>,
    pub library_item: Option<LibraryItem>,
}
```

A playback session, as `POST /api/items/{id}/play` answers.

**Fields**

- `id`: The session id; apps send it back to sync and close.
- `user_id`: The patron's id.
- `library_id`: The library id.
- `library_item_id`: The item id.
- `book_id`: The media id: the item id.
- `episode_id`: Always null.
- `media_type`: Always [`MEDIA_TYPE_BOOK`](#media_type_book).
- `media_metadata`: The book's metadata.
- `chapters`: The book's chapters.
- `display_title`: The title apps show.
- `display_author`: The author apps show.
- `cover_path`: A cover path apps test for presence.
- `duration`: The book's length in seconds.
- `play_method`: How the app plays: 0, direct play of the stored files.
- `media_player`: The player the app reported.
- `device_info`: The device the session was opened from.
- `server_version`: [`crate::COMPATIBLE_SERVER_VERSION`](README.md#compatible_server_version).
- `date`: The day the session started, `YYYY-MM-DD` in UTC.
- `day_of_week`: The weekday the session started, in English.
- `time_listening`: Seconds listened in this session.
- `start_time`: Where the session started, in seconds.
- `current_time`: Where the patron is, in seconds.
- `started_at`: When the session started, in epoch milliseconds.
- `updated_at`: When it last synced, in epoch milliseconds.
- `audio_tracks`: The tracks to play, in order.
- `library_item`: The item, expanded.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SessionPage

```rust
pub struct SessionPage {
    pub total: u64,
    pub num_pages: u64,
    pub page: u64,
    pub items_per_page: u64,
    pub sessions: Vec<PlaybackSession>,
}
```

`GET /api/me/listening-sessions`.

**Fields**

- `total`: Sessions in all.
- `num_pages`: Pages in all.
- `page`: The page number, from 0.
- `items_per_page`: Sessions per page.
- `sessions`: The page's sessions, newest first.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### ListeningStats

```rust
pub struct ListeningStats {
    pub total_time: f64,
    pub items: BTreeMap<String, ItemListening>,
    pub days: BTreeMap<String, f64>,
    pub day_of_week: BTreeMap<String, f64>,
    pub today: f64,
    pub recent_sessions: Vec<PlaybackSession>,
}
```

`GET /api/me/listening-stats`.

**Fields**

- `total_time`: Seconds listened in all.
- `items`: Seconds listened per item, by item id.
- `days`: Seconds listened per day, by `YYYY-MM-DD`.
- `day_of_week`: Seconds listened per weekday, by English weekday name.
- `today`: Seconds listened today.
- `recent_sessions`: The most recent sessions.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### ItemListening

```rust
pub struct ItemListening {
    pub id: String,
    pub time_listening: f64,
    pub media_metadata: BookMetadata,
}
```

Seconds listened to one item.

**Fields**

- `id`: The item id.
- `time_listening`: Seconds listened.
- `media_metadata`: The item's metadata.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### LibraryItems

```rust
pub struct LibraryItems {
    pub library_items: Vec<LibraryItem>,
}
```

`{ "libraryItems": [...] }`: in-progress items and batch reads.

**Fields**

- `library_items`: The items.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Series

```rust
pub struct Series {
    pub id: String,
    pub name: String,
    pub name_ignore_prefix: String,
    pub description: Option<String>,
    pub added_at: i64,
    pub updated_at: i64,
    pub library_id: String,
    pub books: Option<Vec<LibraryItem>>,
    pub progress: Option<SeriesProgress>,
}
```

A series.

**Fields**

- `id`: The series id.
- `name`: Its name.
- `name_ignore_prefix`: The name for sorting.
- `description`: Its description: none.
- `added_at`: When the library first saw it, in epoch milliseconds.
- `updated_at`: When it last changed, in epoch milliseconds.
- `library_id`: The library it is listed in.
- `books`: Its items in series order, on lists.
- `progress`: The patron's progress through it, when asked for.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SeriesProgress

```rust
pub struct SeriesProgress {
    pub library_item_ids: Vec<String>,
    pub library_item_ids_finished: Vec<String>,
    pub is_finished: bool,
}
```

The patron's progress through a series.

**Fields**

- `library_item_ids`: The series' items, by id.
- `library_item_ids_finished`: Those the patron finished.
- `is_finished`: Whether the patron finished every one.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SeriesPage

```rust
pub struct SeriesPage {
    pub results: Vec<Series>,
    pub total: u64,
    pub limit: u64,
    pub page: u64,
    pub sort_by: Option<String>,
    pub sort_desc: bool,
    pub filter_by: Option<String>,
    pub minified: bool,
    pub include: String,
}
```

A page of series.

**Fields**

- `results`: The page's series.
- `total`: Series in all.
- `limit`: Series per page.
- `page`: The page number, from 0.
- `sort_by`: The sort key, echoed.
- `sort_desc`: Whether the order is reversed.
- `filter_by`: The filter, echoed.
- `minified`: Whether items are minified.
- `include`: The include list, echoed.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Author

```rust
pub struct Author {
    pub id: String,
    pub asin: Option<String>,
    pub name: String,
    pub last_first: String,
    pub description: Option<String>,
    pub image_path: Option<String>,
    pub library_id: String,
    pub added_at: i64,
    pub updated_at: i64,
    pub num_books: u64,
    pub library_items: Option<Vec<LibraryItem>>,
    pub series: Option<Vec<AuthorSeries>>,
}
```

An author.

**Fields**

- `id`: The contributor id.
- `asin`: Always null.
- `name`: The author's name.
- `last_first`: The name family name first.
- `description`: Always null.
- `image_path`: Always null: no author images.
- `library_id`: The library it is listed in.
- `added_at`: When the library first saw the author, in epoch milliseconds.
- `updated_at`: When the record last changed, in epoch milliseconds.
- `num_books`: Items by the author the patron can see.
- `library_items`: The author's items, when asked for.
- `series`: The author's series with their items, when asked for.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### AuthorSeries

```rust
pub struct AuthorSeries {
    pub id: String,
    pub name: String,
    pub items: Vec<LibraryItem>,
}
```

A series of an author's, with the author's items in it.

**Fields**

- `id`: The series id.
- `name`: The series name.
- `items`: The items, in series order.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Authors

```rust
pub struct Authors {
    pub authors: Vec<Author>,
}
```

`GET /api/libraries/{id}/authors` without paging.

**Fields**

- `authors`: The authors.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### AuthorPage

```rust
pub struct AuthorPage {
    pub results: Vec<Author>,
    pub total: u64,
    pub limit: u64,
    pub page: u64,
}
```

A page of authors.

**Fields**

- `results`: The page's authors.
- `total`: Authors in all.
- `limit`: Authors per page.
- `page`: The page number, from 0.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Narrator

```rust
pub struct Narrator {
    pub id: String,
    pub name: String,
    pub num_books: u64,
}
```

A narrator with how many items they read.

**Fields**

- `id`: The name in URL-safe base64, as apps filter by it.
- `name`: The narrator's name.
- `num_books`: Items they read that the patron can see.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Narrators

```rust
pub struct Narrators {
    pub narrators: Vec<Narrator>,
}
```

`GET /api/libraries/{id}/narrators`.

**Fields**

- `narrators`: The narrators.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Shelf

```rust
pub struct Shelf {
    pub id: &'static str,
    pub label: &'static str,
    pub label_string_key: &'static str,
    pub kind: &'static str,
    pub entities: ShelfEntities,
    pub total: u64,
}
```

A personalized shelf on the home screen.

**Fields**

- `id`: The shelf id, such as `continue-listening`.
- `label`: The shelf's English label.
- `label_string_key`: The key apps translate the label by.
- `kind`: What the shelf holds: `book`, `series` or `authors`.
- `entities`: The shelf's entries.
- `total`: Entries in all.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SearchResults

```rust
pub struct SearchResults {
    pub book: Vec<SearchBook>,
    pub podcast: Vec<Value>,
    pub narrators: Vec<Narrator>,
    pub tags: Vec<Value>,
    pub genres: Vec<Value>,
    pub series: Vec<SearchSeries>,
    pub authors: Vec<Author>,
}
```

`GET /api/libraries/{id}/search`.

**Fields**

- `book`: Matching items.
- `podcast`: Always empty.
- `narrators`: Matching narrators.
- `tags`: Always empty.
- `genres`: Always empty.
- `series`: Matching series.
- `authors`: Matching authors.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SearchBook

```rust
pub struct SearchBook {
    pub library_item: LibraryItem,
}
```

An item that matched a search.

**Fields**

- `library_item`: The item.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SearchSeries

```rust
pub struct SearchSeries {
    pub series: Series,
    pub books: Vec<LibraryItem>,
}
```

A series that matched a search.

**Fields**

- `series`: The series.
- `books`: Its items.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### LibraryStats

```rust
pub struct LibraryStats {
    pub total_items: u64,
    pub total_authors: u64,
    pub total_genres: u64,
    pub total_duration: f64,
    pub num_audio_tracks: u64,
    pub total_size: u64,
}
```

`GET /api/libraries/{id}/stats`.

**Fields**

- `total_items`: Items.
- `total_authors`: Authors.
- `total_genres`: Genres.
- `total_duration`: Seconds of audio in all.
- `num_audio_tracks`: Audio tracks in all.
- `total_size`: Bytes in all: not counted.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### EmptyPage

```rust
pub struct EmptyPage {
    pub results: Vec<Value>,
    pub total: u64,
    pub limit: u64,
    pub page: u64,
}
```

An empty page, for collections and playlists the library does not
have yet.

**Fields**

- `results`: Always empty.
- `total`: Always 0.
- `limit`: Items per page, echoed.
- `page`: The page number, echoed.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### Playlists

```rust
pub struct Playlists {
    pub playlists: Vec<Value>,
}
```

`{ "playlists": [] }`: the patron's playlists.

**Fields**

- `playlists`: Always empty.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SyncResult

```rust
pub struct SyncResult {
    pub id: String,
    pub success: bool,
    pub error: Option<String>,
}
```

One entry's outcome in a batch sync.

**Fields**

- `id`: The session or item id the entry named.
- `success`: Whether it was applied.
- `error`: Why not, when it was not.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SyncResults

```rust
pub struct SyncResults {
    pub results: Vec<SyncResult>,
}
```

`POST /api/session/local-all`.

**Fields**

- `results`: One outcome per entry, in the order sent.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### ProgressEvent

```rust
pub struct ProgressEvent {
    pub id: String,
    pub session_id: Option<String>,
    pub device_description: String,
    pub data: MediaProgress,
}
```

`user_item_progress_updated` on the socket: progress changed on another
device.

**Fields**

- `id`: The progress id.
- `session_id`: The session that changed it, if one did.
- `device_description`: The device that changed it.
- `data`: The progress.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SocketInit

```rust
pub struct SocketInit {
    pub user_id: String,
    pub username: String,
}
```

`init` on the socket: the socket is signed in.

**Fields**

- `user_id`: The patron's id.
- `username`: The patron's sign-in name.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### SocketAuthFailed

```rust
pub struct SocketAuthFailed {
    pub message: &'static str,
}
```

`auth_failed` on the socket.

**Fields**

- `message`: Why, in English.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

### MetaTags

```rust
pub struct MetaTags {}
```

The tags read from an audio file, as apps read them: always empty until
the stored media probe reads tags, so it serializes as `{}`.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Default`, `Serialize`, `ToSchema`

## Enums

### ShelfEntities

```rust
pub enum ShelfEntities {
    Books(Vec<LibraryItem>),
    Series(Vec<Series>),
    Authors(Vec<Author>),
}
```

A shelf's entries.

**Variants**

- `Books`: Items.
- `Series`: Series.
- `Authors`: Authors.

Implements: `Clone`, `ComposeSchema`, `Debug`, `Serialize`, `ToSchema`

## Functions

### title_ignore_prefix

```rust
pub fn title_ignore_prefix(title: &str) -> String
```

The name an item's title sorts by: a leading `The `, `A ` or `An `
moved to the end (`Hobbit, The`), as apps show it in sorted lists.

### last_first

```rust
pub fn last_first(name: &str) -> String
```

A name family name first (`Le Guin, Ursula K.`): the last word moved to
the front. A one-word name is returned as it is.

### plain_text

```rust
pub fn plain_text(html: &str) -> String
```

Text with markup removed: everything between `<` and `>` dropped and
the common entities decoded.

## Constants

### MEDIA_TYPE_BOOK

```rust
pub const MEDIA_TYPE_BOOK: &str = "book";
```

The media type of every library and item the API lists.

### STATUS_APP

```rust
pub const STATUS_APP: &str = "audiobookshelf";
```

The `app` the status answer names. Apps check it before they offer a
sign-in, so it is the value they expect.

### AUTH_METHODS

```rust
pub const AUTH_METHODS: [&str; 2] = _;
```

The sign-in methods an app may offer: a player password, and the
browser sign-in through the library's own sign-in page.
