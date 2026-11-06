# abi

## Type Aliases

### Abridgement

```ts
type Abridgement = typeof ABRIDGEMENTS[number];
```

Union of `ABRIDGEMENTS` wire names.

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

### ExpressionKind

```ts
type ExpressionKind = typeof EXPRESSION_KINDS[number];
```

Union of `EXPRESSION_KINDS` wire names.

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

### ResolvedSqlType

```ts
type ResolvedSqlType = typeof RESOLVED_SQL_TYPES[number];
```

Union of `RESOLVED_SQL_TYPES` wire names.

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
const ENTRYPOINTS: readonly ["storefront", "storage", "databaseAdapter", "remoteLibrary", "cli", "oidc", "metadata", "covers"];
```

Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint
analogue). Each value is a capability the host calls over RPC; triggers on
the default entrypoint (`event`, `job`) are declared separately.

Ordinal-ordered `Entrypoint` wire names (index = Cap'n Proto ordinal).

***

### EXPRESSION\_KINDS

```ts
const EXPRESSION_KINDS: readonly ["narration", "text", "translation", "dramatisation", "podcastAudio"];
```

How a work is realised (FRBR expression). Ordinal 0 is the common case
for a storefront that sells audiobooks.

Ordinal-ordered `ExpressionKind` wire names (index = Cap'n Proto ordinal).

***

### FEATURE\_SCALAR\_LIMITS

```ts
const FEATURE_SCALAR_LIMITS: "rpc.scalarLimits";
```

Guest honors scalar / stream-window / list-page caps (`rpc.scalarLimits`).

***

### FEATURE\_STORAGE\_COPY

```ts
const FEATURE_STORAGE_COPY: "storage.copy";
```

Guest implements server-side `Destination.copy` (`storage.copy`).

***

### FEATURE\_STREAMS

```ts
const FEATURE_STREAMS: "rpc.streams";
```

Media moves through transferred `ByteRange` / `ByteSource` streams (`rpc.streams`).

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

### MAX\_SCALAR\_BYTES

```ts
const MAX_SCALAR_BYTES: 262144;
```

Maximum decoded size of an ordinary RPC scalar value (not a stream window).

***

### MAX\_STREAM\_WINDOW\_BYTES

```ts
const MAX_STREAM_WINDOW_BYTES: 1048576;
```

Maximum bytes returned by one `ByteSource.pull` (flow-control window).

***

### MEDIA\_KINDS

```ts
const MEDIA_KINDS: readonly ["audiobook", "ebook", "print", "podcastAudio"];
```

Physical or digital form of an edition (FRBR manifestation). Only
`audiobook` and `podcastAudio` editions are acquire candidates.

Ordinal-ordered `MediaKind` wire names (index = Cap'n Proto ordinal).

***

### PLUGIN\_ERROR\_CODES

```ts
const PLUGIN_ERROR_CODES: readonly ["invalid_params", "unauthorized", "forbidden", "not_found", "unavailable", "unsupported", "internal", "payload_too_large", "deadline_exceeded", "invalid_cursor", "cancelled", "conflict"];
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

### WORK\_KINDS

```ts
const WORK_KINDS: readonly ["book", "podcastShow", "podcastEpisode", "drama"];
```

What kind of creative work a record describes (FRBR work). Ordinal 0 is
the common case, so a guest that leaves it unset describes a book.

Ordinal-ordered `WorkKind` wire names (index = Cap'n Proto ordinal).
