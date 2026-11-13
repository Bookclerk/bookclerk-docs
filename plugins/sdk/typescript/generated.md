# generated

GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp.

Author-facing TypeScript projection of every struct, union, and
interface in `crates/bookclerk-plugin-abi/schema/plugin.capnp`. Field
names are the schema names (camelCase). Cap'n Proto unions become
discriminated `{ kind, value }` unions; `Void` members carry no
`value`. `Int64` is `bigint` (exact SQL integers); `UInt64` counters,
sizes, and unix-millisecond timestamps are `number`. `Data` is
`Uint8Array`. Interfaces are capabilities: their methods return
promises and the wire codecs carry them through a transport cap table.

## Interfaces

### AccountCredential

Host-sealed credentials for one account, delivered on `scan`.

#### Properties

##### accountId

```ts
accountId: string;
```

Account id the blob belongs to.

##### credentials

```ts
credentials: Uint8Array;
```

Opaque credential bytes exactly as the guest returned them at login.

***

### AdapterDatabaseSession

Host ↔ database adapter plugin. Capability negotiation + typed execute only.

#### Methods

##### assertRestoreConstraints()

```ts
assertRestoreConstraints(): Promise<EmptyReply>;
```

Verify constraints hold after restore rows were written.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### bootstrap()

```ts
bootstrap(): Promise<DbBootstrapReply>;
```

Bootstrap-only SeaORM proxy metadata (not part of DbCapabilities).

###### Returns

`Promise`\<[`DbBootstrapReply`](#dbbootstrapreply)\>

[DbBootstrapReply](#dbbootstrapreply)

##### capabilities()

```ts
capabilities(): Promise<DbCapabilitiesReply>;
```

Semantic SQL-contract advertisement for this session.

###### Returns

`Promise`\<[`DbCapabilitiesReply`](#dbcapabilitiesreply)\>

[DbCapabilitiesReply](#dbcapabilitiesreply)

##### close()

```ts
close(): Promise<EmptyReply>;
```

Release the session; further calls fail.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### dropUserRelations()

```ts
dropUserRelations(names): Promise<EmptyReply>;
```

Drop the named user relations during restore.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `names` | `string`[] | Relation names from `listUserRelations`. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### execute()

```ts
execute(request): Promise<ExecuteResultReply>;
```

Canonical SQL + required structured proofs (not JSON).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`AdapterExecuteRequest`](#adapterexecuterequest) | Proven statements plus isolation and receipt. |

###### Returns

`Promise`\<[`ExecuteResultReply`](#executeresultreply)\>

[ExecuteResultReply](#executeresultreply)

##### executeStreamed()

```ts
executeStreamed(request): Promise<ExecuteStreamReply>;
```

`execute` whose reply is framed (`ExecuteStreamFrame`) and counted against
`maxStreamedResultBytes` / `maxStreamedResultRows` instead of the inline
caps. One transaction, one snapshot; `unsupported` when the adapter
advertises no streamed budget.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`AdapterExecuteRequest`](#adapterexecuterequest) | Proven statements plus isolation and receipt. |

###### Returns

`Promise`\<[`ExecuteStreamReply`](#executestreamreply)\>

[ExecuteStreamReply](#executestreamreply)

##### exportIdentity()

```ts
exportIdentity(): Promise<IdentityExportReply>;
```

Snapshot/identity/restore primitives (not a SQL dialect API).

###### Returns

`Promise`\<[`IdentityExportReply`](#identityexportreply)\>

[IdentityExportReply](#identityexportreply)

##### importIdentity()

```ts
importIdentity(rows): Promise<EmptyReply>;
```

Restore identity high-water marks captured by `exportIdentity`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `rows` | [`IdentityHighWater`](#identityhighwater)[] | High-water rows to apply. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### listUserRelations()

```ts
listUserRelations(): Promise<UserRelationsReply>;
```

Names of user relations present in the logical database unit.

###### Returns

`Promise`\<[`UserRelationsReply`](#userrelationsreply)\>

[UserRelationsReply](#userrelationsreply)

##### prepareUnitRestore()

```ts
prepareUnitRestore(): Promise<EmptyReply>;
```

Enter restore mode for one logical unit (`atomicUnitRestore`).

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

***

### AdapterExecuteRequest

Host -> adapter batch: proven statements plus isolation and receipt.

#### Properties

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

Deadline hint; `0` means none.

##### isolation

```ts
isolation: "atomicBatch" | "nestedSavepoint" | "consistentSnapshot";
```

Transaction isolation the adapter must realize.

##### operationId

```ts
operationId: string;
```

Host-chosen idempotency key.

##### receipt

```ts
receipt: AdapterReceipt;
```

Replay receipt to persist; zero/empty when not required.

##### requestHash

```ts
requestHash: string;
```

SHA-256 hex of the idempotency-relevant request.

##### statements

```ts
statements: AdapterStatement[];
```

Statements in execution order.

***

### AdapterReceipt

Replay receipt the host asks the adapter to persist with the batch.

#### Properties

##### guestHash

```ts
guestHash: string;
```

Guest `requestHash` compared on replay.

##### guestLen

```ts
guestLen: number;
```

Guest statement count inside the receipt wrap (excluding host prune/select).

***

### AdapterStatement

One host-lowered statement with its proof.

#### Properties

##### kind

```ts
kind: "execute" | "select" | "returning";
```

Statement classification.

##### maxRows

```ts
maxRows: number;
```

Row cap for queries; `0` means adapter default.

##### parameters

```ts
parameters: DbValue[];
```

Positional bind values.

##### proof

```ts
proof: ResolvedStatement;
```

Typed proof bound to `sql`.

##### resultSelection

```ts
resultSelection: "discard" | "affectedRows" | "rows";
```

Which outcome parts to return.

##### sql

```ts
sql: string;
```

Canonical SQL text.

***

### AuthenticateUserParams

Params of `Oidc.authenticateUser`.

#### Properties

##### password

```ts
password: string;
```

Integration password; never logged by the host.

##### username

```ts
username: string;
```

Integration username / login id.

***

### Bindings

Host-granted bindings for one `PluginWorker.open` (`env` in the Workers
idiom). OS paths, FDs, and sockets are transport-private. Capability fields
are null when the manifest does not declare (or the operator did not grant)
the binding.

#### Properties

##### adapter

```ts
adapter: DatabaseAdapterConfig;
```

Database-adapter bootstrap for the `databaseAdapter` entrypoint. First-party
host-managed adapters receive host-private connect params in `config`;
third-party adapters receive this typed bootstrap (and an empty `config`).

##### cancel?

```ts
cancel?: Cancellation | null;
```

Host cancellation for the whole invocation (fence / lease loss).
Null when the guest does not export this capability.

##### config

```ts
config: ExtensibleConfig;
```

`CONFIG`: granted plugin settings (operator `[vars]` plus the family
settings table) as `application/json`; at most `maxConfigPayloadBytes`.

##### databases

```ts
databases: NamedDatabase[];
```

Named plugin-owned `[[databases]]` bindings: each entry is an isolated
database provisioned by the active adapter, separate from the Bookclerk
library and from every other plugin. Empty when the manifest declares none.

##### events?

```ts
events?: EventPublisher | null;
```

`EVENTS`: outbox publisher; null unless `[[events.producers]]` is granted.
Null when the guest does not export this capability.

##### kv

```ts
kv: NamedKv[];
```

Named `[[kv_namespaces]]` bindings: one host-served durable namespace per
binding, scoped to this plugin. Empty when the manifest declares none.

##### queues

```ts
queues: NamedQueue[];
```

Named `[[queues.producers]]` bindings: one host-served producer per
binding, each sending into the queue the row names. Empty when the
manifest declares none.

##### secrets

```ts
secrets: ExtensibleConfig;
```

`SECRETS`: granted secret values as `application/json`; empty payload
when the manifest declares no `[secrets]`.

##### storage?

```ts
storage?: Destination | null;
```

`WORK_FS`: host-granted object storage for durable plugin files (work
filesystem); null unless `[work_fs]` is granted. Job input/output travel on
`JobRunner.job(controller)`, never here.
Null when the guest does not export this capability.

***

### Brand

Portal brand crossing the RPC boundary. Distinct from `plugin.toml`
`logo`: `iconUrl` is the live URL or data URI the SPA renders.

#### Properties

##### accent

```ts
accent: string;
```

Accent CSS color for highlights / CTAs.

##### bg

```ts
bg: string;
```

Background CSS color (hex or named).

##### fg

```ts
fg: string;
```

Foreground CSS color for text on `bg`.

##### iconUrl?

```ts
iconUrl?: string;
```

Icon URL or data URI for the portal.
Omitted when absent (wire zero value).

##### id

```ts
id: string;
```

Brand id (often matches the plugin id); empty means "no brand".

##### name

```ts
name: string;
```

Display name shown next to the brand swatch.

***

### ByteRange

Half-open byte window `[offset, offset + length)`.

#### Properties

##### length

```ts
length: number;
```

Number of bytes; `0` reads to the end.

##### offset

```ts
offset: number;
```

First byte offset.

***

### ByteSource

Transferred readable byte stream. The capability *is* the stream; callers
pull bounded windows. Abort is capability drop / RPC cancel. A failed pull
MUST set `err` — never a successful empty EOF.

#### Methods

##### pull()

```ts
pull(maxBytes): Promise<PullReply>;
```

Pull the next window of bytes. `done = true` on the final chunk.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `maxBytes` | `number` | Upper bound for this window; at most `maxStreamWindowBytes`. |

###### Returns

`Promise`\<[`PullReply`](#pullreply)\>

[PullReply](#pullreply)

***

### Cancellation

Transport cancellation. SDKs project this into a locally created AbortSignal
(AbortSignal is not a serializable Workers RPC value).

#### Methods

##### poll()

```ts
poll(): Promise<boolean>;
```

Non-blocking check; `true` once the host has fenced the invocation.

###### Returns

`Promise`\<`boolean`\>

`cancelled` (boolean)

***

### CancelledOutcome

Job observed cancellation and stopped.

#### Properties

##### message

```ts
message: string;
```

Short human note.

***

### CatalogCapabilities

Catalog behaviours a storefront declares in `describe()`. Catalog search
reads them instead of special-casing plugin ids; every flag defaults to
false, so a guest that says nothing keeps the plain behaviour.

#### Properties

##### detailByIsbn

```ts
detailByIsbn: boolean;
```

True when `catalogDetail` resolves a bare ISBN (sent in both `productId`
and `isbn`) to a full bibliographic record. Catalog search then asks this
store to fill description, narrators, runtime and genres on lean result
rows from any store that carry an ISBN.

***

### CatalogDetail

Success payload of `ContentSource.catalogDetail`.

#### Properties

##### found

```ts
found: boolean;
```

False when the product is unknown to the storefront (`hit` is empty).

##### hit

```ts
hit: CatalogHit;
```

Full catalog record when `found`.

***

### CatalogDetailParams

Params of `ContentSource.catalogDetail`.

#### Properties

##### isbn?

```ts
isbn?: string;
```

ISBN when it differs from `productId`.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Store product id (Libro ISBN or ISBN-slug).

***

### CatalogHit

One catalog / candidate hit returned by `searchCatalog`,
`expandCandidates`, `listDeals`, and `catalogDetail`.

#### Properties

##### abridgement

```ts
abridgement: "unknown" | "unabridged" | "abridged";
```

Whether the edition is abridged when the storefront says so.

##### categories?

```ts
categories?: string;
```

Category / genre labels as a single string when known.
Omitted when absent (wire zero value).

##### coverUrl?

```ts
coverUrl?: string;
```

Cover image URL.
Omitted when absent (wire zero value).

##### credits

```ts
credits: Credit[];
```

Credited contributors in billing order.

##### currency?

```ts
currency?: string;
```

ISO currency code for `priceCents`.
Omitted when absent (wire zero value).

##### description?

```ts
description?: string;
```

Long description / blurb when fetched.
Omitted when absent (wire zero value).

##### expressionKind

```ts
expressionKind: 
  | "unspecified"
  | "text"
  | "narration"
  | "dramatisation"
  | "graphic"
  | "originalAudio";
```

How the work is realised in this edition.

##### identifiers

```ts
identifiers: Identifier[];
```

External identifiers (ISBN, ASIN, OpenLibrary ids).

##### language?

```ts
language?: string;
```

Content language code when known.
Omitted when absent (wire zero value).

##### lengthMinutes?

```ts
lengthMinutes?: bigint;
```

Runtime in whole minutes.
Omitted when absent (wire zero value).

##### mediaKind

```ts
mediaKind: "unspecified" | "audio" | "ebook" | "print";
```

Carrier of this edition.

##### origin

```ts
origin: string;
```

Hit origin label (plugin id or storefront name).

##### parentProductId?

```ts
parentProductId?: string;
```

Product id of the parent podcast show, for a podcast episode.
Omitted when absent (wire zero value).

##### priceCents?

```ts
priceCents?: bigint;
```

Current price in minor units (cents).
Omitted when absent (wire zero value).

##### priceLabel?

```ts
priceLabel?: string;
```

Pre-formatted price for display.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Storefront product / SKU id.

##### publishedAt?

```ts
publishedAt?: string;
```

Publication date string as provided by the storefront.
Omitted when absent (wire zero value).

##### publisher?

```ts
publisher?: string;
```

Publisher name when known.
Omitted when absent (wire zero value).

##### rankHint?

```ts
rankHint?: number;
```

The storefront's own rank for this hit in `searchCatalog` results:
1-based position in the store's ordering for the requested sort across
all pages (page 2 of 20-hit pages starts at 21), or a store-wide
popularity rank when the store has one. 0 means unranked. Catalog search
merges stores by it for relevance and popularity sorts and puts unranked
hits after ranked ones. Other catalog methods leave it 0.
Omitted when absent (wire zero value).

##### ratingCount?

```ts
ratingCount?: bigint;
```

Number of ratings when known.
Omitted when absent (wire zero value).

##### ratingOverall?

```ts
ratingOverall?: number;
```

Aggregate rating when known.
Omitted when absent (wire zero value).

##### seriesEntries

```ts
seriesEntries: SeriesEntry[];
```

Series the work belongs to.

##### subtitle?

```ts
subtitle?: string;
```

Subtitle when distinct from `title`.
Omitted when absent (wire zero value).

##### title

```ts
title: string;
```

Primary title.

##### url?

```ts
url?: string;
```

Storefront product page URL.
Omitted when absent (wire zero value).

##### workKind

```ts
workKind: "unspecified" | "book" | "podcast";
```

Kind of work the hit is.

***

### CatalogHits

Success payload of the catalog list methods.

#### Properties

##### hits

```ts
hits: CatalogHit[];
```

Hits in storefront order.

***

### ChapterMarker

One chapter marker of a fetched title.

#### Properties

##### startMs

```ts
startMs: number;
```

Chapter start offset in milliseconds from the beginning of the title.

##### title

```ts
title: string;
```

Chapter title.

***

### CliArg

One named argument value passed to `PluginCli.invoke`.

#### Properties

##### name

```ts
name: string;
```

Arg name matching a `CliArgSpec.name`.

##### value

```ts
value: string;
```

String form of the value (the guest parses per `CliArgSpec.kind`).

***

### CliArgSpec

One CLI argument or flag under a `CliCommandSpec`.

#### Properties

##### about?

```ts
about?: string;
```

Help text for this arg.
Omitted when absent (wire zero value).

##### default?

```ts
default?: string;
```

Default string form when the operator omits the arg.
Omitted when absent (wire zero value).

##### kind

```ts
kind: "string" | "bool" | "int" | "path";
```

Parsed value kind.

##### long?

```ts
long?: string;
```

Long flag without leading dashes (e.g. "message" -> `--message`).
Omitted when absent (wire zero value).

##### name

```ts
name: string;
```

Internal arg name used as `CliArg.name` on invoke.

##### positional

```ts
positional: boolean;
```

When true, the arg is positional rather than a flagged option.

##### required

```ts
required: boolean;
```

When true, the host rejects invoke if the arg is missing.

##### short?

```ts
short?: string;
```

Short flag character (e.g. "m" -> `-m`).
Omitted when absent (wire zero value).

***

### CliCommandSpec

One plugin CLI command under `CliSchema`.

#### Properties

##### about?

```ts
about?: string;
```

Short help text for `--help`.
Omitted when absent (wire zero value).

##### args

```ts
args: CliArgSpec[];
```

Argument / flag specs for this command.

##### name

```ts
name: string;
```

Command verb after the plugin id (for example "ping").

***

### CliInvokeParams

Params of `PluginCli.invoke`.

#### Properties

##### args

```ts
args: CliArg[];
```

Named argument values.

##### command

```ts
command: string;
```

Command name matching a `CliCommandSpec.name`.

***

### CliInvokeResult

Result of `PluginCli.invoke`.

#### Properties

##### exitCode

```ts
exitCode: number;
```

Process-style exit code (0 = success).

##### payload

```ts
payload: ExtensibleConfig;
```

Structured payload for machine consumers; empty `mediaType` when absent.

##### stderr

```ts
stderr: string;
```

Captured standard error text.

##### stdout

```ts
stdout: string;
```

Captured standard output text.

***

### CliSchema

Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`).

#### Properties

##### commands

```ts
commands: CliCommandSpec[];
```

Commands exposed as `bookclerk plugins <id> <command> ...`.

***

### ColumnReference

`REFERENCES` target of one column.

#### Properties

##### refColumns

```ts
refColumns: string[];
```

Referenced columns.

##### refTable

```ts
refTable: string;
```

Referenced table.

***

### CompletedOutcome

Job finished successfully.

#### Properties

##### bytesCopied

```ts
bytesCopied: number;
```

Bytes produced, when meaningful.

##### message

```ts
message: string;
```

Short human summary.

***

### ConfigOption

One discoverable config option group advertised for sources.

#### Properties

##### key

```ts
key: string;
```

Config key under the plugin's `config.toml` table.

##### label

```ts
label: string;
```

Operator-facing label for the option group.

##### values

```ts
values: ConfigOptionValue[];
```

Allowed selectable values for this key.

***

### ConfigOptionValue

One selectable value under a `ConfigOption`.

#### Properties

##### id

```ts
id: string;
```

Value written to config when selected.

##### label

```ts
label: string;
```

Operator-facing label for this value.

***

### ContentSource

Storefront content source (not byte Source). Every method takes and
returns typed structs from the "Typed method payloads" section.

#### Methods

##### catalogDetail()

```ts
catalogDetail(params): Promise<CatalogDetailReply>;
```

Full catalog record for one product.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`CatalogDetailParams`](#catalogdetailparams) | Product id and optional ISBN. |

###### Returns

`Promise`\<[`CatalogDetailReply`](#catalogdetailreply)\>

[CatalogDetailReply](#catalogdetailreply)

##### diagnose()

```ts
diagnose(): Promise<DiagnoseReply>;
```

Human-readable diagnostic lines.

###### Returns

`Promise`\<[`DiagnoseReply`](#diagnosereply)\>

[DiagnoseReply](#diagnosereply)

##### expandCandidates()

```ts
expandCandidates(params): Promise<CatalogHitsReply>;
```

Related-title expansion from a seed title.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ExpandCandidatesParams`](#expandcandidatesparams) | Seed identity fields and limit. |

###### Returns

`Promise`\<[`CatalogHitsReply`](#cataloghitsreply)\>

[CatalogHitsReply](#cataloghitsreply)

##### fetchTitle()

```ts
fetchTitle(params): Promise<FetchTitleReply>;
```

Download and decrypt one title into `cacheDir`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`FetchTitleParams`](#fetchtitleparams) | Title, credentials, and fetch options. |

###### Returns

`Promise`\<[`FetchTitleReply`](#fetchtitlereply)\>

[FetchTitleReply](#fetchtitlereply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### listAccounts()

```ts
listAccounts(): Promise<SourceAccountsReply>;
```

Enumerate accounts the guest knows about.

###### Returns

`Promise`\<[`SourceAccountsReply`](#sourceaccountsreply)\>

[SourceAccountsReply](#sourceaccountsreply)

##### listDeals()

```ts
listDeals(params): Promise<CatalogHitsReply>;
```

Current storefront deals.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ListDealsParams`](#listdealsparams) | Optional result cap. |

###### Returns

`Promise`\<[`CatalogHitsReply`](#cataloghitsreply)\>

[CatalogHitsReply](#cataloghitsreply)

##### login()

```ts
login(params): Promise<LoginReply>;
```

Connect an account (password or one-shot OAuth).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`LoginParams`](#loginparams) | Credentials, marketplace, callback wiring. |

###### Returns

`Promise`\<[`LoginReply`](#loginreply)\>

[LoginReply](#loginreply)

##### loginComplete()

```ts
loginComplete(params): Promise<LoginReply>;
```

Finish an interactive OAuth login started by `loginStart`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`LoginCompleteParams`](#logincompleteparams) | Session id from `loginStart`. |

###### Returns

`Promise`\<[`LoginReply`](#loginreply)\>

[LoginReply](#loginreply)

##### loginStart()

```ts
loginStart(params): Promise<LoginStartReply>;
```

Begin an interactive OAuth login; returns a session id.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`LoginParams`](#loginparams) | Same shape as `login`; host fills callback IPC. |

###### Returns

`Promise`\<[`LoginStartReply`](#loginstartreply)\>

[LoginStartReply](#loginstartreply)

##### purchaseHint()

```ts
purchaseHint(params): Promise<PurchaseHintReply>;
```

Purchase link / price hint for one title.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`PurchaseHintParams`](#purchasehintparams) | Identity fields and price flag. |

###### Returns

`Promise`\<[`PurchaseHintReply`](#purchasehintreply)\>

[PurchaseHintReply](#purchasehintreply)

##### purchaseHints()

```ts
purchaseHints(params): Promise<PurchaseHintsReply>;
```

Purchase links / price hints for several titles in one call, so a guest
whose backend prices many products per request makes one request for a
page of cards. A guest that cannot batch answers each query as
`purchaseHint` would.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`PurchaseHintsParams`](#purchasehintsparams) | The titles to price, in order. |

###### Returns

`Promise`\<[`PurchaseHintsReply`](#purchasehintsreply)\>

[PurchaseHintsReply](#purchasehintsreply)

##### revokeAccount()

```ts
revokeAccount(params): Promise<EmptyReply>;
```

Undo store-side state for one account before the host deletes its
stored credentials (for example, deregister the device registered at
login). Best effort: the host revokes the account whatever this returns.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`RevokeAccountParams`](#revokeaccountparams) | Account id and its stored credentials. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### scan()

```ts
scan(params): Promise<ScanReply>;
```

One page of one account's library scan pass (see `ScanPage`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ScanParams`](#scanparams) | Account, cursor, sync token, credential. |

###### Returns

`Promise`\<[`ScanReply`](#scanreply)\>

[ScanReply](#scanreply)

##### searchCatalog()

```ts
searchCatalog(params): Promise<CatalogHitsReply>;
```

Free-text storefront catalog search.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`SearchCatalogParams`](#searchcatalogparams) | Query, region, paging, sort, facet. |

###### Returns

`Promise`\<[`CatalogHitsReply`](#cataloghitsreply)\>

[CatalogHitsReply](#cataloghitsreply)

***

### CopyResult

Summary of a server-side copy.

#### Properties

##### bytesCopied

```ts
bytesCopied: number;
```

Bytes copied.

***

### CoverCondition

Validators the host already holds for one URL, sent as conditional request
headers so an unchanged image costs a `304` instead of its body.

#### Properties

##### ifModifiedSince?

```ts
ifModifiedSince?: string;
```

`If-Modified-Since` value: the `Last-Modified` the host stored,
verbatim; empty for none.
Omitted when absent (wire zero value).

##### ifNoneMatch?

```ts
ifNoneMatch?: string;
```

`If-None-Match` value: the `ETag` the host stored, verbatim; empty for
none.
Omitted when absent (wire zero value).

***

### CoverFetcher

`covers` entrypoint: downloads cover images on the host's behalf, so the
host itself never makes an outbound request to a URL a plugin or a store
supplied. The fetcher only transports bytes: the host sniffs, decodes and
re-encodes every image in its media worker before anything stores or
serves it. Network access is limited to what the manifest's
`[capabilities.network]` declares.

#### Methods

##### fetchCovers()

```ts
fetchCovers(params, sink): Promise<CoverImagesReply>;
```

Fetches every URL concurrently and streams each body into `sink` under
its index in `params.urls` (`"0"`, `"1"`, ...). The reply says, per URL,
whether its body was stored.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`CoverFetchParams`](#coverfetchparams) | URLs and the per-image byte cap. |
| `sink` | [`Destination`](#destination) | Host-served store that accepts only those keys. |

###### Returns

`Promise`\<[`CoverImagesReply`](#coverimagesreply)\>

[CoverImagesReply](#coverimagesreply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

***

### CoverFetchParams

Params of `CoverFetcher.fetchCovers`.

#### Properties

##### conditions

```ts
conditions: CoverCondition[];
```

Conditional request headers per URL, in the same order as `urls`, from
the host's cached copy. Empty, or shorter than `urls`, means the
remaining URLs are fetched unconditionally.

##### maxBytes

```ts
maxBytes: number;
```

Largest body, in bytes, the host accepts for one image; at most
`maxCoverBytes`. The sink refuses a larger body, which fails that URL
only.

##### urls

```ts
urls: string[];
```

Absolute `https://` URLs to fetch, at most `maxCoverUrls`; never empty.
The host has already checked each one against its URL policy.

***

### CoverImage

What happened to one requested URL.

#### Properties

##### age?

```ts
age?: string;
```

`Age` the server (or a cache in front of it) sent, verbatim.
Omitted when absent (wire zero value).

##### cacheControl?

```ts
cacheControl?: string;
```

`Cache-Control` the server sent, verbatim (several lines joined with
`, `).
Omitted when absent (wire zero value).

##### contentType?

```ts
contentType?: string;
```

`Content-Type` the server sent. Advisory: the host sniffs the bytes.
Omitted when absent (wire zero value).

##### date?

```ts
date?: string;
```

`Date` the server sent, verbatim.
Omitted when absent (wire zero value).

##### error?

```ts
error?: string;
```

Why nothing was stored (HTTP status, size cap, timeout, refused host).
Omitted when absent (wire zero value).

##### etag?

```ts
etag?: string;
```

`ETag` the server sent, verbatim. Advisory: the host parses and caps it.
Omitted when absent (wire zero value).

##### expires?

```ts
expires?: string;
```

`Expires` the server sent, verbatim.
Omitted when absent (wire zero value).

##### lastModified?

```ts
lastModified?: string;
```

`Last-Modified` the server sent, verbatim.
Omitted when absent (wire zero value).

##### notModified

```ts
notModified: boolean;
```

The server answered `304 Not Modified` to the URL's conditions: nothing
was put into the sink and the host's copy is still current.

##### stored

```ts
stored: boolean;
```

Whether the body was put into the sink under this URL's index.

##### url

```ts
url: string;
```

The requested URL, verbatim.

***

### CoverImages

Success payload of `CoverFetcher.fetchCovers`.

#### Properties

##### images

```ts
images: CoverImage[];
```

One entry per requested URL, in request order.

***

### CreateTableSchema

Parsed `CREATE TABLE` (canonical SQL v1). Per-column lists align with `columns`.

#### Properties

##### columnChecks

```ts
columnChecks: string[];
```

Per-column CHECK expression text; empty when none.

##### columnDefaults

```ts
columnDefaults: string[];
```

Per-column DEFAULT expression text; empty when none.

##### columnNotNull

```ts
columnNotNull: boolean[];
```

Per-column NOT NULL flags.

##### columnPrimaryKey

```ts
columnPrimaryKey: boolean[];
```

Per-column PRIMARY KEY flags.

##### columnReferences

```ts
columnReferences: OptionalColumnReference[];
```

Per-column REFERENCES targets.

##### columns

```ts
columns: NamedSqlType[];
```

Columns with resolved types, in order.

##### columnUnique

```ts
columnUnique: boolean[];
```

Per-column UNIQUE flags.

##### identityColumn

```ts
identityColumn: string;
```

`INTEGER PRIMARY KEY AUTOINCREMENT` column; empty when none.

##### table

```ts
table: string;
```

Table name.

##### tableConstraints

```ts
tableConstraints: TableConstraint[];
```

Table-level constraints, in order.

***

### Credit

One credited contributor. Credits are listed in billing order.

#### Properties

##### level

```ts
level: "roleDefault" | "work" | "expression" | "edition";
```

Level the credit attaches to.

##### name

```ts
name: string;
```

Contributor display name as the source prints it, one person or body.

##### role

```ts
role: 
  | "author"
  | "narrator"
  | "translator"
  | "editor"
  | "illustrator"
  | "publisher"
  | "host"
  | "cast"
  | "adaptor";
```

What the contributor did.

***

### Database

`databaseAdapter` entrypoint (`Entrypoints.databaseAdapter`).

#### Methods

##### dropUnit()

```ts
dropUnit(unitRef): Promise<EmptyReply>;
```

Physically drop one provisioned binding unit (`unitRef` is adapter-owned).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `unitRef` | `string` | Adapter-native unit (sqlite path, postgres database, D1 name). |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### openSession()

```ts
openSession(): Promise<AdapterSessionReply>;
```

Open one adapter session (capability negotiation + typed execute).

###### Returns

`Promise`\<[`AdapterSessionReply`](#adaptersessionreply)\>

[AdapterSessionReply](#adaptersessionreply)

***

### DatabaseAdapterConfig

Author-facing database adapter configuration carried in
`DatabaseContext.adapter`. This is the generic bootstrap mechanism for
third-party adapters: the operator's granted `[database.<id>]` table plus
the scoped writable data dir. First-party host-managed adapters receive
host-private connect params in `DatabaseContext.config` instead.

#### Properties

##### binding?

```ts
binding?: string;
```

Named plugin database binding this open serves; empty for the primary
library open. Adapters advertising `DbCapabilities.pluginDatabases` must
serve each binding from its own isolated database.
Omitted when absent (wire zero value).

##### instanceId?

```ts
instanceId?: string;
```

Host-issued opaque instance id for this (owner plugin, binding) pair.
Collision-resistant and stable across re-opens. Empty for the primary
library open. Third-party adapters must key isolated databases on this
value rather than `binding` alone (two plugins may both declare `DB`).
Omitted when absent (wire zero value).

##### openExisting

```ts
openExisting: boolean;
```

When true, open an existing binding unit and do not provision a missing
one (read-only backup capture). False lets the adapter create the unit.

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped writable directory for this plugin (`.../plugins/<id>/data`).

##### settings

```ts
settings: ExtensibleConfig;
```

Granted plugin settings (operator `[database.<id>]` table) as
`application/json`; `{}` when the operator configured nothing.

***

### DbBootstrap

Bootstrap-only diagnostic metadata (not a capability).

#### Properties

##### engine

```ts
engine: string;
```

Diagnostic physical engine name. Hosts must not admit or generate SQL
from this value. Any string is valid.

***

### DbCapabilities

Semantic SQL-contract advertisement. Diagnostic engine identity is not
part of the capability plane — see `DbBootstrap`.

#### Properties

##### affectedRows

```ts
affectedRows: boolean;
```

Guest reports `rowsAffected`.

##### atomicBatch

```ts
atomicBatch: boolean;
```

Guest can run a bounded statement list as one SQL transaction.

##### atomicUnitRestore

```ts
atomicUnitRestore: boolean;
```

Adapter can destructively replace one logical database unit so an
ordinary restore failure does not leave that unit partially replaced.

##### cancellation

```ts
cancellation: boolean;
```

Guest honors RPC/session cancellation.

##### consistentBackupRead

```ts
consistentBackupRead: boolean;
```

Adapter can expose one stable logical database state while the host
reads schema, rows, and identity.

##### maxAtomicResultBytes

```ts
maxAtomicResultBytes: number;
```

Maximum encoded bytes of one `ExecuteReply`.

##### maxBinds

```ts
maxBinds: number;
```

Maximum bound parameters per statement.

##### maxCellBytes

```ts
maxCellBytes: number;
```

Maximum UTF-8 / blob bytes of one result cell.

##### maxFunctionArgs

```ts
maxFunctionArgs: number;
```

Maximum arguments in one physical function call after adapter hiding
(nested min / max / coalesce). Portable json_object is already ≤ 32
arguments (16 pairs), matching D1. `0` is unspecified.

##### maxLoweredStatementBytes

```ts
maxLoweredStatementBytes: number;
```

Maximum UTF-8 bytes of sqlite-family lowered SQL the adapter can realize
for one statement (INTEGER overflow wraps, LIKE→GLOB, NULLIF, INSERT OR
IGNORE, query LIMIT wrap, bytes-placeholder expansion). `0` is
unspecified: the host does not enforce a lowered-size ceiling. First-party
D1 advertises 100000. Hosts compare a standardized Bookclerk lowering
upper bound against this number and must not branch on engine identity.

##### maxPatternBytes

```ts
maxPatternBytes: number;
```

Maximum UTF-8 bytes of a BookclerkSQL LIKE pattern value (literals and
TEXT binds). Adapters that expand LIKE into GLOB must advertise a
conservative value that still fits the physical pattern cap. `0` is
unspecified.

##### maxPayloadBytes

```ts
maxPayloadBytes: number;
```

Maximum UTF-8 bytes of SQL plus binds per statement.

##### maxRequestBytes

```ts
maxRequestBytes: number;
```

Maximum encoded bytes of one `ExecuteRequest`.

##### maxResultBytes

```ts
maxResultBytes: number;
```

Maximum encoded bytes of one statement's result rows.

##### maxResultRows

```ts
maxResultRows: number;
```

Maximum rows a query statement may return.

##### maxSchemaColumns

```ts
maxSchemaColumns: number;
```

Maximum columns in one CREATE TABLE / result row. `0` is unspecified.

##### maxStatements

```ts
maxStatements: number;
```

Maximum statements in one atomic batch.

##### maxStreamedResultBytes

```ts
maxStreamedResultBytes: number;
```

Maximum encoded bytes of one `ExecuteReply` answered through
`executeStreamed` (the sum of its frames' statement results). `0` means
the adapter does not stream and callers stay under the inline caps.
When set it is at least `maxAtomicResultBytes`.

##### maxStreamedResultRows

```ts
maxStreamedResultRows: number;
```

Maximum rows one statement may return through `executeStreamed`. `0`
means the adapter does not stream. When set it is at least
`maxResultRows`.

##### pluginDatabases

```ts
pluginDatabases: boolean;
```

Append-only. Adapter can open additional isolated sessions
for plugin-owned database bindings (per-binding file / schema / database).

##### returning

```ts
returning: boolean;
```

Guest SQL supports `RETURNING`.

##### schemaMigrations

```ts
schemaMigrations: boolean;
```

Guest versions schema with a `bookclerk_schema_migrations` table.

##### sqlContractVersion

```ts
sqlContractVersion: number;
```

Bookclerk SQL contract version.

##### timing

```ts
timing: boolean;
```

Guest can fill `DbTiming.dbExecutionUs`. Not a host connect minimum.

***

### DbColumn

Result column descriptor.

#### Properties

##### dbType

```ts
dbType: "unspecified" | "bool" | "text" | "int64" | "float64" | "bytes";
```

Declared or inferred column type.

##### name

```ts
name: string;
```

Column name as projected.

***

### DbRow

One result row.

#### Properties

##### values

```ts
values: DbValue[];
```

Cells in `columns` order.

***

### DbStatement

One guest statement in an `ExecuteRequest`.

#### Properties

##### kind

```ts
kind: "execute" | "select" | "returning";
```

Statement classification.

##### maxRows

```ts
maxRows: number;
```

Row cap for queries; `0` means adapter default.

##### parameters

```ts
parameters: DbValue[];
```

Positional bind values.

##### resultSelection

```ts
resultSelection: "discard" | "affectedRows" | "rows";
```

Which outcome parts to return.

##### sql

```ts
sql: string;
```

BookclerkSQL text; at most `maxScalarBytes`.

***

### DbTiming

Engine timing on `ExecuteReply`.

#### Properties

##### attemptElapsedUs

```ts
attemptElapsedUs: number;
```

Monotonic duration of this handler attempt (microseconds).

##### dbExecutionUs

```ts
dbExecutionUs: number;
```

Engine-reported SQL/transaction time when available (`0` = omitted).

##### dbTimingSource

```ts
dbTimingSource: string;
```

How `dbExecutionUs` was measured.

***

### Destination

Object store the host writes acquired media into (`output.*` plugins).
Keys are relative object paths; the plugin owns the physical layout.

#### Methods

##### abortStage()

```ts
abortStage(key, commitToken): Promise<EmptyReply>;
```

Discard a staged object without publishing it.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key that was staged. |
| `commitToken` | `string` | Token from `WriteOptions.commitToken`. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### commit()

```ts
commit(key, commitToken): Promise<PutReply>;
```

Finalize a destination-side staged object. Staging itself is `put` with
`stageOnly = true`; bytes must stream into destination-managed temp/multipart
storage, never a complete local spool on host/adapter/broker/guest.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key that was staged. |
| `commitToken` | `string` | Token from `WriteOptions.commitToken`. |

###### Returns

`Promise`\<[`PutReply`](#putreply)\>

[PutReply](#putreply)

##### copy()

```ts
copy(from, to): Promise<CopyReply>;
```

Server-side copy (requires `storage.copy`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `from` | `string` | Source object key. |
| `to` | `string` | Destination object key. |

###### Returns

`Promise`\<[`CopyReply`](#copyreply)\>

[CopyReply](#copyreply)

##### delete()

```ts
delete(key): Promise<EmptyReply>;
```

Remove an object; deleting a missing key is a success.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key to remove. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### get()

```ts
get(key, options): Promise<GetReply>;
```

Open an object (or a byte range of it) for reading.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key to read. |
| `options` | [`ReadOptions`](#readoptions) | Optional byte range. |

###### Returns

`Promise`\<[`GetReply`](#getreply)\>

[GetReply](#getreply)

##### head()

```ts
head(key): Promise<HeadReply>;
```

Metadata probe. `found = false` is a success, not `not_found`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key to probe. |

###### Returns

`Promise`\<[`HeadReply`](#headreply)\>

[HeadReply](#headreply)

##### list()

```ts
list(options): Promise<ListReply>;
```

Page through objects under a prefix; at most `maxListPage` per page.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options` | [`ListOptions`](#listoptions) | Prefix, cursor, and page size. |

###### Returns

`Promise`\<[`ListReply`](#listreply)\>

[ListReply](#listreply)

##### put()

```ts
put(
   key, 
   body, 
   options
): Promise<PutReply>;
```

Store an object from a transferred byte stream.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key to write. |
| `body` | [`ByteSource`](#bytesource) | Streamed object bytes. |
| `options` | [`WriteOptions`](#writeoptions) | Content type, length, digest, staging. |

###### Returns

`Promise`\<[`PutReply`](#putreply)\>

[PutReply](#putreply)

***

### DiagnoseResult

Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`.

#### Properties

##### lines

```ts
lines: string[];
```

Human-readable probe lines.

***

### DomainEvent

Domain event (not a job). Outbox-produced, at-least-once, idempotent consume.

#### Properties

##### accountId

```ts
accountId: string;
```

Account scope; empty for host-wide events.

##### causationId

```ts
causationId: string;
```

Id of the command or event that caused this one; empty when none.

##### checkpointJson

```ts
checkpointJson: string;
```

Append-only. Resume a prior EventResult.suspended.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty when none.

##### deduplicationKey

```ts
deduplicationKey: string;
```

Consumer-side idempotency key; stable across redeliveries.

##### deliveryAttempt

```ts
deliveryAttempt: number;
```

Delivery counter, starting at 1.

##### eventId

```ts
eventId: string;
```

Unique event id (outbox row identity).

##### eventType

```ts
eventType: string;
```

Dotted event type (e.g. `library.title.added`).

##### invocationSequence

```ts
invocationSequence: number;
```

Resume ordinal; distinct from `deliveryAttempt`.

##### occurredAtUnixMs

```ts
occurredAtUnixMs: number;
```

When the producer observed the fact.

##### payload

```ts
payload: Uint8Array;
```

Encoded event payload; at most `maxEventPayloadBytes`.

##### resumePending

```ts
resumePending: boolean;
```

True when this delivery resumes a prior suspension.

##### schemaVersion

```ts
schemaVersion: number;
```

Schema version of `payload`, owned by the event type.

##### source

```ts
source: string;
```

Append-only. Producer plugin id; empty when unknown.

***

### Entrypoints

Exported entrypoints returned by `PluginWorker.open`. One capability per
`plugin.toml` `entrypoints` entry / trigger; null when not exported. The host
refuses an entrypoint the manifest or operator grant did not allow.

#### Properties

##### cli?

```ts
cli?: PluginCli | null;
```

`cli` entrypoint.
Null when the guest does not export this capability.

##### covers?

```ts
covers?: CoverFetcher | null;
```

`covers` entrypoint.
Null when the guest does not export this capability.

##### databaseAdapter?

```ts
databaseAdapter?: Database | null;
```

`databaseAdapter` entrypoint.
Null when the guest does not export this capability.

##### eventConsumer?

```ts
eventConsumer?: EventConsumer | null;
```

`[[events.consumers]]` trigger: `event(batch)` handler.
Null when the guest does not export this capability.

##### jobRunner?

```ts
jobRunner?: JobRunner | null;
```

`[triggers] jobs` trigger: `job(controller)` handler.
Null when the guest does not export this capability.

##### metadata?

```ts
metadata?: MetadataProvider | null;
```

`metadata` entrypoint.
Null when the guest does not export this capability.

##### oidc?

```ts
oidc?: Oidc | null;
```

`oidc` entrypoint.
Null when the guest does not export this capability.

##### queueConsumer?

```ts
queueConsumer?: QueueConsumer | null;
```

`[[queues.consumers]]` trigger: `queue(batch)` handler.
Null when the guest does not export this capability.

##### remoteLibrary?

```ts
remoteLibrary?: RemoteLibrary | null;
```

`remoteLibrary` entrypoint.
Null when the guest does not export this capability.

##### scheduledHandler?

```ts
scheduledHandler?: ScheduledHandler | null;
```

`[triggers] crons` trigger: `scheduled(controller)` handler.
Null when the guest does not export this capability.

##### storage?

```ts
storage?: Destination | null;
```

`storage` entrypoint.
Null when the guest does not export this capability.

##### storefront?

```ts
storefront?: ContentSource | null;
```

`storefront` entrypoint.
Null when the guest does not export this capability.

***

### EventAck

Event handled; the host marks it delivered.

#### Properties

##### dummy

```ts
dummy: void;
```

Placeholder; the struct carries no data.

***

### EventBatch

One `EventConsumer.event` delivery: at most `maxListPage` events, each
bounded by `maxEventPayloadBytes` / `maxCheckpointBytes`.

#### Properties

##### events

```ts
events: DomainEvent[];
```

Events in delivery order; the reply carries one `EventResult` per entry.

***

### EventConsumer

`[[events.consumers]]` handler (`Entrypoints.eventConsumer`). Delivery is
at-least-once; consumers must be idempotent on `deduplicationKey`.

#### Methods

##### event()

```ts
event(batch): Promise<EventBatchReply>;
```

Deliver one ordered batch of domain events.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `batch` | [`EventBatch`](#eventbatch) | Events in delivery order. |

###### Returns

`Promise`\<[`EventBatchReply`](#eventbatchreply)\>

[EventBatchReply](#eventbatchreply)

***

### EventConsumerSpec

One declared event consumer: a `[[events.consumers]]` row the default
entrypoint's `event(batch)` handler accepts.

#### Properties

##### eventType

```ts
eventType: string;
```

Versioned event type (snake_case, e.g. `book_acquired`).

##### schemaVersions

```ts
schemaVersions: number[];
```

Schema versions the guest can consume; never empty.

##### supportsSuspend

```ts
supportsSuspend: boolean;
```

Whether `EventResult.suspended` is supported for this type.

***

### EventDeadLetter

Event moved to the dead-letter queue for operator review.

#### Properties

##### reason

```ts
reason: string;
```

Short human reason.

***

### EventPollResult

Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick
off workflows.

#### Properties

##### users

```ts
users: ExternalUser[];
```

Newly observed external users since the last poll.

***

### EventPublisher

`EVENTS` binding: host-served outbox publisher granted through
`Bindings.events`. Event types must be listed in `[[events.producers]]`.

#### Methods

##### publish()

```ts
publish(event): Promise<PublishReply>;
```

Append one event to the outbox (idempotent on `deduplicationKey`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `event` | [`PluginEvent`](#pluginevent) | Event to publish. |

###### Returns

`Promise`\<[`PublishReply`](#publishreply)\>

[PublishReply](#publishreply)

##### read()

```ts
read(request): Promise<EventReadReply>;
```

Read one page of past outbox events this instance subscribes to, oldest
first, to catch up after a gap (`env.EVENTS.read`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`EventReadRequest`](#eventreadrequest) | Where to start, which types, how many. |

###### Returns

`Promise`\<[`EventReadReply`](#eventreadreply)\>

[EventReadReply](#eventreadreply)

***

### EventReadPage

Success payload of `EventPublisher.read`.

#### Properties

##### cursor

```ts
cursor: string;
```

Opaque position after the last event scanned; pass it as `after` to
continue. Empty only when nothing was scanned and the request had no
cursor. At most `maxEventCursorBytes`.

##### events

```ts
events: DomainEvent[];
```

Matching events in outbox order. `deliveryAttempt` is zero and
`deduplicationKey` is the event id: a read is not a delivery, so dedupe
on `eventId`.

##### expiredEventIds

```ts
expiredEventIds: string[];
```

Ids of the events on this page whose delivery to this plugin expired
unleased: the events the instance missed.

##### more

```ts
more: boolean;
```

True when more events may follow: the page filled or the host's scan
budget ran out before the newest event.

##### truncated

```ts
truncated: boolean;
```

True when the start position is older than `[events].retention_days`, so
events before the retained window may be gone.

***

### EventReadRequest

Request of `EventPublisher.read`: one page of past outbox events, oldest
first. Only types the instance's consented `[[events.consumers]]` rows
subscribe to are returned, through the same schema-version and filter match
as delivery, and only back to `[events].retention_days`.

#### Properties

##### after

```ts
after: string;
```

Opaque cursor from an earlier `EventReadPage.cursor`; empty starts at
`sinceUnixMs`. At most `maxEventCursorBytes`. Set at most one of `after`
and `sinceUnixMs`.

##### limit

```ts
limit: number;
```

Most events to return; zero means `maxListPage`. At most `maxListPage`.

##### sinceUnixMs

```ts
sinceUnixMs: number;
```

UTC Unix milliseconds to start from when `after` is empty; zero starts at
the oldest retained event.

##### types

```ts
types: string[];
```

Event types to return; empty means every consented subscription. Each
entry must be a consented subscription type; at most `maxListPage`.

***

### EventReject

Event rejected; the host records the reason and stops delivering.

#### Properties

##### reason

```ts
reason: string;
```

Short human reason.

***

### EventRetry

Redeliver later.

#### Properties

##### reason

```ts
reason: string;
```

Short human reason.

##### retryAtUnixMs

```ts
retryAtUnixMs: number;
```

Earliest redelivery time; `0` lets the host choose.

***

### EventSuspended

Append-only. Mirrors job SuspendedOutcome; event handlers persist a
bounded checkpoint and release the process until wakeAtUnixMs. Optional
wake-on-matching-event fields (empty = timestamp-only).

#### Properties

##### checkpointJson

```ts
checkpointJson: string;
```

Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### wakeAtUnixMs

```ts
wakeAtUnixMs: number;
```

Earliest resume time.

##### wakeOnEventType

```ts
wakeOnEventType: string;
```

Also wake when an event of this type arrives; empty disables.

##### wakeOnFilterJson

```ts
wakeOnFilterJson: string;
```

JSON filter applied to matching wake events; empty matches all.

***

### ExecuteReply

Success payload of `execute`.

#### Properties

##### operationId

```ts
operationId: string;
```

Echo of the request `operationId`.

##### statements

```ts
statements: StatementResult[];
```

Per-statement results in request order.

##### timing

```ts
timing: DbTiming;
```

Engine timing.

***

### ExecuteRequest

Guest statement batch for `GuestDatabase.execute`; runs atomically.

#### Properties

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

Deadline hint; `0` means none.

##### operationId

```ts
operationId: string;
```

Caller-chosen idempotency key.

##### requestHash

```ts
requestHash: string;
```

SHA-256 hex of the idempotency-relevant request; empty when omitted.

##### statements

```ts
statements: DbStatement[];
```

Statements in execution order; at most `maxStatements`.

***

### ExecuteStreamHead

First frame of a streamed execute reply: what the reply is about.

#### Properties

##### operationId

```ts
operationId: string;
```

Echo of the request `operationId`.

##### statementCount

```ts
statementCount: number;
```

Number of `statement` frames that follow, one per request statement.

***

### ExecuteStreamStatement

Opens one statement result inside a streamed execute reply. Its rows, if
any, follow in `rows` frames until the next `statement`, `end` or `err`.

#### Properties

##### columns

```ts
columns: DbColumn[];
```

Result column descriptors.

##### index

```ts
index: number;
```

Request statement index, `0`-based and increasing by one per frame.

##### rowsAffected

```ts
rowsAffected: number;
```

Rows changed by a mutation.

***

### ExpandCandidatesParams

Params of `ContentSource.expandCandidates`. Seed fields identify a known
title; the guest returns related catalog hits.

#### Properties

##### asin?

```ts
asin?: string;
```

Seed Amazon ASIN.
Omitted when absent (wire zero value).

##### authors?

```ts
authors?: string;
```

Seed authors string.
Omitted when absent (wire zero value).

##### isbn?

```ts
isbn?: string;
```

Seed ISBN.
Omitted when absent (wire zero value).

##### limit

```ts
limit: number;
```

Maximum candidates to return; the host always sends an explicit value.

##### narrators?

```ts
narrators?: string;
```

Seed narrators string.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Seed storefront product id.

##### region

```ts
region: string;
```

Storefront region / marketplace code.

##### series?

```ts
series?: string;
```

Seed series name.
Omitted when absent (wire zero value).

##### seriesAsin?

```ts
seriesAsin?: string;
```

Seed series ASIN when known.
Omitted when absent (wire zero value).

##### source

```ts
source: string;
```

Source plugin id hint when expanding across storefronts.

##### title

```ts
title: string;
```

Seed title text.

***

### ExtensibleConfig

Plugin-specific extensible config. Not a substitute for typed ABI fields.

#### Properties

##### mediaType

```ts
mediaType: string;
```

Media type of `payload` (e.g. `application/json`).

##### payload

```ts
payload: Uint8Array;
```

Bounded encoded payload; at most `maxConfigPayloadBytes`.

##### schemaVersion

```ts
schemaVersion: number;
```

Version of `payload`'s schema, owned by the plugin.

***

### ExternalUser

One external user observed by an integration. The host may mint claim
tickets without exposing portal details to the guest.

#### Properties

##### accessToken?

```ts
accessToken?: string;
```

Ephemeral remote token (e.g. ABS JWT). Guest-to-host only; never
persisted.
Omitted when absent (wire zero value).

##### displayName?

```ts
displayName?: string;
```

Display name for UI.
Omitted when absent (wire zero value).

##### externalUserId

```ts
externalUserId: string;
```

Provider-scoped user id.

##### provider

```ts
provider: string;
```

Integration provider id (often the plugin id).

***

### FetchedFile

One file `fetchTitle` wrote under the cache directory.

#### Properties

##### contentType?

```ts
contentType?: string;
```

MIME type when the guest knows it (`audio/mp4`, `application/epub+zip`,
`image/jpeg`); the host sniffs the file either way.
Omitted when absent (wire zero value).

##### durationMs?

```ts
durationMs?: number;
```

Playing time of the file in milliseconds, for audio, when known.
Omitted when absent (wire zero value).

##### path

```ts
path: string;
```

Absolute path to the file under `cacheDir`.

##### role

```ts
role: "unspecified" | "part" | "whole" | "cover";
```

What the file is to the edition.

##### title?

```ts
title?: string;
```

Part title (disc or chapter label), for a `part`.
Omitted when absent (wire zero value).

***

### FetchOptions

Fetch-relevant acquire knobs the host forwards so external load matches
in-process. Packaging / naming knobs stay host-side.

#### Properties

##### chapterLayout

```ts
chapterLayout: string;
```

Preferred chapter API layout when fetching (`tree` or `flat`).

##### coverSize

```ts
coverSize: string;
```

Cover image size request (`500`, `1215`, or `native`).

##### downloadClipsBookmarks

```ts
downloadClipsBookmarks: boolean;
```

When true, download clips/bookmarks sidecars when offered.

##### downloadCover

```ts
downloadCover: boolean;
```

When true, download a cover image alongside audio.

##### downloadPdf

```ts
downloadPdf: boolean;
```

When true, download a companion PDF when the store exposes one.

##### downloadSpeedLimitKbps

```ts
downloadSpeedLimitKbps: number;
```

Fetch speed cap in KB/s (`0` = unlimited).

##### retainAaxFile

```ts
retainAaxFile: boolean;
```

When true, keep the encrypted download in storage.

##### saveMetadataJson

```ts
saveMetadataJson: boolean;
```

When true, persist raw catalog API JSON as `metadata.json`.

##### stripAudibleBrandAudio

```ts
stripAudibleBrandAudio: boolean;
```

When true, trim Audible brand intro/outro from the remux window.

##### widevine

```ts
widevine: boolean;
```

Prefer Widevine/CENC download when the store offers it.

##### widevineCdmPath?

```ts
widevineCdmPath?: string;
```

Local Widevine `.wvd` path granted to the guest.
Omitted when absent (wire zero value).

##### widevineCdmProvider?

```ts
widevineCdmProvider?: string;
```

Remote L3 CDM provider URL; empty means the classic default, `off`
disables remote provisioning.
Omitted when absent (wire zero value).

##### xheAac

```ts
xheAac: boolean;
```

Prefer xHE-AAC on the Widevine path when offered.

***

### FetchTitleParams

Params of `ContentSource.fetchTitle`. The plugin writes media under
`cacheDir` and returns plain (DRM-free) paths. The host injects
credentials; guests must not open `library.db` or `master.key`.

#### Properties

##### accountId

```ts
accountId: string;
```

Account whose credentials apply.

##### cacheDir

```ts
cacheDir: string;
```

Absolute path the guest should write media into (jail-granted TMPDIR).

##### credentials?

```ts
credentials?: Uint8Array<ArrayBufferLike>;
```

Host-loaded credential blob for this account; empty when unavailable.
Omitted when absent (wire zero value).

##### fetch

```ts
fetch: FetchOptions;
```

Fetch-relevant acquire options.

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped plugin data directory.

##### sourceConfig

```ts
sourceConfig: ExtensibleConfig;
```

Granted `[sources.<id>]` table as `application/json`.

##### titleId

```ts
titleId: string;
```

Library / storefront title id to download.

***

### ForeignKeyConstraint

Table-level `FOREIGN KEY` constraint.

#### Properties

##### columns

```ts
columns: string[];
```

Local columns.

##### refColumns

```ts
refColumns: string[];
```

Referenced columns.

##### refTable

```ts
refTable: string;
```

Referenced table.

***

### GetOk

Success payload of `Destination.get`.

#### Properties

##### body?

```ts
body?: ByteSource | null;
```

Streamed object bytes.
Null when the guest does not export this capability.

##### meta

```ts
meta: ObjectMetadata;
```

Object metadata.

***

### GuestDatabase

Host-granted SQL for job plugin authors. SDK `DatabaseBinding` mirrors the
Cloudflare Workers D1 surface (`prepare`/`bind`/`run`/`first`/`all`/`raw`,
`batch`, `exec`) over this typed `execute` transport; wire types stay Cap'n
`ExecuteRequest`/`ExecuteReply`.

#### Methods

##### close()

```ts
close(): Promise<EmptyReply>;
```

Release the session; further calls fail.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### execute()

```ts
execute(request): Promise<ExecuteResultReply>;
```

Run one typed statement batch.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ExecuteRequest`](#executerequest) | Statements, binds, and deadline. |

###### Returns

`Promise`\<[`ExecuteResultReply`](#executeresultreply)\>

[ExecuteResultReply](#executeresultreply)

##### executeStreamed()

```ts
executeStreamed(request): Promise<ExecuteStreamReply>;
```

`execute` whose reply is framed (`ExecuteStreamFrame`) and counted against
the session's `maxStreamedResultBytes` / `maxStreamedResultRows`, so a
result past the inline caps still arrives whole. `unsupported` when the
session advertises no streamed budget.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ExecuteRequest`](#executerequest) | Statements, binds, and deadline. |

###### Returns

`Promise`\<[`ExecuteStreamReply`](#executestreamreply)\>

[ExecuteStreamReply](#executestreamreply)

##### limits()

```ts
limits(): Promise<DbCapabilitiesReply>;
```

Negotiated limits of this session, so guests size requests and reads
before sending instead of discovering caps from errors.

###### Returns

`Promise`\<[`DbCapabilitiesReply`](#dbcapabilitiesreply)\>

[DbCapabilitiesReply](#dbcapabilitiesreply)

***

### HeadOk

Success payload of `Destination.head`.

#### Properties

##### found

```ts
found: boolean;
```

Whether the object exists.

##### meta

```ts
meta: ObjectMetadata;
```

Object metadata; meaningful only when `found`.

***

### HealthOk

Typed liveness report.

#### Properties

##### detail

```ts
detail: string;
```

Short human status line; empty when none.

##### ok

```ts
ok: boolean;
```

True when the guest is healthy enough for traffic.

***

### Identifier

One external identifier a storefront or metadata provider reports.

#### Properties

##### marketplace?

```ts
marketplace?: string;
```

Marketplace the id is scoped to (ASIN); absent for global schemes.
Omitted when absent (wire zero value).

##### scheme

```ts
scheme: "isbn" | "asin" | "openLibraryEdition" | "openLibraryWork";
```

Scheme the value belongs to; it decides the identifier's level.

##### value

```ts
value: string;
```

The identifier as the source printed it; the host trims and normalises.

***

### IdentityHighWater

Adapter-private identity high-water (sqlite_sequence / bookclerk_identity).
Column names live in the canonical backup schema, not this catalog.

#### Properties

##### last

```ts
last: bigint;
```

Highest generated or stored value that must not be reused.

##### table

```ts
table: string;
```

Table whose identity column the mark belongs to.

***

### IntegerArithSite

One INTEGER arithmetic expression that must not wrap or error on overflow.

#### Properties

##### full

```ts
full: SqlSpan;
```

Full expression span (`a + b` or `abs(n)`).

##### kind

```ts
kind: "add" | "sub" | "mul" | "abs";
```

Operator.

##### lhs

```ts
lhs: SqlSpan;
```

Left operand (or `abs` argument).

##### rhs

```ts
rhs: SqlSpan;
```

Right operand (`abs` repeats `lhs`).

***

### Invocation

Identity of one `PluginWorker.open` invocation. The host issues ids; guests
echo `correlationId` / `causationId` onto published events.

#### Properties

##### accountId

```ts
accountId: string;
```

Account scope; empty for operator / host-wide invocations.

##### causationId

```ts
causationId: string;
```

Id of the event or command that caused this invocation; empty when none.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty when none.

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

UTC Unix milliseconds; zero when the invocation has no deadline. The host
fence is authoritative.

##### id

```ts
id: string;
```

Unique host-issued invocation id.

***

### JobController

Everything one `JobRunner.job` invocation may touch. Capabilities are
host-served; the guest never sees OS paths or sockets.

#### Properties

##### cancel?

```ts
cancel?: Cancellation | null;
```

Host cancellation probe for this job (fence / lease).
Null when the guest does not export this capability.

##### input?

```ts
input?: Source | null;
```

Job input objects.
Null when the guest does not export this capability.

##### invocation

```ts
invocation: JobInvocation;
```

Durable command envelope.

##### output?

```ts
output?: Destination | null;
```

Job output object store.
Null when the guest does not export this capability.

##### progress?

```ts
progress?: ProgressSink | null;
```

Progress reporter (host coalesces frequent updates).
Null when the guest does not export this capability.

***

### JobInvocation

Durable command envelope (not a domain event). Command payload schema
version and checkpoint schema versions are independent of the plugin ABI.
The envelope itself is not persisted as opaque bytes across ABI majors.
Idempotency keys are scoped to (account, plugin, commandType) until a
terminal fenced outcome is committed.

#### Properties

##### attempt

```ts
attempt: number;
```

Failure retry counter, starting at 1.

##### causationId

```ts
causationId: string;
```

Id of the event or command that caused this one; empty when none.

##### checkpointJson

```ts
checkpointJson: string;
```

Checkpoint persisted by a prior `SuspendedOutcome`; empty on first run.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### commandType

```ts
commandType: string;
```

Command type the handler dispatches on.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty when none.

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

UTC Unix milliseconds. Host fence/lease is authoritative; this hint must
not outlive the fence (clock skew across VPS nodes).

##### idempotencyKey

```ts
idempotencyKey: string;
```

Caller idempotency key scoped to (account, plugin, commandType).

##### invocationId

```ts
invocationId: string;
```

Unique id of this invocation attempt.

##### invocationSequence

```ts
invocationSequence: number;
```

Resume ordinal; distinct from failure `attempt`.

##### payloadJson

```ts
payloadJson: string;
```

Command payload (JSON); at most `maxScalarBytes`.

##### payloadSchemaVersion

```ts
payloadSchemaVersion: number;
```

Schema version of `payloadJson`, owned by the command type.

##### stepId

```ts
stepId: string;
```

Optional step identifier for multi-step commands.

***

### JobRunner

`[triggers] jobs` handler (`Entrypoints.jobRunner`); runs one durable command.

#### Methods

##### job()

```ts
job(controller): Promise<HandleReply>;
```

Run one command invocation to a terminal or suspended outcome. Named
database bindings come from `Bindings.databases` at `open`, not per job.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `controller` | [`JobController`](#jobcontroller) | Envelope plus host-served capabilities. |

###### Returns

`Promise`\<[`HandleReply`](#handlereply)\>

[HandleReply](#handlereply)

***

### KvDeleteOk

Success payload of `KvNamespace.delete`.

#### Properties

##### existed

```ts
existed: boolean;
```

True when a live value existed before the delete.

***

### KvGetOk

Success payload of `KvNamespace.get`.

#### Properties

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Absolute expiry (UTC Unix milliseconds); zero means none.

##### found

```ts
found: boolean;
```

True when the key exists and has not expired.

##### metadata

```ts
metadata: string;
```

Stored metadata JSON text; empty when none.

##### value

```ts
value: Uint8Array;
```

Stored value; empty when not found.

***

### KvListKey

One key on a `KvNamespace.list` page.

#### Properties

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Absolute expiry (UTC Unix milliseconds); zero means none.

##### key

```ts
key: string;
```

Key name.

##### metadata

```ts
metadata: string;
```

Stored metadata JSON text; empty when none.

***

### KvListOk

Success payload of `KvNamespace.list`.

#### Properties

##### cursor

```ts
cursor: string;
```

Cursor for the next page; empty when `listComplete`.

##### keys

```ts
keys: KvListKey[];
```

Keys in lexical order.

##### listComplete

```ts
listComplete: boolean;
```

True when no further page exists.

***

### KvListOptions

Page request for `KvNamespace.list`.

#### Properties

##### cursor

```ts
cursor: string;
```

Opaque cursor from a previous page; empty means the first page.

##### limit

```ts
limit: number;
```

Most keys to return; zero means the host default, at most `maxListPage`.

##### prefix

```ts
prefix: string;
```

Only keys starting with this prefix; empty means all.

***

### KvNamespace

Durable key/value namespace: a Workers `KVNamespace`-shaped binding the host
serves from the library database, scoped to the plugin and the namespace a
`[[kv_namespaces]]` row names, granted through `Bindings.kv`. Values live in
the cluster, so an instance moved to another host reads its own keys.

#### Methods

##### delete()

```ts
delete(key): Promise<KvDeleteReply>;
```

Delete one key; a missing key is not an error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to delete. |

###### Returns

`Promise`\<[`KvDeleteReply`](#kvdeletereply)\>

[KvDeleteReply](#kvdeletereply)

##### get()

```ts
get(key): Promise<KvGetReply>;
```

Read one key; `found = false` when absent or expired.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |

###### Returns

`Promise`\<[`KvGetReply`](#kvgetreply)\>

[KvGetReply](#kvgetreply)

##### list()

```ts
list(options): Promise<KvListReply>;
```

List keys in lexical order, one page at a time.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options` | [`KvListOptions`](#kvlistoptions) | Prefix, cursor, and page size. |

###### Returns

`Promise`\<[`KvListReply`](#kvlistreply)\>

[KvListReply](#kvlistreply)

##### put()

```ts
put(
   key, 
   value, 
   options
): Promise<KvPutReply>;
```

Write one key, replacing any previous value and metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `value` | `Uint8Array` | Value, at most the namespace's `max_value_bytes`. |
| `options` | [`KvPutOptions`](#kvputoptions) | Expiration and metadata. |

###### Returns

`Promise`\<[`KvPutReply`](#kvputreply)\>

[KvPutReply](#kvputreply)

***

### KvPutOk

Success payload of `KvNamespace.put`.

#### Properties

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Effective absolute expiry (UTC Unix milliseconds); zero means none.

***

### KvPutOptions

Expiration and metadata for `KvNamespace.put`.

#### Properties

##### expirationTtlSeconds

```ts
expirationTtlSeconds: number;
```

Relative expiry in seconds from the write; zero means none. Ignored when
`expirationUnixMs` is set.

##### expirationUnixMs

```ts
expirationUnixMs: number;
```

Absolute expiry (UTC Unix milliseconds); zero means none.

##### metadata

```ts
metadata: string;
```

Opaque JSON metadata text, at most `maxKvMetadataBytes`; empty means none.

***

### ListDealsParams

Params of `ContentSource.listDeals`.

#### Properties

##### limit?

```ts
limit?: number;
```

Maximum number of deals to return; guest default when 0.
Omitted when absent (wire zero value).

***

### ListeningProgress

One listening-progress row. The host upserts into `listening_progress`
tagged with the plugin id; plugins never open the library DB.

#### Properties

##### asin?

```ts
asin?: string;
```

Amazon ASIN when known.
Omitted when absent (wire zero value).

##### authors?

```ts
authors?: string;
```

Authors string when known.
Omitted when absent (wire zero value).

##### currentTimeSeconds?

```ts
currentTimeSeconds?: number;
```

Current playback position in seconds.
Omitted when absent (wire zero value).

##### durationSeconds?

```ts
durationSeconds?: number;
```

Total duration in seconds when known.
Omitted when absent (wire zero value).

##### externalItemId

```ts
externalItemId: string;
```

Provider-scoped item / library id.

##### externalUserId

```ts
externalUserId: string;
```

Provider-scoped user id.

##### identityId?

```ts
identityId?: bigint;
```

Bookclerk identity row id when already linked.
Omitted when absent (wire zero value).

##### isbn?

```ts
isbn?: string;
```

ISBN when known.
Omitted when absent (wire zero value).

##### isFinished

```ts
isFinished: boolean;
```

When true, the provider marks the item finished.

##### lastListenedAtUnixMs?

```ts
lastListenedAtUnixMs?: number;
```

Last listen timestamp as unix milliseconds (UTC).
Omitted when absent (wire zero value).

##### progress?

```ts
progress?: number;
```

Fractional progress in `0.0..=1.0` when the provider reports it.
Omitted when absent (wire zero value).

##### title?

```ts
title?: string;
```

Title text when known.
Omitted when absent (wire zero value).

***

### ListOptions

Paging options for `Destination.list`.

#### Properties

##### cursor

```ts
cursor: string;
```

Opaque cursor from a previous `ListPage.nextCursor`; empty starts over.

##### limit

```ts
limit: number;
```

Requested page size; clamped to `maxListPage`. `0` means guest default.

##### prefix

```ts
prefix: string;
```

Only keys starting with this prefix; empty lists everything.

***

### ListPage

One page of `Destination.list` results.

#### Properties

##### nextCursor

```ts
nextCursor: string;
```

Cursor for the next page; empty when exhausted.

##### objects

```ts
objects: ObjectInfo[];
```

Objects in this page, in backend order.

***

### LoginCompleteParams

Params of `ContentSource.loginComplete`.

#### Properties

##### sessionId

```ts
sessionId: string;
```

Session id previously returned by `loginStart`.

***

### LoginParams

Params of `ContentSource.login` and `ContentSource.loginStart`. Password
sources fill email/password; OAuth sources use callback / external fields.
There is no files-dir root or library DB path -- only `pluginDataDir`.

#### Properties

##### callbackBind?

```ts
callbackBind?: string;
```

Bind address for OAuth callback servers (`host:port`). Ignored when
`callbackIpc` is set (host owns the TCP listener).
Omitted when absent (wire zero value).

##### callbackIpc?

```ts
callbackIpc?: string;
```

Host-owned callback IPC endpoint the guest must connect to. When set
(with `callbackPublicBase`), the guest must not bind a TCP listener.
Omitted when absent (wire zero value).

##### callbackPublicBase?

```ts
callbackPublicBase?: string;
```

Public base URL for the host TCP listener, e.g. `http://127.0.0.1:12345`.
Omitted when absent (wire zero value).

##### email?

```ts
email?: string;
```

Account email / username for password logins; empty for pure OAuth.
Omitted when absent (wire zero value).

##### external

```ts
external: boolean;
```

When true, use external / paste-redirect OAuth instead of a local
callback server.

##### extra

```ts
extra: ExtensibleConfig;
```

Store-specific knobs as `application/json`; guests may ignore unknowns.

##### force

```ts
force: boolean;
```

When true, overwrite an existing sealed credential for this account.

##### label?

```ts
label?: string;
```

Operator label stored on the account row.
Omitted when absent (wire zero value).

##### marketplace

```ts
marketplace: string;
```

Marketplace / locale for the storefront; empty means the guest default.

##### password?

```ts
password?: string;
```

Account password for password logins; never logged; empty for OAuth.
Omitted when absent (wire zero value).

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped writable directory for this plugin only (`.../plugins/<id>/data`).

##### responseUrl?

```ts
responseUrl?: string;
```

Pre-supplied OAuth redirect URL (paste flow).
Omitted when absent (wire zero value).

##### showQr

```ts
showQr: boolean;
```

Prefer QR output when the guest supports it.

##### timeoutSecs?

```ts
timeoutSecs?: number;
```

Seconds to wait for OAuth callback capture; guest default when 0.
Omitted when absent (wire zero value).

***

### LoginResult

Result of `ContentSource.login` / `loginComplete`: account metadata plus
opaque credentials for the host to seal into `encrypted_secrets`
(`provider = plugin id`). Guests never write secrets into the library DB.

#### Properties

##### account

```ts
account: SourceAccount;
```

Account row fields for the host to upsert.

##### credentials?

```ts
credentials?: Uint8Array<ArrayBufferLike>;
```

Opaque credential blob the host seals; empty when login only refreshed
metadata. Guests choose the encoding (typically JSON bytes).
Omitted when absent (wire zero value).

***

### LoginStartResult

Result of `ContentSource.loginStart` (interactive OAuth). The operator
opens `url`; `loginComplete` later uses `sessionId`.

#### Properties

##### sessionId

```ts
sessionId: string;
```

Opaque session id for `loginComplete`.

##### url

```ts
url: string;
```

Browser URL the operator should open to complete OAuth.

***

### MediaSupport

Media a plugin works with, declared in `describe()` and in the manifest's
`[media]` table. Required for plugins that export the storefront or
metadata entrypoint: the host refuses to spawn one whose `workKinds` or
`lists` is empty, holds `unspecified`, or is wider than its manifest, and
refuses any record whose kinds fall outside the declaration.

#### Properties

##### fetches

```ts
fetches: ("unspecified" | "audio" | "ebook" | "print")[];
```

Edition carriers `ContentSource.fetchTitle` can deliver; a subset of
`lists`. Empty for metadata providers and for stores that only list.

##### lists

```ts
lists: ("unspecified" | "audio" | "ebook" | "print")[];
```

Edition carriers the plugin reports in scans, catalog search and
metadata records.

##### workKinds

```ts
workKinds: ("unspecified" | "book" | "podcast")[];
```

Work kinds the plugin reports.

***

### MetadataChapter

One chapter in a provider's chapter layout. Chapters are listed in
pre-order: a chapter with `depth` n + 1 is nested under the nearest
earlier chapter with `depth` n.

#### Properties

##### depth

```ts
depth: number;
```

Nesting depth; 0 for a top-level chapter.

##### lengthMs?

```ts
lengthMs?: number;
```

Length in milliseconds when known.
Omitted when absent (wire zero value).

##### startMs

```ts
startMs: number;
```

Start offset in milliseconds from the beginning of the title.

##### title

```ts
title: string;
```

Chapter title.

***

### MetadataChapters

Success payload of `MetadataProvider.chapters`.

#### Properties

##### accurate

```ts
accurate: boolean;
```

Whether the provider marks the offsets as exact rather than estimated.

##### brandIntroMs?

```ts
brandIntroMs?: number;
```

Length of the store's spoken brand intro in milliseconds when known.
Omitted when absent (wire zero value).

##### brandOutroMs?

```ts
brandOutroMs?: number;
```

Length of the store's spoken brand outro in milliseconds when known.
Omitted when absent (wire zero value).

##### chapters

```ts
chapters: MetadataChapter[];
```

Chapters in pre-order; empty when not `found`.

##### found

```ts
found: boolean;
```

False when the provider has no chapter layout for the edition.

##### runtimeMs?

```ts
runtimeMs?: number;
```

Total runtime the layout was measured against, in milliseconds.
Omitted when absent (wire zero value).

***

### MetadataChaptersParams

Params of `MetadataProvider.chapters`.

#### Properties

##### identifiers

```ts
identifiers: Identifier[];
```

Identifiers of one edition (an ASIN for Audnexus); never empty.

##### marketplace?

```ts
marketplace?: string;
```

Marketplace to answer for when the provider is regional.
Omitted when absent (wire zero value).

***

### MetadataLookupParams

Params of `MetadataProvider.lookup`.

#### Properties

##### identifiers

```ts
identifiers: Identifier[];
```

Identifiers naming the wanted edition or work; never empty. A provider
ignores schemes it does not serve.

##### limit?

```ts
limit?: number;
```

Most records to return; the provider picks a small default when absent.
Omitted when absent (wire zero value).

##### marketplace?

```ts
marketplace?: string;
```

Marketplace to answer for (`us`, `uk`, ...) when the provider is
regional.
Omitted when absent (wire zero value).

***

### MetadataProvider

`metadata` entrypoint: bibliographic lookups against a public catalog
(OpenLibrary, Audnexus, ...). A provider describes what it finds as
edition records; the host decides identity and never trusts a record to
name an existing library row. Network access is limited to the domains the
manifest's `[capabilities.network]` declares.

#### Methods

##### chapters()

```ts
chapters(params): Promise<MetadataChaptersReply>;
```

Chapter layout of the edition the given identifiers name.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`MetadataChaptersParams`](#metadatachaptersparams) | Identifiers of one edition. |

###### Returns

`Promise`\<[`MetadataChaptersReply`](#metadatachaptersreply)\>

[MetadataChaptersReply](#metadatachaptersreply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### lookup()

```ts
lookup(params): Promise<MetadataRecordsReply>;
```

Records for the editions the given identifiers name.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`MetadataLookupParams`](#metadatalookupparams) | Identifiers to resolve and a result cap. |

###### Returns

`Promise`\<[`MetadataRecordsReply`](#metadatarecordsreply)\>

[MetadataRecordsReply](#metadatarecordsreply)

##### search()

```ts
search(params): Promise<MetadataRecordsReply>;
```

Records matching a title and, optionally, an author.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`MetadataSearchParams`](#metadatasearchparams) | Title, author, language and a result cap. |

###### Returns

`Promise`\<[`MetadataRecordsReply`](#metadatarecordsreply)\>

[MetadataRecordsReply](#metadatarecordsreply)

***

### MetadataRecord

One edition as a metadata provider describes it: the same structured
fields as `CatalogHit`, without store pricing.

#### Properties

##### abridgement

```ts
abridgement: "unknown" | "unabridged" | "abridged";
```

Whether the edition is abridged when the provider says so.

##### categories?

```ts
categories?: string;
```

Category / genre labels as a single string when known.
Omitted when absent (wire zero value).

##### coverUrl?

```ts
coverUrl?: string;
```

Cover image URL.
Omitted when absent (wire zero value).

##### credits

```ts
credits: Credit[];
```

Credited contributors in billing order.

##### description?

```ts
description?: string;
```

Long description / blurb when known.
Omitted when absent (wire zero value).

##### expressionKind

```ts
expressionKind: 
  | "unspecified"
  | "text"
  | "narration"
  | "dramatisation"
  | "graphic"
  | "originalAudio";
```

How the work is realised in this edition.

##### identifiers

```ts
identifiers: Identifier[];
```

External identifiers (ISBN, ASIN, OpenLibrary ids).

##### language?

```ts
language?: string;
```

Content language code when known.
Omitted when absent (wire zero value).

##### lengthMinutes?

```ts
lengthMinutes?: number;
```

Runtime in whole minutes when known.
Omitted when absent (wire zero value).

##### mediaKind

```ts
mediaKind: "unspecified" | "audio" | "ebook" | "print";
```

Carrier of this edition.

##### publishedAt?

```ts
publishedAt?: string;
```

Publication date string as the provider prints it.
Omitted when absent (wire zero value).

##### publisher?

```ts
publisher?: string;
```

Publisher name when known.
Omitted when absent (wire zero value).

##### ratingCount?

```ts
ratingCount?: number;
```

Number of ratings behind `ratingOverall` when known.
Omitted when absent (wire zero value).

##### ratingOverall?

```ts
ratingOverall?: number;
```

Aggregate rating when the provider publishes one.
Omitted when absent (wire zero value).

##### seriesEntries

```ts
seriesEntries: SeriesEntry[];
```

Series the work belongs to.

##### subtitle?

```ts
subtitle?: string;
```

Subtitle when distinct from `title`.
Omitted when absent (wire zero value).

##### title

```ts
title: string;
```

Primary title.

##### url?

```ts
url?: string;
```

The provider's page for this record.
Omitted when absent (wire zero value).

##### workKind

```ts
workKind: "unspecified" | "book" | "podcast";
```

Kind of work the record describes.

***

### MetadataRecords

Success payload of `MetadataProvider.lookup` / `search`.

#### Properties

##### records

```ts
records: MetadataRecord[];
```

Records in the provider's relevance order; empty when nothing matched.

***

### MetadataSearchParams

Params of `MetadataProvider.search`.

#### Properties

##### author?

```ts
author?: string;
```

Author display name to narrow the match when known.
Omitted when absent (wire zero value).

##### language?

```ts
language?: string;
```

Preferred content language code when known.
Omitted when absent (wire zero value).

##### limit?

```ts
limit?: number;
```

Most records to return; the provider picks a small default when absent.
Omitted when absent (wire zero value).

##### marketplace?

```ts
marketplace?: string;
```

Marketplace to answer for when the provider is regional.
Omitted when absent (wire zero value).

##### title

```ts
title: string;
```

Title to match; never empty.

***

### NamedDatabase

One named plugin-owned database binding delivered on `Bindings.databases`.

#### Properties

##### database?

```ts
database?: GuestDatabase | null;
```

Isolated typed SQL session for this binding (plugin-owned schema).
Null when the guest does not export this capability.

##### name

```ts
name: string;
```

Binding name from `plugin.toml` `[[databases]]`.

***

### NamedKv

One named durable namespace delivered on `Bindings.kv`.

#### Properties

##### kv?

```ts
kv?: KvNamespace | null;
```

Host-served namespace.
Null when the guest does not export this capability.

##### name

```ts
name: string;
```

Binding name from `plugin.toml` `[[kv_namespaces]]`.

##### namespace

```ts
namespace: string;
```

Namespace the binding is scoped to.

***

### NamedQueue

One named queue producer delivered on `Bindings.queues`.

#### Properties

##### name

```ts
name: string;
```

Binding name from `plugin.toml` `[[queues.producers]]`.

##### producer?

```ts
producer?: QueueProducer | null;
```

Host-served producer for that queue.
Null when the guest does not export this capability.

##### queue

```ts
queue: string;
```

Queue the binding sends to.

***

### NamedSqlType

Column name paired with its resolved type.

#### Properties

##### name

```ts
name: string;
```

Column name.

##### sqlType

```ts
sqlType: "boolean" | "text" | "integer" | "real" | "blob" | "null";
```

Resolved type.

***

### ObjectInfo

Compact object entry in a list page.

#### Properties

##### key

```ts
key: string;
```

Object key.

##### size

```ts
size: number;
```

Object size in bytes.

***

### ObjectMetadata

Full metadata of one stored object.

#### Properties

##### contentType

```ts
contentType: string;
```

MIME type; empty when unknown.

##### etag

```ts
etag: string;
```

Backend entity tag; empty when unsupported.

##### key

```ts
key: string;
```

Object key.

##### sha256

```ts
sha256: Uint8Array;
```

Raw SHA-256 digest (32 bytes) or empty when unknown.

##### size

```ts
size: number;
```

Object size in bytes.

***

### Oidc

`oidc` entrypoint: relying-party client templates and credential
verification on behalf of the host authorization server.

#### Methods

##### authenticateUser()

```ts
authenticateUser(params): Promise<ExternalUserReply>;
```

Verify remote credentials on behalf of the host.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`AuthenticateUserParams`](#authenticateuserparams) | Username and password. |

###### Returns

`Promise`\<[`ExternalUserReply`](#externaluserreply)\>

[ExternalUserReply](#externaluserreply)

##### clients()

```ts
clients(): Promise<OidcClientsReply>;
```

Plugin-provided OIDC AS client templates. Empty list when unused.

###### Returns

`Promise`\<[`OidcClientsReply`](#oidcclientsreply)\>

[OidcClientsReply](#oidcclientsreply)

***

### OidcClientsOk

Success payload of `Oidc.clients`.

#### Properties

##### clients

```ts
clients: OidcClientTemplate[];
```

Client templates; empty when the plugin is not a relying party.

***

### OidcClientTemplate

Bookclerk-as-IdP relying-party template. Plugins declare callback path and
client id; the host materializes `oidc_clients` rows and remains the AS.
`originConfigKey` is a dotted config path (e.g. integrations.audiobookshelf.base_url).

#### Properties

##### callbackPath

```ts
callbackPath: string;
```

Redirect URI path relative to the integration origin.

##### clientId

```ts
clientId: string;
```

OIDC client id the host materializes.

##### defaultScopes

```ts
defaultScopes: string[];
```

Scopes granted by default.

##### displayName

```ts
displayName: string;
```

Name shown on consent / admin screens.

##### issueRefreshToken

```ts
issueRefreshToken: boolean;
```

When true, the AS issues refresh tokens to this client.

##### originConfigKey

```ts
originConfigKey: string;
```

Dotted config path holding the client's origin URL.

##### publicClient

```ts
publicClient: boolean;
```

When true, the client uses PKCE without a client secret.

***

### OpenOk

Success payload of `Source.open`.

#### Properties

##### body?

```ts
body?: ByteSource | null;
```

Streamed object bytes.
Null when the guest does not export this capability.

##### meta

```ts
meta: ObjectMetadata;
```

Object metadata.

***

### PhysicalAccess

Physical table/column access used for authorization.

#### Properties

##### column

```ts
column: string;
```

Empty = table presence only; "*" = projection wildcard.

##### table

```ts
table: string;
```

Physical table name.

***

### PlainFetch

Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
guests decrypt before responding.

#### Properties

##### chapters

```ts
chapters: ChapterMarker[];
```

Chapter markers; empty when unknown.

##### files

```ts
files: FetchedFile[];
```

The files the guest wrote under the cache directory: the edition's
ordered `part`s or one `whole` file, plus an optional `cover`.

##### mediaKind

```ts
mediaKind: "unspecified" | "audio" | "ebook" | "print";
```

Carrier of what was fetched. Must be one of the storefront's declared
`MediaSupport.fetches`; `unspecified` is refused.

##### pdfUrl?

```ts
pdfUrl?: string;
```

Companion PDF download URL when the store exposes one; the host
downloads it itself.
Omitted when absent (wire zero value).

##### refreshedCredentials?

```ts
refreshedCredentials?: Uint8Array<ArrayBufferLike>;
```

The account's credentials as the guest holds them after refreshing them
during the fetch, as in `ScanPage.refreshedCredentials`. Empty when
nothing changed.
Omitted when absent (wire zero value).

***

### PluginCapabilities

Typed capability declaration returned by `describe()`. The host compares
it with `plugin.toml` and the operator grant; widening is rejected at spawn.

#### Properties

##### bindings

```ts
bindings: string[];
```

Other named bindings the guest expects on `env` (`CONFIG`, `SECRETS`,
`WORK_FS`, `OAUTH`, `KV`, `EVENTS`, `[[queues.producers]]` names, ...).

##### consumes

```ts
consumes: EventConsumerSpec[];
```

Event types the default entrypoint consumes (`event(batch)` trigger).

##### crons

```ts
crons: string[];
```

Five-field UTC cron schedules the default entrypoint handles
(`scheduled(controller)` trigger, `[triggers] crons`), normalized to
single spaces.

##### databases

```ts
databases: string[];
```

Plugin-owned database binding names (`[[databases]]`).

##### entrypoints

```ts
entrypoints: (
  | "storefront"
  | "storage"
  | "databaseAdapter"
  | "remoteLibrary"
  | "cli"
  | "oidc"
  | "metadata"
  | "covers")[];
```

Named entrypoints the guest exports.

##### jobs

```ts
jobs: string[];
```

Command types the default entrypoint runs (`job(controller)` trigger).

##### produces

```ts
produces: string[];
```

Event types the guest may publish through its `EVENTS` binding.

##### queueConsumers

```ts
queueConsumers: string[];
```

Queues the default entrypoint consumes (`queue(batch)` trigger).

##### queueProducers

```ts
queueProducers: string[];
```

Queues the guest may send to through its `[[queues.producers]]` bindings.

***

### PluginCli

`cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`.

#### Methods

##### describe()

```ts
describe(): Promise<CliSchemaReply>;
```

Declared CLI surface.

###### Returns

`Promise`\<[`CliSchemaReply`](#clischemareply)\>

[CliSchemaReply](#clischemareply)

##### invoke()

```ts
invoke(params): Promise<CliInvokeReply>;
```

Run one plugin CLI command.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`CliInvokeParams`](#cliinvokeparams) | Command name and argument values. |

###### Returns

`Promise`\<[`CliInvokeReply`](#cliinvokereply)\>

[CliInvokeReply](#cliinvokereply)

***

### PluginDescribe

Guest identity and negotiation surface returned by `describe()`.

#### Properties

##### aliases

```ts
aliases: string[];
```

Alternate ids accepted for config / CLI targeting.

##### apiVersion

```ts
apiVersion: number;
```

ABI version the guest speaks; must equal `apiVersion`.

##### brand

```ts
brand: Brand;
```

Portal brand colors and icon URL; `brand.id` is empty when the guest has
no brand and the host renders a neutral fallback.

##### capabilities

```ts
capabilities: PluginCapabilities;
```

Exported entrypoints, triggers, and bindings the guest implements. The
host rejects anything wider than the manifest and the operator grant.

##### catalog

```ts
catalog: CatalogCapabilities;
```

Catalog behaviours a storefront declares so the host can use any store
for catalog search without naming it; all false for other kinds.

##### cli

```ts
cli: CliSchema;
```

Embedded CLI schema (same shape as `PluginCli.describe`); empty when unused.

##### configOptions

```ts
configOptions: ConfigOption[];
```

Discoverable config option groups for source UIs.

##### displayName

```ts
displayName: string;
```

Human-readable name for UI lists.

##### id

```ts
id: string;
```

Stable plugin id (`[a-z][a-z0-9_]{0,63}`).

##### media

```ts
media: MediaSupport;
```

Work kinds and edition carriers the plugin lists and fetches; required
for storefronts and metadata providers, empty lists for every other
kind (see `MediaSupport`).

##### passwordEnvVar?

```ts
passwordEnvVar?: string;
```

Env var name operators may set for password helpers; never required for
Accounts UI connect. Empty when the guest accepts none.
Omitted when absent (wire zero value).

##### portalAuthMode

```ts
portalAuthMode: "unspecified" | "password" | "oauth";
```

Portal Accounts connect mode for storefronts.

##### rpcFeatures

```ts
rpcFeatures: string[];
```

Negotiable feature names the guest supports (see `feature*` constants).

##### scalarLimits

```ts
scalarLimits: ScalarLimits;
```

Guest caps when `rpc.scalarLimits` is advertised.

##### sortKey

```ts
sortKey: number;
```

UI sort weight among peers of the same family; lower sorts first.

***

### PluginError

`code` is a snake_case string (`not_found`, `invalid_cursor`, …). Unknown
codes are forwarded as-is; SDKs surface them as PluginErrorCode.unknown
while retaining the raw wire code.

#### Properties

##### code

```ts
code: string;
```

Stable snake_case error code (see `PluginErrorCode`).

##### details

```ts
details: string;
```

Structured extras as one JSON object (`limitExceeded` carries `limit`,
`cap` and `used`; engine failures carry `engineCode`). Empty when none.

##### message

```ts
message: string;
```

Human-readable detail; never contains secrets.

***

### PluginEvent

Guest-published domain event (`EventPublisher.publish`). The host stamps
`eventId`, `source` (the plugin id), and `accountId` from the invocation;
guests cannot forge either.

#### Properties

##### causationId

```ts
causationId: string;
```

Id of the event or command that caused this one; empty inherits
`Invocation.causationId`.

##### correlationId

```ts
correlationId: string;
```

Trace correlation id; empty inherits `Invocation.correlationId`.

##### deduplicationKey

```ts
deduplicationKey: string;
```

Producer idempotency key, unique per (account, source, eventType) in the
outbox; a repeat returns `PublishOk.duplicate = true` with the earlier id.
Empty publishes unconditionally.

##### eventType

```ts
eventType: string;
```

Snake_case event type; must be listed in `[[events.producers]]`.

##### occurredAtUnixMs

```ts
occurredAtUnixMs: number;
```

When the producer observed the fact; zero means "now" on the host clock.

##### payload

```ts
payload: Uint8Array;
```

Encoded event payload; at most `maxEventPayloadBytes`.

##### schemaVersion

```ts
schemaVersion: number;
```

Schema version of `payload`, owned by the event type.

***

### PluginMigration

One plugin-owned migration application. `id` is an opaque plugin-chosen
stable identity (name, UUID, timestamp-like string, or digits-as-text).
Bookclerk assigns no order, version, or predecessor meaning to `id`.
Registration order is the forward sequence.

#### Properties

##### id

```ts
id: string;
```

Opaque plugin-chosen stable identity.

##### operations

```ts
operations: PluginMigrationOp[];
```

Operations in forward order; at most `maxPluginMigrationOps`.

***

### PluginMigrationsOk

Success payload of `PluginWorker.databaseMigrations`.

#### Properties

##### migrations

```ts
migrations: PluginMigration[];
```

At most `maxListPage` entries; aggregate id+SQL bytes at most
`maxPluginMigrationRegistrationBytes`; total operations at most
`maxPluginMigrationTotalOps`.

***

### PluginWorker

Plugin bootstrap capability: the guest's root object (Workers `default`
export analogue). `describe()` first; `open()` once per invocation.

#### Methods

##### databaseMigrations()

```ts
databaseMigrations(binding): Promise<PluginMigrationsReply>;
```

Complete ordered plugin-owned migration sequence for one named binding.
The host calls this before `open` while provisioning `[[databases]]`
(a binding must be migrated before any `Bindings.databases` session is
handed out). Empty list means the binding has no plugin-owned migrations.
Bounded by `maxListPage` / `maxPluginMigrationOps` /
`maxPluginMigrationTotalOps` / `maxScalarBytes` /
`maxPluginMigrationRegistrationBytes`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `binding` | `string` | Binding name from `plugin.toml`. |

###### Returns

`Promise`\<[`PluginMigrationsReply`](#pluginmigrationsreply)\>

[PluginMigrationsReply](#pluginmigrationsreply)

##### describe()

```ts
describe(): Promise<DescribeReply>;
```

Identity, ABI version, negotiated features, and typed capabilities.

###### Returns

`Promise`\<[`DescribeReply`](#describereply)\>

[DescribeReply](#describereply)

##### open()

```ts
open(invocation, bindings): Promise<EntrypointsReply>;
```

Open the exported entrypoints for one invocation with host-granted
bindings. Entrypoints the manifest does not export are null.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `invocation` | [`Invocation`](#invocation) | Host-issued invocation identity. |
| `bindings` | [`Bindings`](#bindings) | Granted `env` bindings. |

###### Returns

`Promise`\<[`EntrypointsReply`](#entrypointsreply)\>

[EntrypointsReply](#entrypointsreply)

##### shutdown()

```ts
shutdown(): Promise<EmptyReply>;
```

Flush and release resources before the process exits.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

***

### ProgressSink

Host-side progress reporter handed to job handlers.

#### Methods

##### report()

```ts
report(percent, message): Promise<EmptyReply>;
```

Report progress; the host coalesces frequent updates.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `percent` | `number` | Completion in `[0, 100]`. |
| `message` | `string` | Short human-readable status line. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

***

### PublishOk

Success payload of `EventPublisher.publish`.

#### Properties

##### duplicate

```ts
duplicate: boolean;
```

True when `deduplicationKey` matched an existing outbox row.

##### eventId

```ts
eventId: string;
```

Outbox event id (new or the earlier row when `duplicate`).

***

### PullOk

Success payload of `ByteSource.pull`.

#### Properties

##### chunk

```ts
chunk: Uint8Array;
```

Next bytes; may be shorter than requested.

##### done

```ts
done: boolean;
```

True when the stream is exhausted after `chunk`.

***

### PurchaseHint

Purchase hint for SPA / CLI purchase deep-links.

#### Properties

##### currency?

```ts
currency?: string;
```

ISO currency code for price fields.
Omitted when absent (wire zero value).

##### listPriceCents?

```ts
listPriceCents?: bigint;
```

List / MSRP price in minor units.
Omitted when absent (wire zero value).

##### listPriceLabel?

```ts
listPriceLabel?: string;
```

Pre-formatted list price.
Omitted when absent (wire zero value).

##### memberPriceCents?

```ts
memberPriceCents?: bigint;
```

Member / Plus price in minor units.
Omitted when absent (wire zero value).

##### memberPriceLabel?

```ts
memberPriceLabel?: string;
```

Pre-formatted member price.
Omitted when absent (wire zero value).

##### priceCents?

```ts
priceCents?: bigint;
```

Current price in minor units.
Omitted when absent (wire zero value).

##### priceLabel?

```ts
priceLabel?: string;
```

Pre-formatted current price.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Storefront product id.

##### title?

```ts
title?: string;
```

Title when resolved.
Omitted when absent (wire zero value).

##### url?

```ts
url?: string;
```

Purchase or product-page URL.
Omitted when absent (wire zero value).

***

### PurchaseHintParams

Params of `ContentSource.purchaseHint`. At least one identity field
(`productId` / `asin` / `isbn` / title+authors) should be set; guests may
return `invalid_params` when none are usable.

#### Properties

##### asin?

```ts
asin?: string;
```

Amazon ASIN when known.
Omitted when absent (wire zero value).

##### authors?

```ts
authors?: string;
```

Authors string for fuzzy lookup.
Omitted when absent (wire zero value).

##### isbn?

```ts
isbn?: string;
```

ISBN when known.
Omitted when absent (wire zero value).

##### productId?

```ts
productId?: string;
```

Storefront product id when known.
Omitted when absent (wire zero value).

##### region

```ts
region: string;
```

Storefront region / marketplace code.

##### title?

```ts
title?: string;
```

Title text for fuzzy lookup.
Omitted when absent (wire zero value).

##### withPrice

```ts
withPrice: boolean;
```

When true, guests should include live price fields when available.

***

### PurchaseHintResult

Success payload of `ContentSource.purchaseHint`.

#### Properties

##### found

```ts
found: boolean;
```

False when the guest could not resolve the title (`hint` is empty).

##### hint

```ts
hint: PurchaseHint;
```

Purchase hint when `found`.

***

### PurchaseHintsParams

Params of `ContentSource.purchaseHints`: the titles to price together.

#### Properties

##### queries

```ts
queries: PurchaseHintParams[];
```

One query per title, each as for `purchaseHint`; at most `maxListPage`.

***

### PurchaseHintsResult

Success payload of `ContentSource.purchaseHints`.

#### Properties

##### results

```ts
results: PurchaseHintResult[];
```

One answer per query, in the order the queries were sent.

***

### PutResult

Summary of a stored (or committed) object.

#### Properties

##### bytesWritten

```ts
bytesWritten: number;
```

Bytes persisted.

##### etag

```ts
etag: string;
```

Backend entity tag; empty when unsupported.

##### key

```ts
key: string;
```

Object key written.

##### sha256

```ts
sha256: Uint8Array;
```

Raw SHA-256 digest of the stored bytes; empty when not computed.

***

### QueueBatch

One `QueueConsumer.queue` delivery: messages from one queue in send order.

#### Properties

##### messages

```ts
messages: QueueMessage[];
```

Messages in delivery order; the reply carries one `QueueResult` per entry.

##### queue

```ts
queue: string;
```

Queue the messages were sent to.

***

### QueueBatchOk

Success payload of `QueueConsumer.queue`: exactly one entry per
`QueueBatch.messages` entry, in order. A short list is a host-side error
and the missing tail is retried.

#### Properties

##### results

```ts
results: QueueResult[];
```

Per-message outcomes in delivery order.

***

### QueueConsumer

`[[queues.consumers]]` handler (`Entrypoints.queueConsumer`). Delivery is
at-least-once; consumers must be idempotent on `QueueMessage.id`.

#### Methods

##### queue()

```ts
queue(batch): Promise<QueueBatchReply>;
```

Deliver one ordered batch of messages from one queue.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `batch` | [`QueueBatch`](#queuebatch) | Messages in delivery order. |

###### Returns

`Promise`\<[`QueueBatchReply`](#queuebatchreply)\>

[QueueBatchReply](#queuebatchreply)

***

### QueueMessage

One message on a Workers-style queue, delivered by `QueueConsumer.queue`.

#### Properties

##### attempts

```ts
attempts: number;
```

Delivery counter, starting at 1.

##### body

```ts
body: Uint8Array;
```

Encoded body; at most `maxEventPayloadBytes`.

##### contentType

```ts
contentType: string;
```

Body encoding: `json`, `text`, or `bytes`.

##### id

```ts
id: string;
```

Outbox row id; stable across redeliveries.

##### source

```ts
source: string;
```

Producer plugin id; empty when the host sent it.

##### timestampUnixMs

```ts
timestampUnixMs: number;
```

When the producer sent the message (UTC Unix milliseconds).

***

### QueueProducer

Queue producer binding: host-served send into the queue a
`[[queues.producers]]` row names, granted through `Bindings.queues`.

#### Methods

##### send()

```ts
send(message): Promise<QueueSendReply>;
```

Send one message.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `message` | [`QueueSendMessage`](#queuesendmessage) | Body, encoding, and delay. |

###### Returns

`Promise`\<[`QueueSendReply`](#queuesendreply)\>

[QueueSendReply](#queuesendreply)

##### sendBatch()

```ts
sendBatch(messages, delaySeconds): Promise<QueueSendReply>;
```

Send up to `maxListPage` messages in order; `delaySeconds` applies to
every message that carries no delay of its own.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `messages` | [`QueueSendMessage`](#queuesendmessage)[] | Messages in send order. |
| `delaySeconds` | `number` | Default delay for the batch. |

###### Returns

`Promise`\<[`QueueSendReply`](#queuesendreply)\>

[QueueSendReply](#queuesendreply)

***

### QueueRetry

Redeliver a queue message later.

#### Properties

##### delaySeconds

```ts
delaySeconds: number;
```

Seconds before redelivery; zero applies the consumer's `retry_delay`.

##### reason

```ts
reason: string;
```

Short human reason.

***

### QueueSendMessage

One message to send through a `[[queues.producers]]` binding.

#### Properties

##### body

```ts
body: Uint8Array;
```

Encoded body; at most `maxEventPayloadBytes`.

##### contentType

```ts
contentType: string;
```

Body encoding: `json` (default when empty), `text`, or `bytes`.

##### delaySeconds

```ts
delaySeconds: number;
```

Seconds before the message may be delivered; zero means now.

***

### QueueSendOk

Success payload of `QueueProducer.send` / `sendBatch`.

#### Properties

##### messageIds

```ts
messageIds: string[];
```

Outbox row ids in send order.

***

### ReadOptions

Options for `Destination.get`.

#### Properties

##### range

```ts
range: ByteRange;
```

Byte range to read; an all-zero range reads the whole object.

***

### RejectedOutcome

Job failed permanently; no retry.

#### Properties

##### message

```ts
message: string;
```

Short human reason.

***

### RemoteLibrary

`remoteLibrary` entrypoint: long-running remote-library lifecycle (start /
stop, library rescan, listening sync, external-user polling). Event delivery
is `EventConsumer`; credential verification is `Oidc`.

#### Methods

##### diagnose()

```ts
diagnose(): Promise<DiagnoseReply>;
```

Human-readable diagnostic lines.

###### Returns

`Promise`\<[`DiagnoseReply`](#diagnosereply)\>

[DiagnoseReply](#diagnosereply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### pollEvents()

```ts
pollEvents(): Promise<EventPollReply>;
```

Drain external users the remote side observed since the last poll.

###### Returns

`Promise`\<[`EventPollReply`](#eventpollreply)\>

[EventPollReply](#eventpollreply)

##### scanLibrary()

```ts
scanLibrary(params): Promise<EmptyReply>;
```

Re-sync the remote library.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`ScanLibraryParams`](#scanlibraryparams) | Full-rescan flag. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### start()

```ts
start(): Promise<EmptyReply>;
```

Start background work after the host has granted bindings.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### stop()

```ts
stop(): Promise<EmptyReply>;
```

Stop background work; the host may drop the capability afterwards.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### syncListening()

```ts
syncListening(): Promise<SyncListeningReply>;
```

Push / pull listening progress.

###### Returns

`Promise`\<[`SyncListeningReply`](#synclisteningreply)\>

[SyncListeningReply](#synclisteningreply)

***

### ResolvedAssignment

Destination assignment `lhs = rhs` (INSERT, UPDATE, ...).

#### Properties

##### column

```ts
column: string;
```

Target column.

##### dest

```ts
dest: "boolean" | "text" | "integer" | "real" | "blob" | "null";
```

Declared column type.

##### source

```ts
source: "boolean" | "text" | "integer" | "real" | "blob" | "null";
```

Resolved type of the assigned expression.

##### table

```ts
table: string;
```

Target table.

***

### ResolvedStatement

Host-produced typed proof bound to one exact canonical statement.

#### Properties

##### assignments

```ts
assignments: ResolvedAssignment[];
```

Mutation destination assignments.

##### functions

```ts
functions: string[];
```

Function names invoked (folded), for authorization.

##### integerArithSites

```ts
integerArithSites: IntegerArithSite[];
```

INTEGER overflow sites (`+` `-` `*` `abs`).

##### outputColumns

```ts
outputColumns: NamedSqlType[];
```

SELECT / RETURNING / VALUES output columns in order.

##### physicalAccesses

```ts
physicalAccesses: PhysicalAccess[];
```

Physical tables/columns referenced (authorization).

##### schemaAction

```ts
schemaAction: SchemaAction;
```

DDL action + fingerprint.

##### statementHash

```ts
statementHash: string;
```

SHA-256 hex of the exact canonical SQL this proof claims.

##### textCollateSites

```ts
textCollateSites: TextCollateSite[];
```

TEXT expression spans needing bytewise collation.

***

### RetryableOutcome

Job failed transiently; the host reschedules it.

#### Properties

##### message

```ts
message: string;
```

Short human reason.

##### retryAfterUnixMs

```ts
retryAfterUnixMs: number;
```

Earliest retry time; `0` lets the host choose.

***

### RevokeAccountParams

Params of `ContentSource.revokeAccount`. The host sends the credentials it
is about to delete so the guest can sign the device out at the store.

#### Properties

##### accountId

```ts
accountId: string;
```

Account being revoked.

##### credentials?

```ts
credentials?: Uint8Array<ArrayBufferLike>;
```

Decrypted credential blob for this account, exactly as login returned
it; never log it. Empty when none is stored.
Omitted when absent (wire zero value).

***

### ScalarLimits

Guest-advertised caps for `rpc.scalarLimits`; never above the file constants.

#### Properties

##### maxListPage

```ts
maxListPage: number;
```

Largest list page; at most `maxListPage`.

##### maxScalarBytes

```ts
maxScalarBytes: number;
```

Largest scalar the guest accepts; at most `maxScalarBytes`.

##### maxStreamWindowBytes

```ts
maxStreamWindowBytes: number;
```

Largest `ByteSource.pull` window; at most `maxStreamWindowBytes`.

***

### ScanBook

One library title returned by `ContentSource.scan`. The host forces `source`
to the plugin id and, until the library stores works, expressions and
editions, maps the record onto one flat library row.

#### Properties

##### abridgement

```ts
abridgement: "unknown" | "unabridged" | "abridged";
```

Whether the edition is abridged when the storefront says so.

##### accountId

```ts
accountId: string;
```

Account that owns this library entry.

##### categories?

```ts
categories?: string;
```

Category / genre labels as a single string when known.
Omitted when absent (wire zero value).

##### credits

```ts
credits: Credit[];
```

Credited contributors in billing order.

##### expressionKind

```ts
expressionKind: 
  | "unspecified"
  | "text"
  | "narration"
  | "dramatisation"
  | "graphic"
  | "originalAudio";
```

How the work is realised in this edition.

##### identifiers

```ts
identifiers: Identifier[];
```

External identifiers (ISBN, ASIN, OpenLibrary ids).

##### language?

```ts
language?: string;
```

Content language code when known.
Omitted when absent (wire zero value).

##### lengthMinutes?

```ts
lengthMinutes?: bigint;
```

Runtime in whole minutes when known.
Omitted when absent (wire zero value).

##### marketplace?

```ts
marketplace?: string;
```

Marketplace / region when known.
Omitted when absent (wire zero value).

##### mediaKind

```ts
mediaKind: "unspecified" | "audio" | "ebook" | "print";
```

Carrier of this edition.

##### parentProductId?

```ts
parentProductId?: string;
```

Product id of the parent podcast show, for a podcast episode.
Omitted when absent (wire zero value).

##### productId

```ts
productId: string;
```

Storefront product / SKU id.

##### publishedAt?

```ts
publishedAt?: string;
```

Publication date as RFC 3339 when known.
Omitted when absent (wire zero value).

##### publisher?

```ts
publisher?: string;
```

Publisher name when known.
Omitted when absent (wire zero value).

##### purchasedAt?

```ts
purchasedAt?: string;
```

When the account bought or added the title, as RFC 3339, when known.
Omitted when absent (wire zero value).

##### seriesEntries

```ts
seriesEntries: SeriesEntry[];
```

Series the work belongs to.

##### subtitle?

```ts
subtitle?: string;
```

Subtitle when distinct from `title`.
Omitted when absent (wire zero value).

##### title

```ts
title: string;
```

Primary title string.

##### workKind

```ts
workKind: "unspecified" | "book" | "podcast";
```

Kind of work the title is.

***

### ScanLibraryParams

Params of `RemoteLibrary.scanLibrary` (remote library sync).

#### Properties

##### force

```ts
force: boolean;
```

When true, force a full rescan even if the guest would otherwise
incremental-sync.

***

### ScanPage

One page of an account's scan pass, the result of `ContentSource.scan`.
A pass is the run of pages from an empty `cursor` to the page whose
`nextCursor` is empty; `syncToken` and `complete` are read from that last
page only. A guest that does not page answers the first call with every
title, an empty `nextCursor` and `complete = true`.

#### Properties

##### books

```ts
books: ScanBook[];
```

Titles for the host to upsert; at most `maxScanPageBooks`.

##### complete

```ts
complete: boolean;
```

On the last page: true when the pass listed every title the account
holds (a full pass), false for a delta pass that listed only changes.

##### nextCursor

```ts
nextCursor: string;
```

Opaque position of the next page; empty ends the pass. At most
`maxScanCursorBytes`; never equal to the cursor this page answered.

##### refreshedCredentials?

```ts
refreshedCredentials?: Uint8Array<ArrayBufferLike>;
```

The account's credentials as the guest holds them after refreshing them
while producing this page (for example a renewed access token): the
same encoding login returned, a JSON object of at most
`maxRefreshedCredentialBytes`. The host stores them in place of the
credentials it sent, unless the account was signed in again meanwhile,
and sends them on the pass's later pages. Empty when nothing changed.
Omitted when absent (wire zero value).

##### removed

```ts
removed: string[];
```

Product ids the storefront reports as gone from the account since the
pass's `syncToken` (delta passes only); at most `maxScanPageBooks`.

##### storePages

```ts
storePages: number;
```

Storefront requests this page took.

##### syncToken

```ts
syncToken: string;
```

On the last page: the opaque token the host sends as
`ScanParams.syncToken` on the account's next pass; empty means the next
pass is a full one. At most `maxScanCursorBytes`.

***

### ScanParams

Params of `ContentSource.scan`: one page of one account's scan pass. The
host injects that account's sealed credentials so the plugin does not need a
private credential store under `pluginDataDir`, and drives the pass page by
page: the first call of a pass sends an empty `cursor`, every later call
echoes the previous page's `nextCursor`.

#### Properties

##### accountId

```ts
accountId: string;
```

Account this pass scans; always one of the plugin's connected accounts.

##### credential

```ts
credential: AccountCredential;
```

Host-loaded credential blob for `accountId`.

##### cursor

```ts
cursor: string;
```

Opaque position the guest returned as `ScanPage.nextCursor`; empty on the
first page of a pass. At most `maxScanCursorBytes`.

##### importEpisodes

```ts
importEpisodes: boolean;
```

When true, import podcast/episode-style rows.

##### importPlusTitles

```ts
importPlusTitles: boolean;
```

When true, import Plus/catalog entitlement titles.

##### pageSize

```ts
pageSize: number;
```

Storefront page size; the host always sends an explicit value.

##### pluginDataDir

```ts
pluginDataDir: string;
```

Scoped plugin data directory.

##### syncToken

```ts
syncToken: string;
```

Opaque token the guest returned as `ScanPage.syncToken` at the end of the
account's last pass; empty asks for a full pass. At most
`maxScanCursorBytes`.

***

### ScheduledController

One fire of a `[triggers] crons` schedule (`scheduled(controller)`).

#### Properties

##### attempt

```ts
attempt: number;
```

Failure attempt, starting at 1.

##### cron

```ts
cron: string;
```

The schedule that fired, normalized to single spaces (`0 3 * * MON`).

##### deadlineUnixMs

```ts
deadlineUnixMs: number;
```

UTC unix milliseconds. Host fence/lease is authoritative; this hint must
not outlive the fence.

##### fireId

```ts
fireId: string;
```

Durable id of this fire, stable across retries; dedupe on it.

##### scheduledTime

```ts
scheduledTime: number;
```

Fire time, UTC unix milliseconds (always a whole minute). Retries of
one fire keep it.

***

### ScheduledHandler

`[triggers] crons` handler (`Entrypoints.scheduledHandler`), the Workers
`scheduled(controller)` trigger. Each fire is delivered at least once;
a fire that throws is retried with backoff, keeping `scheduledTime` and
`fireId`, so handlers dedupe on `fireId`.

#### Methods

##### scheduled()

```ts
scheduled(controller): Promise<EmptyReply>;
```

Run one cron fire to completion.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `controller` | [`ScheduledController`](#scheduledcontroller) | The fire being delivered. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

***

### SchemaCreate

`CREATE TABLE` action with its durable fingerprint.

#### Properties

##### fingerprint

```ts
fingerprint: string;
```

Structured schema fingerprint (hex SHA-256).

##### noop

```ts
noop: boolean;
```

True when the catalog already holds this exact fingerprint.

##### schema

```ts
schema: CreateTableSchema;
```

Parsed table schema.

***

### SearchCatalogParams

Params of `ContentSource.searchCatalog`.

#### Properties

##### field

```ts
field: "author" | "narrator" | "any" | "series" | "genre";
```

Facet restriction.

##### language?

```ts
language?: string;
```

Preferred content language (soft-prioritize; e.g. `en`).
Omitted when absent (wire zero value).

##### limit

```ts
limit: number;
```

Maximum hits to return; 0 means the storefront default (20 hits).

##### page

```ts
page: number;
```

1-based page for storefronts that page.

##### query

```ts
query: string;
```

Free-text search query.

##### region

```ts
region: string;
```

Storefront region / marketplace code; empty means the guest default.

##### sort

```ts
sort: "author" | "relevance" | "popularity" | "rating" | "title";
```

Sort order.

***

### SeriesEntry

A work's place in one series.

#### Properties

##### name

```ts
name: string;
```

Series name as the source prints it.

##### position?

```ts
position?: string;
```

Position label as the source prints it (`1`, `2.5`, `1-3`).
Omitted when absent (wire zero value).

##### storeSeriesId?

```ts
storeSeriesId?: string;
```

The store's own id for the series (an Audible series ASIN) when known.
Omitted when absent (wire zero value).

***

### Source

Read-only byte source for job inputs (not a storefront).

#### Methods

##### open()

```ts
open(key): Promise<OpenReply>;
```

Open an object for streaming reads.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key to open. |

###### Returns

`Promise`\<[`OpenReply`](#openreply)\>

[OpenReply](#openreply)

***

### SourceAccount

Source account metadata returned from login and stored by the host.

#### Properties

##### accountId

```ts
accountId: string;
```

Stable account id within this source plugin.

##### label?

```ts
label?: string;
```

Operator-facing label.
Omitted when absent (wire zero value).

##### marketplace

```ts
marketplace: string;
```

Storefront marketplace / region code (for example `us`, `uk`).

##### scanEnabled

```ts
scanEnabled: boolean;
```

When true, bare / scheduled scans include this account. Explicit CLI
`--account` bypasses this flag.

##### source

```ts
source: string;
```

Source plugin id (the host forces this to the guest's install id).

***

### SourceAccounts

Success payload of `ContentSource.listAccounts`.

#### Properties

##### accounts

```ts
accounts: SourceAccount[];
```

Accounts the guest knows about.

***

### SqlSpan

Byte span in the exact canonical SQL string a proof is bound to.

#### Properties

##### end

```ts
end: number;
```

Exclusive end byte offset.

##### start

```ts
start: number;
```

Inclusive start byte offset.

***

### StatementResult

Outcome of one statement.

#### Properties

##### columns

```ts
columns: DbColumn[];
```

Result column descriptors.

##### rows

```ts
rows: DbRow[];
```

Result rows (empty unless `rows` selected).

##### rowsAffected

```ts
rowsAffected: number;
```

Rows changed by a mutation.

***

### SuspendedOutcome

Job released the process and asks to be resumed later.

#### Properties

##### checkpointJson

```ts
checkpointJson: string;
```

Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.

##### checkpointSchemaVersion

```ts
checkpointSchemaVersion: number;
```

Schema version of `checkpointJson`.

##### wakeAtUnixMs

```ts
wakeAtUnixMs: number;
```

Earliest resume time.

***

### SyncListeningResult

Success payload of `RemoteLibrary.syncListening`.

#### Properties

##### items

```ts
items: ListeningProgress[];
```

Progress snapshots to upsert.

***

### TextCollateSite

One TEXT expression the adapter must collate bytewise (`COLLATE "C"`).

#### Properties

##### span

```ts
span: SqlSpan;
```

Identifier or string-literal span in canonical SQL.

***

### WriteOptions

Options for `Destination.put`.

#### Properties

##### commitToken

```ts
commitToken: string;
```

Destination-side stage-and-publish. Empty means a one-shot put.

##### contentLength

```ts
contentLength: number;
```

Expected body length in bytes; `0` when unknown (chunked).

##### contentType

```ts
contentType: string;
```

MIME type to record; empty when unknown.

##### sha256

```ts
sha256: Uint8Array;
```

Expected raw SHA-256 digest; empty skips verification.

##### stageOnly

```ts
stageOnly: boolean;
```

When true, `put` stages remotely and does not publish until `commit`.

## Type Aliases

### AdapterSessionReply

```ts
type AdapterSessionReply = 
  | {
  kind: "ok";
  value: AdapterDatabaseSession;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Database.openSession`.

***

### CatalogDetailReply

```ts
type CatalogDetailReply = 
  | {
  kind: "ok";
  value: CatalogDetail;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.catalogDetail`.

***

### CatalogHitsReply

```ts
type CatalogHitsReply = 
  | {
  kind: "ok";
  value: CatalogHits;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `searchCatalog` / `expandCandidates` / `listDeals`.

***

### CliInvokeReply

```ts
type CliInvokeReply = 
  | {
  kind: "ok";
  value: CliInvokeResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginCli.invoke`.

***

### CliSchemaReply

```ts
type CliSchemaReply = 
  | {
  kind: "ok";
  value: CliSchema;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginCli.describe`.

***

### CopyReply

```ts
type CopyReply = 
  | {
  kind: "ok";
  value: CopyResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.copy`.

***

### CoverImagesReply

```ts
type CoverImagesReply = 
  | {
  kind: "ok";
  value: CoverImages;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `CoverFetcher.fetchCovers`.

***

### DbBootstrapReply

```ts
type DbBootstrapReply = 
  | {
  kind: "ok";
  value: DbBootstrap;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.bootstrap`.

***

### DbCapabilitiesReply

```ts
type DbCapabilitiesReply = 
  | {
  kind: "ok";
  value: DbCapabilities;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.capabilities`.

***

### DbValue

```ts
type DbValue = 
  | {
  kind: "null";
  value: DbType;
}
  | {
  kind: "boolean";
  value: boolean;
}
  | {
  kind: "int64";
  value: bigint;
}
  | {
  kind: "float64";
  value: number;
}
  | {
  kind: "text";
  value: string;
}
  | {
  kind: "bytes";
  value: Uint8Array;
};
```

One typed SQL cell or bind parameter.

***

### DescribeReply

```ts
type DescribeReply = 
  | {
  kind: "ok";
  value: PluginDescribe;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginWorker.describe`.

***

### DiagnoseReply

```ts
type DiagnoseReply = 
  | {
  kind: "ok";
  value: DiagnoseResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `diagnose`.

***

### EmptyReply

```ts
type EmptyReply = 
  | {
  kind: "ok";
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `methods without a success payload`.

***

### EntrypointsReply

```ts
type EntrypointsReply = 
  | {
  kind: "ok";
  value: Entrypoints;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginWorker.open`.

***

### EventBatchReply

```ts
type EventBatchReply = 
  | {
  kind: "ok";
  value: EventResult[];
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `EventConsumer.event`. `ok` has exactly one entry per
`EventBatch.events` entry, in order; a short list is a host-side error and
the missing tail is retried.

***

### EventPollReply

```ts
type EventPollReply = 
  | {
  kind: "ok";
  value: EventPollResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `RemoteLibrary.pollEvents`.

***

### EventReadReply

```ts
type EventReadReply = 
  | {
  kind: "ok";
  value: EventReadPage;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `EventPublisher.read`.

***

### EventResult

```ts
type EventResult = 
  | {
  kind: "ack";
  value: EventAck;
}
  | {
  kind: "retry";
  value: EventRetry;
}
  | {
  kind: "reject";
  value: EventReject;
}
  | {
  kind: "deadLetter";
  value: EventDeadLetter;
}
  | {
  kind: "suspended";
  value: EventSuspended;
};
```

Per-event outcome inside an `EventConsumer.event` batch reply.

***

### ExecuteResultReply

```ts
type ExecuteResultReply = 
  | {
  kind: "ok";
  value: ExecuteReply;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `execute`.

***

### ExecuteStreamFrame

```ts
type ExecuteStreamFrame = 
  | {
  kind: "head";
  value: ExecuteStreamHead;
}
  | {
  kind: "statement";
  value: ExecuteStreamStatement;
}
  | {
  kind: "rows";
  value: DbRow[];
}
  | {
  kind: "end";
  value: DbTiming;
}
  | {
  kind: "err";
  value: PluginError;
};
```

One frame of a streamed execute reply (`executeStreamed`). Frames are
consecutive unpacked single-segment Cap'n Proto messages, each at most
`maxScalarBytes` unless it carries one row that alone exceeds it. The
sequence is `head`, then for each statement `statement` followed by its
`rows` frames in order, then `end`; `err` ends the sequence early.

***

### ExecuteStreamReply

```ts
type ExecuteStreamReply = 
  | {
  kind: "inline";
  value: Uint8Array;
}
  | {
  kind: "stream";
  value: ByteSource;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `executeStreamed`. The frames (`ExecuteStreamFrame`) travel
inline when they fit one scalar, otherwise as a transferred byte stream in
`maxStreamWindowBytes` windows. The adapter counts the reply against
`maxStreamedResultBytes` and `maxStreamedResultRows` before it commits, so
results still fail closed and are never truncated.

***

### ExternalUserReply

```ts
type ExternalUserReply = 
  | {
  kind: "ok";
  value: ExternalUser;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Oidc.authenticateUser`.

***

### FetchTitleReply

```ts
type FetchTitleReply = 
  | {
  kind: "ok";
  value: PlainFetch;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.fetchTitle`.

***

### GetReply

```ts
type GetReply = 
  | {
  kind: "ok";
  value: GetOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.get`.

***

### GuestDatabaseReply

```ts
type GuestDatabaseReply = 
  | {
  kind: "ok";
  value: GuestDatabase;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `guest database opens`.

***

### HandleReply

```ts
type HandleReply = 
  | {
  kind: "ok";
  value: JobOutcome;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `JobRunner.job`.

***

### HeadReply

```ts
type HeadReply = 
  | {
  kind: "ok";
  value: HeadOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.head`.

***

### HealthReply

```ts
type HealthReply = 
  | {
  kind: "ok";
  value: HealthOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `health`.

***

### IdentityExportReply

```ts
type IdentityExportReply = 
  | {
  kind: "ok";
  value: IdentityHighWater[];
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.exportIdentity`.

***

### JobOutcome

```ts
type JobOutcome = 
  | {
  kind: "completed";
  value: CompletedOutcome;
}
  | {
  kind: "retryable";
  value: RetryableOutcome;
}
  | {
  kind: "rejected";
  value: RejectedOutcome;
}
  | {
  kind: "cancelled";
  value: CancelledOutcome;
}
  | {
  kind: "suspended";
  value: SuspendedOutcome;
};
```

Terminal or suspended result of `JobRunner.job`.

***

### KvDeleteReply

```ts
type KvDeleteReply = 
  | {
  kind: "ok";
  value: KvDeleteOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.delete`.

***

### KvGetReply

```ts
type KvGetReply = 
  | {
  kind: "ok";
  value: KvGetOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.get`.

***

### KvListReply

```ts
type KvListReply = 
  | {
  kind: "ok";
  value: KvListOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.list`.

***

### KvPutReply

```ts
type KvPutReply = 
  | {
  kind: "ok";
  value: KvPutOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `KvNamespace.put`.

***

### ListReply

```ts
type ListReply = 
  | {
  kind: "ok";
  value: ListPage;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.list`.

***

### LoginReply

```ts
type LoginReply = 
  | {
  kind: "ok";
  value: LoginResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.login` / `loginComplete`.

***

### LoginStartReply

```ts
type LoginStartReply = 
  | {
  kind: "ok";
  value: LoginStartResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.loginStart`.

***

### MetadataChaptersReply

```ts
type MetadataChaptersReply = 
  | {
  kind: "ok";
  value: MetadataChapters;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `MetadataProvider.chapters`.

***

### MetadataRecordsReply

```ts
type MetadataRecordsReply = 
  | {
  kind: "ok";
  value: MetadataRecords;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `MetadataProvider.lookup` / `search`.

***

### OidcClientsReply

```ts
type OidcClientsReply = 
  | {
  kind: "ok";
  value: OidcClientsOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Oidc.clients`.

***

### OpenReply

```ts
type OpenReply = 
  | {
  kind: "ok";
  value: OpenOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Source.open`.

***

### OptionalColumnReference

```ts
type OptionalColumnReference = 
  | {
  kind: "none";
}
  | {
  kind: "some";
  value: ColumnReference;
};
```

Per-column `REFERENCES` slot in `CreateTableSchema`.

***

### PluginMigrationOp

```ts
type PluginMigrationOp = 
  | {
  kind: "schema";
  value: string;
}
  | {
  kind: "data";
  value: string;
};
```

One already-separated BookclerkSQL operation in a plugin-owned migration.
`schema` is admitted DDL; `data` is admitted DML. There is no native SQL
escape hatch.

***

### PluginMigrationsReply

```ts
type PluginMigrationsReply = 
  | {
  kind: "ok";
  value: PluginMigrationsOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `PluginWorker.databaseMigrations`.

***

### PublishReply

```ts
type PublishReply = 
  | {
  kind: "ok";
  value: PublishOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `EventPublisher.publish`.

***

### PullReply

```ts
type PullReply = 
  | {
  kind: "ok";
  value: PullOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ByteSource.pull`.

***

### PurchaseHintReply

```ts
type PurchaseHintReply = 
  | {
  kind: "ok";
  value: PurchaseHintResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.purchaseHint`.

***

### PurchaseHintsReply

```ts
type PurchaseHintsReply = 
  | {
  kind: "ok";
  value: PurchaseHintsResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.purchaseHints`. A whole-batch `err` (for
example `rateLimited`) applies to every query in it.

***

### PutReply

```ts
type PutReply = 
  | {
  kind: "ok";
  value: PutResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.put` / `Destination.commit`.

***

### QueueBatchReply

```ts
type QueueBatchReply = 
  | {
  kind: "ok";
  value: QueueBatchOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `QueueConsumer.queue`.

***

### QueueResult

```ts
type QueueResult = 
  | {
  kind: "ack";
}
  | {
  kind: "retry";
  value: QueueRetry;
};
```

Per-message outcome inside a `QueueConsumer.queue` batch reply.

***

### QueueSendReply

```ts
type QueueSendReply = 
  | {
  kind: "ok";
  value: QueueSendOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `QueueProducer.send` / `sendBatch`.

***

### ScanReply

```ts
type ScanReply = 
  | {
  kind: "ok";
  value: ScanPage;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.scan`.

***

### SchemaAction

```ts
type SchemaAction = 
  | {
  kind: "none";
}
  | {
  kind: "create";
  value: SchemaCreate;
}
  | {
  kind: "drop";
  value: string;
};
```

CREATE/DROP action recorded on a proof.

***

### SourceAccountsReply

```ts
type SourceAccountsReply = 
  | {
  kind: "ok";
  value: SourceAccounts;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.listAccounts`.

***

### SyncListeningReply

```ts
type SyncListeningReply = 
  | {
  kind: "ok";
  value: SyncListeningResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `RemoteLibrary.syncListening`.

***

### TableConstraint

```ts
type TableConstraint = 
  | {
  kind: "primaryKey";
  value: string[];
}
  | {
  kind: "unique";
  value: string[];
}
  | {
  kind: "check";
  value: string;
}
  | {
  kind: "foreignKey";
  value: ForeignKeyConstraint;
};
```

One table-level constraint.

***

### UserRelationsReply

```ts
type UserRelationsReply = 
  | {
  kind: "ok";
  value: string[];
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AdapterDatabaseSession.listUserRelations`.

## References

### Abridgement

Re-exports [Abridgement](abi.md#abridgement)

***

### CatalogField

Re-exports [CatalogField](abi.md#catalogfield)

***

### CatalogSort

Re-exports [CatalogSort](abi.md#catalogsort)

***

### CliArgKind

Re-exports [CliArgKind](abi.md#cliargkind)

***

### CreditLevel

Re-exports [CreditLevel](abi.md#creditlevel)

***

### CreditRole

Re-exports [CreditRole](abi.md#creditrole)

***

### DbResultSelection

Re-exports [DbResultSelection](abi.md#dbresultselection)

***

### DbStatementKind

Re-exports [DbStatementKind](abi.md#dbstatementkind)

***

### DbType

Re-exports [DbType](abi.md#dbtype)

***

### Entrypoint

Re-exports [Entrypoint](abi.md#entrypoint)

***

### ExpressionKind

Re-exports [ExpressionKind](abi.md#expressionkind)

***

### FetchedFileRole

Re-exports [FetchedFileRole](abi.md#fetchedfilerole)

***

### IdentifierScheme

Re-exports [IdentifierScheme](abi.md#identifierscheme)

***

### IntegerArithKind

Re-exports [IntegerArithKind](abi.md#integerarithkind)

***

### IsolationReq

Re-exports [IsolationReq](abi.md#isolationreq)

***

### MediaKind

Re-exports [MediaKind](abi.md#mediakind)

***

### PluginErrorCode

Re-exports [PluginErrorCode](abi.md#pluginerrorcode)

***

### PortalAuthMode

Re-exports [PortalAuthMode](abi.md#portalauthmode)

***

### ResolvedSqlType

Re-exports [ResolvedSqlType](abi.md#resolvedsqltype)

***

### WorkKind

Re-exports [WorkKind](abi.md#workkind)
