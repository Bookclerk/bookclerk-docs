# abi

## Type Aliases

### Abridgement

```ts
type Abridgement = typeof ABRIDGEMENTS[number];
```

Union of `ABRIDGEMENTS` wire names.

***

### AiTask

```ts
type AiTask = typeof AI_TASKS[number];
```

Union of `AI_TASKS` wire names.

***

### AiTextRole

```ts
type AiTextRole = typeof AI_TEXT_ROLES[number];
```

Union of `AI_TEXT_ROLES` wire names.

***

### CatalogField

```ts
type CatalogField = typeof CATALOG_FIELDS[number];
```

Union of `CATALOG_FIELDS` wire names.

***

### CatalogSort

```ts
type CatalogSort = typeof CATALOG_SORTS[number];
```

Union of `CATALOG_SORTS` wire names.

***

### ChapterKind

```ts
type ChapterKind = typeof CHAPTER_KINDS[number];
```

Union of `CHAPTER_KINDS` wire names.

***

### CliArgKind

```ts
type CliArgKind = typeof CLI_ARG_KINDS[number];
```

Union of `CLI_ARG_KINDS` wire names.

***

### CreditLevel

```ts
type CreditLevel = typeof CREDIT_LEVELS[number];
```

Union of `CREDIT_LEVELS` wire names.

***

### CreditRole

```ts
type CreditRole = typeof CREDIT_ROLES[number];
```

Union of `CREDIT_ROLES` wire names.

***

### DbResultSelection

```ts
type DbResultSelection = typeof DB_RESULT_SELECTIONS[number];
```

Union of `DB_RESULT_SELECTIONS` wire names.

***

### DbStatementKind

```ts
type DbStatementKind = typeof DB_STATEMENT_KINDS[number];
```

Union of `DB_STATEMENT_KINDS` wire names.

***

### DbType

```ts
type DbType = typeof DB_TYPES[number];
```

Union of `DB_TYPES` wire names.

***

### Entrypoint

```ts
type Entrypoint = typeof ENTRYPOINTS[number];
```

Union of `ENTRYPOINTS` wire names.

***

### Explicitness

```ts
type Explicitness = typeof EXPLICITNESSS[number];
```

Union of `EXPLICITNESSS` wire names.

***

### ExpressionKind

```ts
type ExpressionKind = typeof EXPRESSION_KINDS[number];
```

Union of `EXPRESSION_KINDS` wire names.

***

### FetchedFileRole

```ts
type FetchedFileRole = typeof FETCHED_FILE_ROLES[number];
```

Union of `FETCHED_FILE_ROLES` wire names.

***

### IdentifierScheme

```ts
type IdentifierScheme = typeof IDENTIFIER_SCHEMES[number];
```

Union of `IDENTIFIER_SCHEMES` wire names.

***

### IntegerArithKind

```ts
type IntegerArithKind = typeof INTEGER_ARITH_KINDS[number];
```

Union of `INTEGER_ARITH_KINDS` wire names.

***

### IsolationReq

```ts
type IsolationReq = typeof ISOLATION_REQS[number];
```

Union of `ISOLATION_REQS` wire names.

***

### MediaKind

```ts
type MediaKind = typeof MEDIA_KINDS[number];
```

Union of `MEDIA_KINDS` wire names.

***

### MetadataField

```ts
type MetadataField = typeof METADATA_FIELDS[number];
```

Union of `METADATA_FIELDS` wire names.

***

### NotificationAlertAction

```ts
type NotificationAlertAction = typeof NOTIFICATION_ALERT_ACTIONS[number];
```

Union of `NOTIFICATION_ALERT_ACTIONS` wire names.

***

### NotificationAudience

```ts
type NotificationAudience = typeof NOTIFICATION_AUDIENCES[number];
```

Union of `NOTIFICATION_AUDIENCES` wire names.

***

### NotificationSeverity

```ts
type NotificationSeverity = typeof NOTIFICATION_SEVERITYS[number];
```

Union of `NOTIFICATION_SEVERITYS` wire names.

***

### NotifyDestinationKind

```ts
type NotifyDestinationKind = typeof NOTIFY_DESTINATION_KINDS[number];
```

Union of `NOTIFY_DESTINATION_KINDS` wire names.

***

### ObjectChangeKind

```ts
type ObjectChangeKind = typeof OBJECT_CHANGE_KINDS[number];
```

Union of `OBJECT_CHANGE_KINDS` wire names.

***

### OfferedAacProfile

```ts
type OfferedAacProfile = typeof OFFERED_AAC_PROFILES[number];
```

Union of `OFFERED_AAC_PROFILES` wire names.

***

### OfferedCodec

```ts
type OfferedCodec = typeof OFFERED_CODECS[number];
```

Union of `OFFERED_CODECS` wire names.

***

### PluginErrorCode

```ts
type PluginErrorCode = typeof PLUGIN_ERROR_CODES[number];
```

Union of `PLUGIN_ERROR_CODES` wire names.

***

### PortalAuthMode

```ts
type PortalAuthMode = typeof PORTAL_AUTH_MODES[number];
```

Union of `PORTAL_AUTH_MODES` wire names.

***

### PriceLookup

```ts
type PriceLookup = typeof PRICE_LOOKUPS[number];
```

Union of `PRICE_LOOKUPS` wire names.

***

### ResolvedSqlType

```ts
type ResolvedSqlType = typeof RESOLVED_SQL_TYPES[number];
```

Union of `RESOLVED_SQL_TYPES` wire names.

***

### SettingGrantState

```ts
type SettingGrantState = typeof SETTING_GRANT_STATES[number];
```

Union of `SETTING_GRANT_STATES` wire names.

***

### SettingNetworkKind

```ts
type SettingNetworkKind = typeof SETTING_NETWORK_KINDS[number];
```

Union of `SETTING_NETWORK_KINDS` wire names.

***

### WorkKind

```ts
type WorkKind = typeof WORK_KINDS[number];
```

Union of `WORK_KINDS` wire names.

## Variables

### ABRIDGEMENTS

```ts
const ABRIDGEMENTS: readonly ["unknown", "unabridged", "abridged"];
```

Whether an edition is abridged.

Ordinal-ordered `Abridgement` wire names (index = Cap'n Proto ordinal).

***

### AI\_TASKS

```ts
const AI_TASKS: readonly ["textEmbeddings"];
```

What a model does. A model has exactly one task, and the task fixes the
shape of `AiRunParams.input` and of the reply.

Ordinal-ordered `AiTask` wire names (index = Cap'n Proto ordinal).

***

### AI\_TEXT\_ROLES

```ts
const AI_TEXT_ROLES: readonly ["document", "query"];
```

What a text is to an embedding model. Some models embed a search query
differently from the documents it is matched against.

Ordinal-ordered `AiTextRole` wire names (index = Cap'n Proto ordinal).

***

### CATALOG\_FIELDS

```ts
const CATALOG_FIELDS: readonly ["any", "author", "narrator", "series", "genre"];
```

Catalog search facet restricting which field the query matches.

Ordinal-ordered `CatalogField` wire names (index = Cap'n Proto ordinal).

***

### CATALOG\_SORTS

```ts
const CATALOG_SORTS: readonly ["relevance", "popularity", "rating", "title", "author"];
```

Catalog search ordering.

Ordinal-ordered `CatalogSort` wire names (index = Cap'n Proto ordinal).

***

### CHAPTER\_KINDS

```ts
const CHAPTER_KINDS: readonly ["unknown", "chapters", "tracks"];
```

What a storefront's chapter markers mark (`PlainFetch.chapterKind`).

Ordinal-ordered `ChapterKind` wire names (index = Cap'n Proto ordinal).

***

### CLI\_ARG\_KINDS

```ts
const CLI_ARG_KINDS: readonly ["string", "bool", "int", "path"];
```

Value kind for a `CliArgSpec`.

Ordinal-ordered `CliArgKind` wire names (index = Cap'n Proto ordinal).

***

### CREDIT\_LEVELS

```ts
const CREDIT_LEVELS: readonly ["roleDefault", "work", "expression", "edition"];
```

Which level a credit attaches to. Ordinal 0 is the role's usual level, so
a guest sets this only when a credit departs from it.

Ordinal-ordered `CreditLevel` wire names (index = Cap'n Proto ordinal).

***

### CREDIT\_ROLES

```ts
const CREDIT_ROLES: readonly ["author", "narrator", "translator", "editor", "illustrator", "publisher", "host", "cast", "adaptor"];
```

A contributor's responsibility for a work, expression or edition.

Ordinal-ordered `CreditRole` wire names (index = Cap'n Proto ordinal).

***

### DB\_RESULT\_SELECTIONS

```ts
const DB_RESULT_SELECTIONS: readonly ["discard", "affectedRows", "rows"];
```

Which part of a statement's outcome the caller wants back.

Ordinal-ordered `DbResultSelection` wire names (index = Cap'n Proto ordinal).

***

### DB\_STATEMENT\_KINDS

```ts
const DB_STATEMENT_KINDS: readonly ["execute", "select", "returning"];
```

How the host classifies a statement for result handling.

Ordinal-ordered `DbStatementKind` wire names (index = Cap'n Proto ordinal).

***

### DB\_TYPES

```ts
const DB_TYPES: readonly ["unspecified", "bool", "int64", "float64", "text", "bytes"];
```

Universal database cell/parameter domain. Engine-native arrays, enums,
unsigned integers, and JSON text sentinels are not baseline ABI values.

Ordinal-ordered `DbType` wire names (index = Cap'n Proto ordinal).

***

### ENTRYPOINTS

```ts
const ENTRYPOINTS: readonly ["storefront", "storage", "databaseAdapter", "remoteLibrary", "cli", "oidc", "metadata", "covers", "metadataFiles", "notifier", "ingress", "ai"];
```

Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint
analogue). Each value is a capability the host calls over RPC; triggers on
the default entrypoint (`event`, `job`) are declared separately.

Ordinal-ordered `Entrypoint` wire names (index = Cap'n Proto ordinal).

***

### EXPLICITNESSS

```ts
const EXPLICITNESSS: readonly ["unknown", "clean", "explicit"];
```

Whether an edition is marked as explicit (adult) content. A patron scope
can hide explicit editions; `unknown` and `clean` editions stay visible.

Ordinal-ordered `Explicitness` wire names (index = Cap'n Proto ordinal).

***

### EXPRESSION\_KINDS

```ts
const EXPRESSION_KINDS: readonly ["unspecified", "text", "narration", "dramatisation", "graphic", "originalAudio"];
```

How a work is realised (FRBR expression): the kind of content you get,
independent of the carrier. A translation is not a kind: the expression's
language and its translator credits say it. `unspecified` is refused.

Ordinal-ordered `ExpressionKind` wire names (index = Cap'n Proto ordinal).

***

### FEATURE\_SCALAR\_LIMITS

```ts
const FEATURE_SCALAR_LIMITS: "rpc.scalarLimits";
```

Guest honors scalar / stream-window / list-page caps (`rpc.scalarLimits`).

***

### FEATURE\_STORAGE\_CHANGES

```ts
const FEATURE_STORAGE_CHANGES: "storage.changes";
```

Guest implements `Destination.changes` (`storage.changes`).

***

### FEATURE\_STORAGE\_CHECKED\_WRITE

```ts
const FEATURE_STORAGE_CHECKED_WRITE: "storage.checkedWrite";
```

Guest enforces `WriteOptions.ifMatch` / `ifNoneMatch` by writing, then
checking and withdrawing its write when another got in first
(`storage.checkedWrite`); a reader may briefly see the losing write.

***

### FEATURE\_STORAGE\_CONDITIONAL\_WRITE

```ts
const FEATURE_STORAGE_CONDITIONAL_WRITE: "storage.conditionalWrite";
```

Guest enforces `WriteOptions.ifMatch` / `ifNoneMatch` (`storage.conditionalWrite`).

***

### FEATURE\_STORAGE\_COPY

```ts
const FEATURE_STORAGE_COPY: "storage.copy";
```

Guest implements server-side `Destination.copy` (`storage.copy`).

***

### FEATURE\_STORAGE\_DELETE\_MANY

```ts
const FEATURE_STORAGE_DELETE_MANY: "storage.deleteMany";
```

Guest implements `Destination.deleteMany` (`storage.deleteMany`).

***

### FEATURE\_STORAGE\_HEAD\_MANY

```ts
const FEATURE_STORAGE_HEAD_MANY: "storage.headMany";
```

Guest implements `Destination.headMany` (`storage.headMany`).

***

### FEATURE\_STORAGE\_MOVE

```ts
const FEATURE_STORAGE_MOVE: "storage.move";
```

Guest implements `Destination.move` (`storage.move`).

***

### FEATURE\_STORAGE\_MOVE\_MANY

```ts
const FEATURE_STORAGE_MOVE_MANY: "storage.moveMany";
```

Guest implements `Destination.moveMany` (`storage.moveMany`).

***

### FEATURE\_STORAGE\_TRANSLATE\_KEYS

```ts
const FEATURE_STORAGE_TRANSLATE_KEYS: "storage.translateKeys";
```

Guest implements `Destination.translateKeys` (`storage.translateKeys`).

***

### FEATURE\_STREAMS

```ts
const FEATURE_STREAMS: "rpc.streams";
```

Media moves through transferred `ByteRange` / `ByteSource` streams (`rpc.streams`).

***

### FETCHED\_FILE\_ROLES

```ts
const FETCHED_FILE_ROLES: readonly ["unspecified", "part", "whole", "cover"];
```

What one fetched file is to the edition it belongs to.

Ordinal-ordered `FetchedFileRole` wire names (index = Cap'n Proto ordinal).

***

### IDENTIFIER\_SCHEMES

```ts
const IDENTIFIER_SCHEMES: readonly ["isbn", "asin", "openLibraryEdition", "openLibraryWork"];
```

Scheme of an external identifier. The scheme fixes whether the id names a
work or an edition, and whether it is strong (unique) or weak (shared).

Ordinal-ordered `IdentifierScheme` wire names (index = Cap'n Proto ordinal).

***

### INTEGER\_ARITH\_KINDS

```ts
const INTEGER_ARITH_KINDS: readonly ["add", "sub", "mul", "abs"];
```

INTEGER `+` `-` `*` `abs` site lowered to overflow -> NULL.

Ordinal-ordered `IntegerArithKind` wire names (index = Cap'n Proto ordinal).

***

### ISOLATION\_REQS

```ts
const ISOLATION_REQS: readonly ["atomicBatch", "nestedSavepoint", "consistentSnapshot"];
```

Host → adapter execute. GuestDatabase stays ExecuteRequest-only.

Ordinal-ordered `IsolationReq` wire names (index = Cap'n Proto ordinal).

***

### MAX\_AI\_EMBED\_DIMENSIONS

```ts
const MAX_AI_EMBED_DIMENSIONS: 4096;
```

Most numbers in one embedding vector.

***

### MAX\_AI\_EMBED\_TEXT\_BYTES

```ts
const MAX_AI_EMBED_TEXT_BYTES: 16384;
```

Longest text in an `AiTextEmbeddingsInput`, in UTF-8 bytes.

***

### MAX\_AI\_EMBED\_TEXTS

```ts
const MAX_AI_EMBED_TEXTS: 64;
```

Most texts in one `AiTextEmbeddingsInput`.

***

### MAX\_AI\_LANGUAGES

```ts
const MAX_AI_LANGUAGES: 64;
```

Most language tags on one `AiModel`.

***

### MAX\_AI\_MODEL\_NAME\_BYTES

```ts
const MAX_AI_MODEL_NAME_BYTES: 192;
```

Longest `AiModel.name`, in UTF-8 bytes (`@<plugin id>/<model id>`).

***

### MAX\_AI\_MODELS

```ts
const MAX_AI_MODELS: 64;
```

Most models one `AiModels.models` reply lists.

***

### MAX\_BATCH\_KEYS

```ts
const MAX_BATCH_KEYS: 256;
```

Maximum keys or key pairs in one `Destination.moveMany`, `deleteMany` or
`headMany` call, and maximum outcomes in one reply.

***

### MAX\_CHANGES\_CURSOR\_BYTES

```ts
const MAX_CHANGES_CURSOR_BYTES: 4096;
```

Maximum bytes of a `Destination.changes` cursor.

***

### MAX\_CHANGES\_PAGE

```ts
const MAX_CHANGES_PAGE: 256;
```

Maximum changes in one `Destination.changes` page.

***

### MAX\_CHECKPOINT\_BYTES

```ts
const MAX_CHECKPOINT_BYTES: 65536;
```

Maximum job / event checkpoint payload size (bytes).

***

### MAX\_CONFIG\_PAYLOAD\_BYTES

```ts
const MAX_CONFIG_PAYLOAD_BYTES: 65536;
```

Maximum granted config payload size (bytes).

***

### MAX\_COVER\_BYTES

```ts
const MAX_COVER_BYTES: 10485760;
```

Largest image body `CoverFetcher.fetchCovers` may return for one URL.

***

### MAX\_COVER\_HEADER\_BYTES

```ts
const MAX_COVER_HEADER_BYTES: 1024;
```

Longest cache header value, in bytes, `CoverFetcher.fetchCovers` carries in
either direction (a condition the host sends, a header the server sent).
The host ignores a longer header the fetcher returns.

***

### MAX\_COVER\_URLS

```ts
const MAX_COVER_URLS: 32;
```

Most URLs in one `CoverFetcher.fetchCovers` call.

***

### MAX\_CRON\_BYTES

```ts
const MAX_CRON_BYTES: 128;
```

Maximum `[triggers] crons` source length (bytes), as in `ScheduledController.cron`.

***

### MAX\_EVENT\_CURSOR\_BYTES

```ts
const MAX_EVENT_CURSOR_BYTES: 512;
```

Maximum bytes of an `EventReadRequest.after` / `EventReadPage.cursor`.

***

### MAX\_EVENT\_PAYLOAD\_BYTES

```ts
const MAX_EVENT_PAYLOAD_BYTES: 65536;
```

Maximum decoded size of a domain-event scalar payload (not a stream).

***

### MAX\_IDENTIFIER\_BYTES

```ts
const MAX_IDENTIFIER_BYTES: 64;
```

Maximum plugin / account identifier length (bytes).

***

### MAX\_INGRESS\_ORIGIN\_BYTES

```ts
const MAX_INGRESS_ORIGIN_BYTES: 253;
```

Longest `IngressStarted.origin`, in bytes.

***

### MAX\_KV\_KEY\_BYTES

```ts
const MAX_KV_KEY_BYTES: 512;
```

Longest `KvNamespace` key in UTF-8 bytes.

***

### MAX\_KV\_METADATA\_BYTES

```ts
const MAX_KV_METADATA_BYTES: 1024;
```

Largest `KvNamespace` metadata JSON text in bytes.

***

### MAX\_KV\_VALUE\_BYTES

```ts
const MAX_KV_VALUE_BYTES: 1048576;
```

Largest `KvNamespace` value in bytes; a `[[kv_namespaces]]` row may lower
its own cap but never raise it past this.

***

### MAX\_LIST\_PAGE

```ts
const MAX_LIST_PAGE: 256;
```

Maximum objects in one `Destination.list` page.

***

### MAX\_METADATA\_AUDIO\_FILES

```ts
const MAX_METADATA_AUDIO_FILES: 4096;
```

Most entries in `MetadataFileRenderParams.audioFiles`.

***

### MAX\_METADATA\_FILE\_BYTES

```ts
const MAX_METADATA_FILE_BYTES: 262144;
```

Largest metadata file `MetadataFiles.parse` reads and `MetadataFiles.render`
returns. The host skips a larger sidecar before any plugin sees it.

***

### MAX\_METADATA\_FILE\_CHAPTERS

```ts
const MAX_METADATA_FILE_CHAPTERS: 2000;
```

Most chapters in a `MetadataFileRecord`.

***

### MAX\_METADATA\_FILE\_DESCRIPTION\_BYTES

```ts
const MAX_METADATA_FILE_DESCRIPTION_BYTES: 65536;
```

Longest description a `MetadataFileRecord` carries, in UTF-8 bytes.

***

### MAX\_METADATA\_FILE\_LIST\_ENTRIES

```ts
const MAX_METADATA_FILE_LIST_ENTRIES: 256;
```

Most entries in each list of a `MetadataFileRecord` other than chapters
(credits, genres, tags, identifiers, series).

***

### MAX\_METADATA\_FILE\_TEXT\_BYTES

```ts
const MAX_METADATA_FILE_TEXT_BYTES: 1024;
```

Longest other text value in a `MetadataFileRecord` (a title, a name, a
genre, a chapter title), in UTF-8 bytes.

***

### MAX\_NETWORK\_GRANT\_ENTRIES

```ts
const MAX_NETWORK_GRANT_ENTRIES: 256;
```

Most entries in each list of a `NetworkGrant` (fetch hosts, TCP
destinations, network settings).

***

### MAX\_NOTIFICATION\_FIELD\_BYTES

```ts
const MAX_NOTIFICATION_FIELD_BYTES: 1024;
```

Longest `NotificationField.value`, link, group key or alert key, in UTF-8
bytes.

***

### MAX\_NOTIFICATION\_FIELDS

```ts
const MAX_NOTIFICATION_FIELDS: 25;
```

Most `NotificationField`s on one message or raise.

***

### MAX\_NOTIFICATION\_HTML\_BYTES

```ts
const MAX_NOTIFICATION_HTML_BYTES: 65536;
```

Longest `NotificationMessage.html`, in UTF-8 bytes.

***

### MAX\_NOTIFICATION\_TEXT\_BYTES

```ts
const MAX_NOTIFICATION_TEXT_BYTES: 16384;
```

Longest `NotificationMessage.text` or `markdown`, in UTF-8 bytes.

***

### MAX\_NOTIFICATION\_TITLE\_BYTES

```ts
const MAX_NOTIFICATION_TITLE_BYTES: 256;
```

Longest `NotificationMessage.title` and `NotificationField` name, in UTF-8
bytes.

***

### MAX\_NOTIFIER\_ADDRESS\_BYTES

```ts
const MAX_NOTIFIER_ADDRESS_BYTES: 4096;
```

Longest destination address document (`addressJson`), in UTF-8 bytes.

***

### MAX\_ORIGIN\_HEADER\_BYTES

```ts
const MAX_ORIGIN_HEADER_BYTES: 65536;
```

Largest total UTF-8 bytes of header names and values on one `Origin.fetch`
request, and separately on its response.

***

### MAX\_ORIGIN\_HEADERS

```ts
const MAX_ORIGIN_HEADERS: 128;
```

Most header fields on one `Origin.fetch` request, and separately on its
response.

***

### MAX\_ORIGIN\_TARGET\_BYTES

```ts
const MAX_ORIGIN_TARGET_BYTES: 8192;
```

Longest `OriginRequest.target` (path and query), in bytes.

***

### MAX\_PLUGIN\_MIGRATION\_OPS

```ts
const MAX_PLUGIN_MIGRATION_OPS: 256;
```

Maximum already-separated operations in one plugin-owned migration.

***

### MAX\_PLUGIN\_MIGRATION\_REGISTRATION\_BYTES

```ts
const MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES: 262144;
```

Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one `databaseMigrations` registration.

***

### MAX\_PLUGIN\_MIGRATION\_TOTAL\_OPS

```ts
const MAX_PLUGIN_MIGRATION_TOTAL_OPS: 2048;
```

Maximum already-separated operations across one `databaseMigrations` registration.

***

### MAX\_REFRESHED\_CREDENTIAL\_BYTES

```ts
const MAX_REFRESHED_CREDENTIAL_BYTES: 65536;
```

Maximum bytes of refreshed credentials a storefront hands back
(`ScanPage.refreshedCredentials`, `PlainFetch.refreshedCredentials`,
`QualityOffersResult.refreshedCredentials`).

***

### MAX\_SCALAR\_BYTES

```ts
const MAX_SCALAR_BYTES: 262144;
```

Maximum decoded size of an ordinary RPC scalar value (not a stream window).

***

### MAX\_SCAN\_CURSOR\_BYTES

```ts
const MAX_SCAN_CURSOR_BYTES: 4096;
```

Maximum bytes of a scan cursor or sync token.

***

### MAX\_SCAN\_PAGE\_BOOKS

```ts
const MAX_SCAN_PAGE_BOOKS: 500;
```

Maximum titles, and separately removed product ids, in one `ScanPage`.

***

### MAX\_SOCKET\_ATTACHMENT\_BYTES

```ts
const MAX_SOCKET_ATTACHMENT_BYTES: 2048;
```

Largest per-socket attachment the host keeps for a sleeping guest.

***

### MAX\_SOCKET\_AUTO\_RESPONSE\_BYTES

```ts
const MAX_SOCKET_AUTO_RESPONSE_BYTES: 256;
```

Largest heartbeat request or response frame, in bytes.

***

### MAX\_SOCKET\_AUTO\_RESPONSES

```ts
const MAX_SOCKET_AUTO_RESPONSES: 4;
```

Most heartbeat pairs one `Sockets.connect` may declare.

***

### MAX\_SOCKET\_FRAME\_BYTES

```ts
const MAX_SOCKET_FRAME_BYTES: 262144;
```

Largest WebSocket message, text or binary, that `Sockets.send` accepts or
`SocketHandler.message` delivers.

***

### MAX\_SOCKET\_HEADERS

```ts
const MAX_SOCKET_HEADERS: 16;
```

Most request headers, and separately subprotocols, on one `Sockets.connect`.

***

### MAX\_SOCKET\_URL\_BYTES

```ts
const MAX_SOCKET_URL_BYTES: 2048;
```

Longest `SocketConnectRequest.url`, in bytes.

***

### MAX\_SOCKETS\_PER\_INSTANCE

```ts
const MAX_SOCKETS_PER_INSTANCE: 8;
```

Most host-brokered sockets one plugin instance may hold open at once.

***

### MAX\_STREAM\_WINDOW\_BYTES

```ts
const MAX_STREAM_WINDOW_BYTES: 1048576;
```

Maximum bytes returned by one `ByteSource.pull` (flow-control window).

***

### MAX\_WEBHOOK\_BODY\_BYTES

```ts
const MAX_WEBHOOK_BODY_BYTES: 1048576;
```

Largest stored webhook request body, in bytes (1 MiB); a `[[webhooks]]`
row may lower its own cap but never raise it past this.

***

### MAX\_WEBHOOK\_HEADER\_VALUE\_BYTES

```ts
const MAX_WEBHOOK_HEADER_VALUE_BYTES: 8192;
```

Longest kept request header value, in bytes.

***

### MAX\_WEBHOOK\_HEADERS

```ts
const MAX_WEBHOOK_HEADERS: 16;
```

Most request headers a `[[webhooks]]` row keeps; a delivery also carries
`content-type`, so it holds at most one more.

***

### MAX\_WEBHOOK\_LEASE\_ID\_BYTES

```ts
const MAX_WEBHOOK_LEASE_ID_BYTES: 128;
```

Longest `WebhookDelivery.leaseId`, in bytes.

***

### MAX\_WEBHOOK\_NAME\_BYTES

```ts
const MAX_WEBHOOK_NAME_BYTES: 32;
```

Longest `[[webhooks]]` name, in bytes.

***

### MAX\_WEBHOOK\_PULL\_BATCH

```ts
const MAX_WEBHOOK_PULL_BATCH: 100;
```

Most deliveries one `Webhooks.pull` returns.

***

### MAX\_WEBHOOK\_VISIBILITY\_MS

```ts
const MAX_WEBHOOK_VISIBILITY_MS: 43200000;
```

Longest `Webhooks.pull` visibility timeout, in milliseconds (12 hours).

***

### MEDIA\_KINDS

```ts
const MEDIA_KINDS: readonly ["unspecified", "audio", "ebook", "print"];
```

Carrier of an edition (FRBR manifestation): what form you receive.
Whether the host acquires an edition depends on the storefront's
`MediaSupport.fetches` and what the host supports, not on this value
alone. `unspecified` is refused.

Ordinal-ordered `MediaKind` wire names (index = Cap'n Proto ordinal).

***

### METADATA\_FIELDS

```ts
const METADATA_FIELDS: readonly ["title", "subtitle", "description", "publisher", "publishedAt", "language", "genres", "tags", "abridgement", "expressionKind", "identifiers", "credits", "series", "chapters", "duration", "explicitness"];
```

A field of a `MetadataFileRecord`. Dialects declare in the manifest which
fields their format carries; a parsed record lists the ones its file
states, and a render request lists the ones to write.

Ordinal-ordered `MetadataField` wire names (index = Cap'n Proto ordinal).

***

### NOTIFICATION\_ALERT\_ACTIONS

```ts
const NOTIFICATION_ALERT_ACTIONS: readonly ["none", "trigger", "resolve"];
```

Lifecycle step of an alert, for services such as PagerDuty that open and
close incidents.

Ordinal-ordered `NotificationAlertAction` wire names (index = Cap'n Proto ordinal).

***

### NOTIFICATION\_AUDIENCES

```ts
const NOTIFICATION_AUDIENCES: readonly ["operational", "patron", "account"];
```

Who a notification type is for. The account audience is reserved to the
host; a plugin may declare only operational and patron types.

Ordinal-ordered `NotificationAudience` wire names (index = Cap'n Proto ordinal).

***

### NOTIFICATION\_SEVERITYS

```ts
const NOTIFICATION_SEVERITYS: readonly ["info", "warning", "critical"];
```

How urgent a notification is. Routing rules carry a minimum severity.

Ordinal-ordered `NotificationSeverity` wire names (index = Cap'n Proto ordinal).

***

### NOTIFY\_DESTINATION\_KINDS

```ts
const NOTIFY_DESTINATION_KINDS: readonly ["channel", "personal"];
```

Kind of destination a `Notifier` sends to.

Ordinal-ordered `NotifyDestinationKind` wire names (index = Cap'n Proto ordinal).

***

### OBJECT\_CHANGE\_KINDS

```ts
const OBJECT_CHANGE_KINDS: readonly ["changed", "upserted", "removed"];
```

What happened to an object in a change feed. Unknown codes are kept and
treated as `changed`.

Ordinal-ordered `ObjectChangeKind` wire names (index = Cap'n Proto ordinal).

***

### OFFERED\_AAC\_PROFILES

```ts
const OFFERED_AAC_PROFILES: readonly ["unknown", "lc", "he", "heV2", "xhe"];
```

AAC profile of an offered AAC stream (`QualityOffer.aacProfile`).

Ordinal-ordered `OfferedAacProfile` wire names (index = Cap'n Proto ordinal).

***

### OFFERED\_CODECS

```ts
const OFFERED_CODECS: readonly ["unknown", "aac", "alac", "flac", "mp3", "opus", "vorbis", "eac3", "ac4"];
```

Audio codec of a stream a store offers (`QualityOffer.codec`). Spelled as
the host's probed codecs are, so an offer compares with a held file.

Ordinal-ordered `OfferedCodec` wire names (index = Cap'n Proto ordinal).

***

### PLUGIN\_ERROR\_CODES

```ts
const PLUGIN_ERROR_CODES: readonly ["invalid_params", "unauthorized", "forbidden", "not_found", "unavailable", "unsupported", "internal", "payload_too_large", "deadline_exceeded", "invalid_cursor", "cancelled", "conflict", "limit_exceeded", "rate_limited", "network_not_granted"];
```

Stable `PluginError.code` strings. Unknown future codes are forwarded
as-is; SDKs surface them as a local `unknown` while keeping the raw wire
code.

Wire strings of `PluginErrorCode` in ordinal order (snake_case `Text` codes).

***

### PORTAL\_AUTH\_MODES

```ts
const PORTAL_AUTH_MODES: readonly ["unspecified", "password", "oauth"];
```

Portal Accounts connect mode for storefronts.

Ordinal-ordered `PortalAuthMode` wire names (index = Cap'n Proto ordinal).

***

### PRICE\_LOOKUPS

```ts
const PRICE_LOOKUPS: readonly ["unspecified", "product", "isbn", "asin", "title"];
```

How a purchase-hint query names the title to price.

Ordinal-ordered `PriceLookup` wire names (index = Cap'n Proto ordinal).

***

### PRODUCT\_API\_VERSION

```ts
const PRODUCT_API_VERSION: 3;
```

Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`).

***

### RESOLVED\_SQL\_TYPES

```ts
const RESOLVED_SQL_TYPES: readonly ["integer", "real", "text", "blob", "boolean", "null"];
```

Resolved BookclerkSQL column type after type checking.

Ordinal-ordered `ResolvedSqlType` wire names (index = Cap'n Proto ordinal).

***

### SETTING\_GRANT\_STATES

```ts
const SETTING_GRANT_STATES: readonly ["approved", "pending", "unset", "invalid"];
```

Where a network setting's destination stands.

Ordinal-ordered `SettingGrantState` wire names (index = Cap'n Proto ordinal).

***

### SETTING\_NETWORK\_KINDS

```ts
const SETTING_NETWORK_KINDS: readonly ["fetch", "tcp"];
```

How a setting's destination is reached.

Ordinal-ordered `SettingNetworkKind` wire names (index = Cap'n Proto ordinal).

***

### WORK\_KINDS

```ts
const WORK_KINDS: readonly ["unspecified", "book", "podcast"];
```

What family of creative work a record describes (FRBR work). Every record
names one: `unspecified` is refused by the host, so a guest that forgets
the field fails instead of silently describing a book. Whether a podcast
work is a show or an episode is structural, not a kind: an episode names
its show through `parentProductId`, a show names no parent.

Ordinal-ordered `WorkKind` wire names (index = Cap'n Proto ordinal).
