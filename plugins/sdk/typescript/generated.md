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

### AiEmbedText

One text to embed.

#### Properties

##### role

```ts
role: "document" | "query";
```

Document or query.

##### text

```ts
text: string;
```

The text, at most `maxAiEmbedTextBytes`; never empty.

***

### AiInput

Input to one model run: one field per `AiTask`, and exactly one is set
(the others are null). A new task appends a field.

#### Properties

##### textEmbeddings

```ts
textEmbeddings: AiTextEmbeddingsInput;
```

`textEmbeddings` task.

***

### AiModel

One model an `ai` plugin serves.

#### Properties

##### dimensions?

```ts
dimensions?: number;
```

Text embeddings: numbers per vector, at most `maxAiEmbedDimensions`.
Omitted when absent (wire zero value).

##### languages

```ts
languages: string[];
```

BCP 47 tags of the languages the model is loaded for, at most
`maxAiLanguages`; empty when it has no language packs.

##### maxTokens?

```ts
maxTokens?: number;
```

Text embeddings: tokens read from one text; the rest is cut off.
Omitted when absent (wire zero value).

##### modelId

```ts
modelId: string;
```

Model id within the plugin (`bekko-a8m`): lowercase letters, digits,
`.`, `_` and `-`. Callers name it `@<plugin id>/<model id>`; the whole
name is at most `maxAiModelNameBytes`.

##### revision

```ts
revision: string;
```

Digest of everything that decides the output: weights, tokenizer and
the language packs loaded. Outputs are comparable only when it matches.

##### task

```ts
task: "textEmbeddings";
```

The model's one task.

***

### AiModelList

Success payload of `AiModels.models`.

#### Properties

##### models

```ts
models: AiModel[];
```

Served models, at most `maxAiModels`, each `modelId` once.

***

### AiModels

`ai` entrypoint: serves models. The plugin lists the models it serves,
each with one task, and runs one model on one input. The host routes
every call (its own and other plugins' through the `AI` binding) to the
deployed plugin that serves the named model; a model plugin never learns
who asked. Bulk work arrives as many `run` calls from a host job, each
within the per-call caps.

#### Methods

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe: model files present and loadable.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### models()

```ts
models(): Promise<AiModelsReply>;
```

The models this plugin serves right now, under its current settings.

###### Returns

`Promise`\<[`AiModelsReply`](#aimodelsreply)\>

[AiModelsReply](#aimodelsreply)

##### run()

```ts
run(params): Promise<AiRunReply>;
```

Run one model on one input.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`AiRunParams`](#airunparams) | Model name and the task's input. |

###### Returns

`Promise`\<[`AiRunReply`](#airunreply)\>

[AiRunReply](#airunreply)

***

### AiOutput

Output of one model run: one field per `AiTask`, and exactly one is set
(the others are null). A new task appends a field.

#### Properties

##### textEmbeddings

```ts
textEmbeddings: AiTextEmbeddingsOutput;
```

`textEmbeddings` task.

***

### AiRunner

`AI` binding: runs a model by name, as `env.AI.run(model, input)` does on
Cloudflare Workers. The host checks the caller's `[ai]` grant for the
model's task, applies the call's deadline and the caller's budget, and
forwards to the serving plugin's `AiModels.run`.

#### Methods

##### run()

```ts
run(params): Promise<AiRunReply>;
```

Run one model on one input.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`AiRunParams`](#airunparams) | Model name or task alias, and the task's input. |

###### Returns

`Promise`\<[`AiRunReply`](#airunreply)\>

[AiRunReply](#airunreply)

***

### AiRunOk

Success payload of a model run.

#### Properties

##### model?

```ts
model?: string;
```

Full name of the model that ran (`@<plugin id>/<model id>`), so a
caller that used a task alias knows what answered. Set by the host on
`AiRunner.run`; empty on `AiModels.run`.
Omitted when absent (wire zero value).

##### output

```ts
output: AiOutput;
```

The output; the field set matches the model's task.

***

### AiRunParams

Params of `AiModels.run` and `AiRunner.run`.

#### Properties

##### input

```ts
input: AiInput;
```

The input; the field set must match the model's task.

##### model

```ts
model: string;
```

On `AiModels.run`, the plugin's own `AiModel.modelId`. On
`AiRunner.run`, `@<plugin id>/<model id>` or a task alias
(`@library/text-embeddings`: the model the library chose for the task).

***

### AiTextEmbeddingsInput

Input of the `textEmbeddings` task.

#### Properties

##### texts

```ts
texts: AiEmbedText[];
```

Texts to embed, 1 to `maxAiEmbedTexts`.

***

### AiTextEmbeddingsOutput

Output of the `textEmbeddings` task.

#### Properties

##### dimensions

```ts
dimensions: number;
```

Numbers per vector; equals the model's `AiModel.dimensions`.

##### fallbacks

```ts
fallbacks: number[];
```

Indexes (into the input texts, ascending, each once) of the texts
where the tokenizer met text outside the loaded language packs and
fell back to smaller pieces; such a vector should be recomputed after
the missing pack is added.

##### revision

```ts
revision: string;
```

`AiModel.revision` of the model that made the vectors.

##### vectors

```ts
vectors: Uint8Array;
```

One vector per input text, in input order: `dimensions` little-endian
IEEE 754 32-bit floats each, concatenated. Every number is finite.

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

##### ai?

```ts
ai?: AiRunner | null;
```

`AI`: runs models other plugins serve; null unless `[ai]` names the
tasks the plugin calls and the operator granted them.
Null when the guest does not export this capability.

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

##### network

```ts
network: NetworkGrant;
```

`NETWORK`: what the operator granted this plugin's network, including
the destination each network setting names and whether it is approved,
so a guest can tell an unapproved destination from an unreachable one.

##### notify?

```ts
notify?: NotificationRaiser | null;
```

`NOTIFY`: raises the notification types the plugin declares in
`[[notifications.types]]`; null when it declares none.
Null when the guest does not export this capability.

##### origin?

```ts
origin?: Origin | null;
```

`ORIGIN`: the library itself, for a relay `ingress` plugin to send the
public requests it carries to; null unless the plugin is the library's
active relay ingress.
Null when the guest does not export this capability.

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

##### sockets?

```ts
sockets?: Sockets | null;
```

`SOCKETS`: host-brokered outbound WebSockets that stay open while the
guest sleeps; null unless `[sockets]` is declared and granted.
Null when the guest does not export this capability.

##### storage?

```ts
storage?: Destination | null;
```

`WORK_FS`: host-granted object storage for durable plugin files (work
filesystem); null unless `[work_fs]` is granted. Job input/output travel on
`JobRunner.job(controller)`, never here.
Null when the guest does not export this capability.

##### webhooks?

```ts
webhooks?: Webhooks | null;
```

`WEBHOOKS`: the instance's inbound webhooks, their URLs and the
deliveries the host stored; null unless `[[webhooks]]` is declared and
granted.
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

##### detailByProductId

```ts
detailByProductId: boolean;
```

True when `catalogDetail` resolves this store's own `productId` (an id
its search, expand or deals hits gave) to a full bibliographic record.
Catalog search then asks this store to fill lean result rows it listed
itself, and title dialogs ask it for the editions it listed. A store
that leaves it false is never woken for a detail lookup by product id.

##### priceByAsin

```ts
priceByAsin: boolean;
```

True when `purchaseHint` / `purchaseHints` price a query by its exact
ASIN (`lookup = asin`), as `priceByIsbn` does for ISBNs.

##### priceByIsbn

```ts
priceByIsbn: boolean;
```

True when `purchaseHint` / `purchaseHints` price a query by its exact
ISBN (`lookup = isbn`): the answer is the store's listing of that ISBN
or not found, never a title match. The host then prices a card at this
store by the ISBNs of the card's recording before searching by title.

##### qualityOffers

```ts
qualityOffers: boolean;
```

True when `qualityOffers` answers which audio quality an account would
download for a title. The host's quality-upgrade check asks only stores
that declare it.

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

##### explicitness

```ts
explicitness: "unknown" | "clean" | "explicit";
```

Whether the storefront marks the edition explicit (adult) content.

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

##### sampleUrl?

```ts
sampleUrl?: string;
```

Absolute `https://` URL of a short audio sample of this edition that
the storefront offers publicly (a few minutes, in any format the media
worker decodes). The host never hands it to a client: a book's details
play it through a same-origin path, fetched by a `mediaDownload` plugin with
`media = audio` and re-encoded in the media worker. Empty when the store
has none.
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

### ChangesPage

One page of `Destination.changes`.

#### Properties

##### changes

```ts
changes: ObjectChange[];
```

Changes since the request's cursor, oldest first. A key may repeat.

##### complete

```ts
complete: boolean;
```

False when the feed may have missed changes since the request's cursor
(an overflowed watcher, expired queue retention, a restart): the host
backs it with a full listing. True means these are all the changes.

##### cursor

```ts
cursor: string;
```

Cursor for the next call; never empty.

##### liveWatch

```ts
liveWatch: boolean;
```

True when this running guest produces the feed itself (a filesystem
watcher) and stopping it would lose changes, so the host keeps it awake
while `complete` stays true. Durable feeds (a notification queue) leave
it false. The host honours it only from the first-party storage plugins
it starts on demand.

##### more

```ts
more: boolean;
```

True when more changes are ready now; call again with `cursor`.

***

### ChangesRequest

Argument of `Destination.changes`. Fields are only ever appended.

#### Properties

##### cursor

```ts
cursor: string;
```

`ChangesPage.cursor` from the previous call. Empty asks for the current
position: the page carries a cursor and no changes.

##### limit

```ts
limit: number;
```

Requested changes per page, clamped to `maxChangesPage`; zero means
guest default.

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

##### qualityOffers()

```ts
qualityOffers(params): Promise<QualityOffersReply>;
```

The audio quality one account would download for each of several
titles today, so the host can tell when a better file than the one it
holds is on offer. Asked only of guests that declare
`CatalogCapabilities.qualityOffers`; it must not license or download.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`QualityOffersParams`](#qualityoffersparams) | Account, credentials, and title ids. |

###### Returns

`Promise`\<[`QualityOffersReply`](#qualityoffersreply)\>

[QualityOffersReply](#qualityoffersreply)

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

### DeleteManyResult

Result of `Destination.deleteMany`.

#### Properties

##### results

```ts
results: KeyOutcome[];
```

One outcome per requested key, in request order. A missing key succeeds.

***

### DeprovisionRequest

Arguments of `Provisioning.deprovision`.

#### Properties

##### grant

```ts
grant: ProvisionGrant;
```

The operator's grant, or kind `none` to use the stored secrets.

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

##### changes()

```ts
changes(request): Promise<ChangesReply>;
```

Keys changed since a cursor (requires `storage.changes`). A hint for
what to read; a scheduled listing stays the backstop.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ChangesRequest`](#changesrequest) | Cursor and page size. |

###### Returns

`Promise`\<[`ChangesReply`](#changesreply)\>

[ChangesReply](#changesreply)

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

##### deleteMany()

```ts
deleteMany(keys): Promise<DeleteManyReply>;
```

Remove up to `maxBatchKeys` objects (requires `storage.deleteMany`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `keys` | `string`[] | Object keys to remove. |

###### Returns

`Promise`\<[`DeleteManyReply`](#deletemanyreply)\>

[DeleteManyReply](#deletemanyreply)

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

##### headMany()

```ts
headMany(keys): Promise<HeadManyReply>;
```

Probe up to `maxBatchKeys` objects (requires `storage.headMany`).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `keys` | `string`[] | Object keys to probe. |

###### Returns

`Promise`\<[`HeadManyReply`](#headmanyreply)\>

[HeadManyReply](#headmanyreply)

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

##### move()

```ts
move(
   from, 
   to, 
   options
): Promise<MoveReply>;
```

Move one object (requires `storage.move`). Repeating a finished move
succeeds with `alreadyMoved`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `from` | `string` | Source object key. |
| `to` | `string` | Target object key. |
| `options` | [`MoveOptions`](#moveoptions) | Overwrite and source preconditions. |

###### Returns

`Promise`\<[`MoveReply`](#movereply)\>

[MoveReply](#movereply)

##### moveMany()

```ts
moveMany(request): Promise<MoveManyReply>;
```

Move a set of objects or a whole prefix (requires `storage.moveMany`).
Paged and safe to repeat.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`MoveManyRequest`](#movemanyrequest) | Pairs or prefix, cursor, and options. |

###### Returns

`Promise`\<[`MoveManyReply`](#movemanyreply)\>

[MoveManyReply](#movemanyreply)

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

##### translateKeys()

```ts
translateKeys(keys): Promise<TranslateKeysReply>;
```

Translate up to `maxBatchKeys` host keys into the keys the guest stores
them under (requires `storage.translateKeys`). A host key is `/`-joined
segments, none empty, `.`, `..` or holding `` or a control character.
The same host key must always translate to the same key. Without the
feature the host stores each host key as written.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `keys` | `string`[] | Host keys to translate. |

###### Returns

`Promise`\<[`TranslateKeysReply`](#translatekeysreply)\>

[TranslateKeysReply](#translatekeysreply)

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

### DownloadCondition

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

### DownloadResult

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

### DownloadResults

Success payload of `MediaDownloader.downloadMedia`.

#### Properties

##### downloads

```ts
downloads: DownloadResult[];
```

One entry per requested URL, in request order.

***

### Entrypoints

Exported entrypoints returned by `PluginWorker.open`. One capability per
`plugin.toml` `entrypoints` entry / trigger; null when not exported. The host
refuses an entrypoint the manifest or operator grant did not allow.

#### Properties

##### ai?

```ts
ai?: AiModels | null;
```

`ai` entrypoint.
Null when the guest does not export this capability.

##### cli?

```ts
cli?: PluginCli | null;
```

`cli` entrypoint.
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

##### ingress?

```ts
ingress?: Ingress | null;
```

`ingress` entrypoint.
Null when the guest does not export this capability.

##### jobRunner?

```ts
jobRunner?: JobRunner | null;
```

`[triggers] jobs` trigger: `job(controller)` handler.
Null when the guest does not export this capability.

##### mediaDownload?

```ts
mediaDownload?: MediaDownloader | null;
```

`mediaDownload` entrypoint.
Null when the guest does not export this capability.

##### metadata?

```ts
metadata?: MetadataProvider | null;
```

`metadata` entrypoint.
Null when the guest does not export this capability.

##### metadataFiles?

```ts
metadataFiles?: MetadataFiles | null;
```

`metadataFiles` entrypoint.
Null when the guest does not export this capability.

##### notifier?

```ts
notifier?: Notifier | null;
```

`notifier` entrypoint.
Null when the guest does not export this capability.

##### oidc?

```ts
oidc?: Oidc | null;
```

`oidc` entrypoint.
Null when the guest does not export this capability.

##### provisioning?

```ts
provisioning?: Provisioning | null;
```

`provisioning` entrypoint: guided setup.
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

##### socketHandler?

```ts
socketHandler?: SocketHandler | null;
```

`[sockets]` handler: messages and closes on host-brokered sockets.
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

##### webhookHandler?

```ts
webhookHandler?: WebhookHandler | null;
```

`[[webhooks]]` handler: called after the host stores deliveries.
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

### EventAckParams

Params of `RemoteLibrary.ackEvents`.

#### Properties

##### externalUserIds

```ts
externalUserIds: string[];
```

`ExternalUser.externalUserId` of each user the host passed on.

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

### EventPollParams

Params of `RemoteLibrary.pollEvents`.

#### Properties

##### ackFollows

```ts
ackFollows: boolean;
```

The host calls `ackEvents` with every user it passes on, so the guest
keeps returning a user until it is acknowledged. False from hosts that
predate `ackEvents`: the guest then treats a returned user as delivered.

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
role: 
  | "unspecified"
  | "part"
  | "whole"
  | "cover"
  | "catalogRecord"
  | "annotations";
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

When true, hand back the account's clips and bookmarks for the title as
an `annotations` file when the store keeps them.

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

Retired: hosts always send false and guests ignore it. A kept encrypted
download plays only with its keys, which the host does not store.

##### saveMetadataJson

```ts
saveMetadataJson: boolean;
```

When true, hand back the store's catalog record of the title as a
`catalogRecord` file.

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

### HeadManyResult

Result of `Destination.headMany`.

#### Properties

##### objects

```ts
objects: HeadOk[];
```

One probe per requested key, in request order.

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

### Ingress

A public address provider (`Entrypoints.ingress`): the one `ingress`
family instance the library runs. A relay provider carries traffic to
`Bindings.origin`; a direct provider keeps a DNS name pointing at this
install.

#### Methods

##### health()

```ts
health(): Promise<IngressHealthReply>;
```

Whether the public address is serving now.

###### Returns

`Promise`\<[`IngressHealthReply`](#ingresshealthreply)\>

[IngressHealthReply](#ingresshealthreply)

##### start()

```ts
start(): Promise<IngressStartReply>;
```

Bring the public address up; answers the origin it serves.

###### Returns

`Promise`\<[`IngressStartReply`](#ingressstartreply)\>

[IngressStartReply](#ingressstartreply)

##### stop()

```ts
stop(): Promise<EmptyReply>;
```

Take the public address down; stopping a stopped one is not an error.

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

***

### IngressHealth

Success payload of `Ingress.health`.

#### Properties

##### detail?

```ts
detail?: string;
```

Why it is not serving, for the operator; empty when serving.
Omitted when absent (wire zero value).

##### serving

```ts
serving: boolean;
```

True when the address is serving.

***

### IngressStarted

Success payload of `Ingress.start`.

#### Properties

##### origin

```ts
origin: string;
```

The public origin now serving: `https://` and a host name, no port, path
or trailing slash, at most `maxIngressOriginBytes` after the scheme.

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

##### purpose

```ts
purpose: "serve" | "provision";
```

What the host opens the entrypoints for. `provision` opens an instance
that may not be configured yet: `CONFIG` and `SECRETS` hold what is
stored so far, required settings may be missing, and the host calls
only `Entrypoints.provisioning`.

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

### KeyOutcome

Outcome for one key of a batch call.

#### Properties

##### error

```ts
error: PluginError;
```

Failure for this key; an empty `code` means it succeeded.

##### key

```ts
key: string;
```

The key this outcome is for (the source key for moves).

***

### KeyPair

One source and target key.

#### Properties

##### from

```ts
from: string;
```

Source object key (or source prefix).

##### to

```ts
to: string;
```

Target object key (or target prefix).

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
the library, so an instance moved to another host reads its own keys.

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

##### modifiedSinceMs

```ts
modifiedSinceMs: number;
```

Hint: only objects modified at or after this time (UTC unix
milliseconds); zero lists everything. `ListPage.modifiedSinceApplied`
says whether the guest filtered; when it did not, the page is unfiltered.

##### prefix

```ts
prefix: string;
```

Only keys starting with this prefix; empty lists everything.

##### withSignatures

```ts
withSignatures: boolean;
```

Hint: fill `etag`, `modifiedAtMs`, `versionId` and `signature` on each
entry. `ListPage.signaturesIncluded` says whether the guest did.

***

### ListPage

One page of `Destination.list` results.

#### Properties

##### modifiedSinceApplied

```ts
modifiedSinceApplied: boolean;
```

True when the guest left out objects modified before
`ListOptions.modifiedSinceMs`. False means the page is unfiltered.

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

##### signaturesIncluded

```ts
signaturesIncluded: boolean;
```

True when every entry carries the signature fields the backend has
(`ListOptions.withSignatures` honoured).

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

### MediaDownloader

`mediaDownload` entrypoint: downloads public media (cover images and
storefronts' audio samples) on the host's behalf, so the host itself never
makes an outbound request to a URL a plugin or a store supplied. The
downloader only transports bytes: the host sniffs, decodes and re-encodes
every body in its media worker before anything stores or serves it. Network access is limited to what the manifest's
`[capabilities.network]` declares.

#### Methods

##### downloadMedia()

```ts
downloadMedia(params, sink): Promise<DownloadResultsReply>;
```

Fetches every URL concurrently and streams each body into `sink` under
its index in `params.urls` (`"0"`, `"1"`, ...). The reply says, per URL,
whether its body was stored.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`MediaDownloadParams`](#mediadownloadparams) | URLs, media kind and the per-body byte cap. |
| `sink` | [`Destination`](#destination) | Host-served store that accepts only those keys. |

###### Returns

`Promise`\<[`DownloadResultsReply`](#downloadresultsreply)\>

[DownloadResultsReply](#downloadresultsreply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

***

### MediaDownloadParams

Params of `MediaDownloader.downloadMedia`.

#### Properties

##### conditions

```ts
conditions: DownloadCondition[];
```

Conditional request headers per URL, in the same order as `urls`, from
the host's cached copy. Empty, or shorter than `urls`, means the
remaining URLs are fetched unconditionally.

##### maxBytes

```ts
maxBytes: number;
```

Largest body, in bytes, the host accepts for one URL; at most
`maxImageBytes` for images and `maxAudioBytes` for audio. The sink refuses a larger body, which fails that URL
only.

##### media

```ts
media: "audio" | "image";
```

What the URLs name: cover art (the default) or a storefront's audio
sample (`CatalogHit.sampleUrl`). A fetcher sends a matching `Accept`
header and refuses a body whose `Content-Type` names something else;
the host checks and re-encodes either kind regardless.

##### urls

```ts
urls: string[];
```

Absolute `https://` URLs to fetch, at most `maxDownloadUrls`; never empty.
The host has already checked each one against its URL policy.

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

### MetadataAudioFile

One audio file of the edition a sidecar describes, for formats that name
their audio (a CUE sheet's `FILE` lines).

#### Properties

##### durationMs

```ts
durationMs: number;
```

Running time in milliseconds; 0 when the host does not know it.

##### name

```ts
name: string;
```

The file's name without any folder, as it sits next to the sidecar.

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

### MetadataFileParsed

Success payload of `MetadataFiles.parse`.

#### Properties

##### record

```ts
record: MetadataFileRecord;
```

What the file says.

***

### MetadataFileParseParams

Params of `MetadataFiles.parse`.

#### Properties

##### bytes

```ts
bytes: Uint8Array;
```

The whole file; at most `maxMetadataFileBytes`.

##### dialect

```ts
dialect: string;
```

Dialect id from the manifest's `[[metadata_files.dialects]]` that the
host routed the file to.

##### fileName

```ts
fileName: string;
```

The file's name without any folder (`metadata.json`, `Book.opf`).

***

### MetadataFileRecord

What one metadata file says about the edition it sits beside. A field the
file does not state is left out of `present` and means "unknown", never
"cleared"; a field in `present` with an empty value means the file states
that it is empty. The host checks every value against the `maxMetadataFile*`
caps and refuses a record that states a field its dialect does not carry.

#### Properties

##### abridgement

```ts
abridgement: "unknown" | "unabridged" | "abridged";
```

Whether the edition is abridged; `unknown` when the file does not say.

##### chapters

```ts
chapters: MetadataChapter[];
```

Chapter layout in pre-order (see `MetadataChapter`); at most
`maxMetadataFileChapters`, starts never decreasing.

##### creditRoles

```ts
creditRoles: (
  | "author"
  | "narrator"
  | "translator"
  | "editor"
  | "illustrator"
  | "publisher"
  | "host"
  | "cast"
  | "adaptor")[];
```

Credit roles the file states, each at most once. A role listed here
with no credit means the file states nobody in that role; a role left
out is unknown. Empty unless `present` holds `credits`.

##### credits

```ts
credits: Credit[];
```

Credited contributors in billing order, only in the roles `creditRoles`
names.

##### description?

```ts
description?: string;
```

Long description / blurb; at most `maxMetadataFileDescriptionBytes`.
Omitted when absent (wire zero value).

##### durationMs?

```ts
durationMs?: number;
```

Running time in milliseconds.
Omitted when absent (wire zero value).

##### explicitness

```ts
explicitness: "unknown" | "clean" | "explicit";
```

Whether the edition is explicit (adult) content; `unknown` when the
file does not say.

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

How the work is realised; `unspecified` when the file does not say.

##### genres

```ts
genres: string[];
```

Genre labels in file order.

##### identifiers

```ts
identifiers: Identifier[];
```

External identifiers the file carries.

##### language?

```ts
language?: string;
```

Content language as the file prints it (a code or a name).
Omitted when absent (wire zero value).

##### present

```ts
present: (
  | "publisher"
  | "chapters"
  | "title"
  | "series"
  | "subtitle"
  | "description"
  | "publishedAt"
  | "language"
  | "genres"
  | "tags"
  | "abridgement"
  | "expressionKind"
  | "identifiers"
  | "credits"
  | "duration"
  | "explicitness")[];
```

Fields the file states, each at most once; at most the fields the
dialect declares.

##### publishedAt?

```ts
publishedAt?: string;
```

Publication date as `YYYY`, `YYYY-MM` or `YYYY-MM-DD`.
Omitted when absent (wire zero value).

##### publisher?

```ts
publisher?: string;
```

Publisher name.
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

##### tags

```ts
tags: string[];
```

Free tags in file order.

##### title?

```ts
title?: string;
```

Primary title.
Omitted when absent (wire zero value).

***

### MetadataFileRendered

Success payload of `MetadataFiles.render`.

#### Properties

##### bytes

```ts
bytes: Uint8Array;
```

The new file; at most `maxMetadataFileBytes`. Empty when the dialect has
nothing left to write; the host decides whether to remove the file.

***

### MetadataFileRenderParams

Params of `MetadataFiles.render`.

#### Properties

##### audioFiles

```ts
audioFiles: MetadataAudioFile[];
```

The edition's audio files in playback order, at most
`maxMetadataAudioFiles`; empty when the host has none or does not know
them. Chapter starts in `record` count from the start of the first file.

##### dialect

```ts
dialect: string;
```

Dialect id from the manifest; one that declares `writes = true`.

##### existing

```ts
existing: Uint8Array;
```

The file as it is now; empty when the host is creating it. Everything
the dialect does not change (unknown keys, comments, other fields) is
kept. At most `maxMetadataFileBytes`.

##### fields

```ts
fields: (
  | "publisher"
  | "chapters"
  | "title"
  | "series"
  | "subtitle"
  | "description"
  | "publishedAt"
  | "language"
  | "genres"
  | "tags"
  | "abridgement"
  | "expressionKind"
  | "identifiers"
  | "credits"
  | "duration"
  | "explicitness")[];
```

Fields to set, each at most once and all declared by the dialect. A
listed field with an empty value removes it from the file.

##### record

```ts
record: MetadataFileRecord;
```

Values to write. Only the fields in `fields` are read from it.

***

### MetadataFiles

`metadataFiles` entrypoint: reads and writes metadata files that sit next
to an edition's media (sidecars such as `metadata.json`, OPF, NFO or CUE),
one dialect per format. The plugin only converts bytes: the host reads the
file, chooses every path and writes the result, so a dialect plugin needs
no network or file grants. The manifest's `[metadata_files]` table names
each dialect, the file names it reads and the fields it carries, so the
host routes a file to a sleeping plugin without starting it.

#### Methods

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### parse()

```ts
parse(params): Promise<MetadataFileParseReply>;
```

What one metadata file says about its edition.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`MetadataFileParseParams`](#metadatafileparseparams) | Dialect, file name and bytes. |

###### Returns

`Promise`\<[`MetadataFileParseReply`](#metadatafileparsereply)\>

[MetadataFileParseReply](#metadatafileparsereply)

##### render()

```ts
render(params): Promise<MetadataFileRenderReply>;
```

The bytes of a metadata file with the given fields set to the record's
values and everything else kept from the existing file.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`MetadataFileRenderParams`](#metadatafilerenderparams) | Dialect, record, fields and existing bytes. |

###### Returns

`Promise`\<[`MetadataFileRenderReply`](#metadatafilerenderreply)\>

[MetadataFileRenderReply](#metadatafilerenderreply)

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

### MoveManyPage

One page of `Destination.moveMany` progress.

#### Properties

##### atomic

```ts
atomic: boolean;
```

True when every move in this page was atomic.

##### nextCursor

```ts
nextCursor: string;
```

Prefix form: cursor for the next call; empty when every object moved.

##### objectsMoved

```ts
objectsMoved: number;
```

Objects moved in this page (a folder rename counts what it carried
when known, else the outcomes).

##### results

```ts
results: KeyOutcome[];
```

Outcomes in request order. A prefix move done in one step (a folder
rename) may report a single outcome keyed by the source prefix.

***

### MoveManyRequest

Argument of `Destination.moveMany`. Fields are only ever appended.

#### Properties

##### cursor

```ts
cursor: string;
```

Prefix form: `MoveManyPage.nextCursor` from the previous call; empty
starts. Ignored for `pairs`.

##### limit

```ts
limit: number;
```

Prefix form: objects to move in this call, clamped to `maxBatchKeys`;
zero means guest default.

##### options

```ts
options: MoveOptions;
```

Overwrite rule applied to every move (`ifSourceMatch` is ignored).

##### target

```ts
target: MoveManyTarget;
```

Explicit pairs or a prefix.

***

### MoveOptions

Options for `Destination.move` and `Destination.moveMany`. Fields are only
ever appended.

#### Properties

##### ifSourceMatch

```ts
ifSourceMatch: string;
```

`move` only: move only while the source's `etag` equals this value,
else fail with `conflict`. Empty skips the check.

##### noOverwrite

```ts
noOverwrite: boolean;
```

Fail with `conflict` instead of replacing an object at the target key.

***

### MoveResult

Summary of `Destination.move`.

#### Properties

##### alreadyMoved

```ts
alreadyMoved: boolean;
```

True when the source was already gone and the target present, as when
a move is repeated after a restart. The move counts as done.

##### atomic

```ts
atomic: boolean;
```

True when the backend moved atomically (a rename): no reader saw the
object at both keys or at neither.

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

### NetworkGrant

The network grant in effect for one `PluginWorker.open`. Read-only
feedback: the host enforces the grant whatever the guest does with this.

#### Properties

##### domains

```ts
domains: string[];
```

Fetch destinations in effect, as `plugin.toml` `domains` entries
(`host`, `*.host`, `host:port`, or `*` for any public host), including
approved setting destinations; at most `maxNetworkGrantEntries`.

##### outbound

```ts
outbound: boolean;
```

True when the grant allows outbound connections; false for `deny`.

##### settings

```ts
settings: SettingNetworkGrant[];
```

One entry per `[[settings]]` entry that declares `network`, in
declaration order; at most `maxNetworkGrantEntries`.

##### tcp

```ts
tcp: string[];
```

Raw TCP destinations in effect, as `host:port` (IPv6 in brackets), one
per granted port; at most `maxNetworkGrantEntries`.

***

### NotificationField

One labelled value shown with a notification.

#### Properties

##### name

```ts
name: string;
```

Label; at most `maxNotificationTitleBytes`.

##### value

```ts
value: string;
```

Value; at most `maxNotificationFieldBytes`.

***

### NotificationMessage

A message the host rendered for one notification. Every link is built
from the library's public origin, never from a request.

#### Properties

##### alertAction

```ts
alertAction: "none" | "trigger" | "resolve";
```

Alert lifecycle step; `none` for a one-off message.

##### alertKey?

```ts
alertKey?: string;
```

Alert this message opens or closes; required unless `alertAction` is
`none`.
Omitted when absent (wire zero value).

##### fields

```ts
fields: NotificationField[];
```

Labelled values, at most `maxNotificationFields`.

##### groupKey?

```ts
groupKey?: string;
```

Messages with one group key may be threaded or collapsed together.
Omitted when absent (wire zero value).

##### html?

```ts
html?: string;
```

Minimal HTML body for email; at most `maxNotificationHtmlBytes`.
Omitted when absent (wire zero value).

##### libraryOrigin?

```ts
libraryOrigin?: string;
```

The library's home page (`https://library.example.com/`), built from its
public origin like `link`; absent when no public origin is set. Email
footers link to it.
Omitted when absent (wire zero value).

##### link?

```ts
link?: string;
```

Absolute `https` (or loopback `http`) link into the library.
Omitted when absent (wire zero value).

##### markdown?

```ts
markdown?: string;
```

Markdown body for chat services; at most `maxNotificationTextBytes`.
Omitted when absent (wire zero value).

##### notificationId

```ts
notificationId: string;
```

Host-assigned id, the same on every destination's copy; plugins use it
as an idempotency key where the service takes one.

##### notificationType

```ts
notificationType: string;
```

Notification type id (`job_failed`, `<plugin>:sync_failed`).

##### sensitive

```ts
sensitive: boolean;
```

True when the body carries a secret (an account-mail link): the plugin
and SDKs must never log it.

##### severity

```ts
severity: "info" | "warning" | "critical";
```

How urgent it is.

##### text

```ts
text: string;
```

Plain-text body; at most `maxNotificationTextBytes`.

##### title

```ts
title: string;
```

Short title; at most `maxNotificationTitleBytes`.

***

### NotificationRaiser

`NOTIFY` binding: raises a notification type the plugin declared in
`[[notifications.types]]`. The host router applies the administrators'
routing and each patron's preferences; the raising plugin never learns
where the notification went.

#### Methods

##### raise()

```ts
raise(params): Promise<NotifyRaiseReply>;
```

Raise one notification.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`NotifyRaiseParams`](#notifyraiseparams) | Declared type, fields and optional patron. |

###### Returns

`Promise`\<[`NotifyRaiseReply`](#notifyraisereply)\>

[NotifyRaiseReply](#notifyraisereply)

***

### Notifier

`notifier` entrypoint: delivers one host-rendered message to one
destination (an email address, a chat webhook, a push subscription). The
host decides what is worth a notification, who gets it and which instance
carries it; the plugin only sends. Each send is one claimed `notify.send`
job, so the host owns retries: a reply of `rate_limited`, `unavailable`,
`deadline_exceeded` or `cancelled` is retried (`rate_limited` honouring
`details.retryAfterMs`), any other error is a permanent failure for that
destination.

#### Methods

##### checkDeliverability()

```ts
checkDeliverability(params): Promise<NotifyDeliverabilityReply>;
```

Operator's deliverability check: send a sample the way real messages go,
ask checker services how it scores, and merge their findings into
advice. Only plugins that declare `[notifier].deliverability_check`
implement it (others answer `unsupported`); it may run up to
`deliverabilityCheckDeadlineMs`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`NotifyDeliverabilityParams`](#notifydeliverabilityparams) | Sample message to check. |

###### Returns

`Promise`\<[`NotifyDeliverabilityReply`](#notifydeliverabilityreply)\>

[NotifyDeliverabilityReply](#notifydeliverabilityreply)

##### health()

```ts
health(): Promise<HealthReply>;
```

Liveness / readiness probe: credentials and reachability.

###### Returns

`Promise`\<[`HealthReply`](#healthreply)\>

[HealthReply](#healthreply)

##### send()

```ts
send(params): Promise<NotifySendReply>;
```

Deliver one message to one destination.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`NotifySendParams`](#notifysendparams) | Message and destination. |

###### Returns

`Promise`\<[`NotifySendReply`](#notifysendreply)\>

[NotifySendReply](#notifysendreply)

##### validateDestination()

```ts
validateDestination(params): Promise<NotifyValidateReply>;
```

Normalise and check a personal destination's address before the host
saves it.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`NotifyValidateParams`](#notifyvalidateparams) | Address document to check. |

###### Returns

`Promise`\<[`NotifyValidateReply`](#notifyvalidatereply)\>

[NotifyValidateReply](#notifyvalidatereply)

***

### NotifyDeliverabilityAdvice

One thing the operator can do, merged from every check.

#### Properties

##### detail

```ts
detail: string;
```

What to change and where; at most `maxNotificationFieldBytes`.

##### outcome

```ts
outcome: "warning" | "passed" | "failed" | "skipped";
```

`failed` is a fix, `warning` an improvement, `passed` what is already
right; never `skipped`.

##### sources

```ts
sources: string[];
```

The `service` names of the checks that raised it.

##### title

```ts
title: string;
```

Short heading; at most `maxNotificationTitleBytes`.

***

### NotifyDeliverabilityCheck

One checker's result.

#### Properties

##### outcome

```ts
outcome: "warning" | "passed" | "failed" | "skipped";
```

How it came out.

##### reportUrl?

```ts
reportUrl?: string;
```

Absolute `https` link to the service's full report, when it has one.
Omitted when absent (wire zero value).

##### service

```ts
service: string;
```

Service name (`Postmark spam check`); at most `maxNotificationTitleBytes`.

##### summary

```ts
summary: string;
```

One-line result (`SpamAssassin score 0.8; under 5 passes`); at most
`maxNotificationFieldBytes`.

***

### NotifyDeliverabilityParams

Params of `Notifier.checkDeliverability`.

#### Properties

##### message

```ts
message: NotificationMessage;
```

The host's `test` notification, rendered as a real one is, so the
checkers score the plugin's real output.

***

### NotifyDeliverabilityReport

Success payload of `Notifier.checkDeliverability`.

#### Properties

##### advice

```ts
advice: NotifyDeliverabilityAdvice[];
```

Merged advice, most urgent first; at most `maxDeliverabilityAdvice`.

##### checks

```ts
checks: NotifyDeliverabilityCheck[];
```

One entry per checker, at most `maxDeliverabilityChecks`.

***

### NotifyDestination

Where a `Notifier.send` goes.

#### Properties

##### addressJson

```ts
addressJson: string;
```

Personal address as a JSON object of the fields the manifest's
`[[notifier.address_fields]]` declare; empty for a channel. At most
`maxNotifierAddressBytes`. May hold secrets; never logged.

##### destinationId

```ts
destinationId: string;
```

Host id of the destination, stable across sends.

##### kind

```ts
kind: "channel" | "personal";
```

Channel or personal destination.

***

### NotifyRaised

Success payload of `NotificationRaiser.raise`.

#### Properties

##### notificationId

```ts
notificationId: string;
```

Host-assigned notification id.

***

### NotifyRaiseParams

Params of `NotificationRaiser.raise`.

#### Properties

##### alertAction

```ts
alertAction: "none" | "trigger" | "resolve";
```

Alert lifecycle step; `none` for a one-off notification.

##### alertKey?

```ts
alertKey?: string;
```

Alert this raise opens or closes; required unless `alertAction` is
`none`.
Omitted when absent (wire zero value).

##### fields

```ts
fields: NotificationField[];
```

Labelled values, at most `maxNotificationFields`.

##### groupKey?

```ts
groupKey?: string;
```

Messages with one group key may be threaded or collapsed together.
Omitted when absent (wire zero value).

##### linkPath?

```ts
linkPath?: string;
```

Path inside the library the notification links to (starts with `/`);
the host prefixes the public origin.
Omitted when absent (wire zero value).

##### notificationType

```ts
notificationType: string;
```

Type id from the plugin's `[[notifications.types]]` (without the plugin
prefix the host adds).

##### patronId?

```ts
patronId?: string;
```

Patron the notification is for; required for a patron-audience type
and must be inside the invocation's account scope; empty otherwise.
Omitted when absent (wire zero value).

***

### NotifySendParams

Params of `Notifier.send`.

#### Properties

##### destination

```ts
destination: NotifyDestination;
```

Where to send it.

##### message

```ts
message: NotificationMessage;
```

What to send.

***

### NotifySent

Success payload of `Notifier.send`.

#### Properties

##### providerId?

```ts
providerId?: string;
```

The service's id for the delivered message, when it gives one.
Omitted when absent (wire zero value).

***

### NotifyValidated

Success payload of `Notifier.validateDestination`.

#### Properties

##### addressJson

```ts
addressJson: string;
```

Normalised address document the host saves; at most
`maxNotifierAddressBytes`.

##### label?

```ts
label?: string;
```

Short label shown to the patron (`a***@example.com`); never a secret.
Omitted when absent (wire zero value).

***

### NotifyValidateParams

Params of `Notifier.validateDestination`.

#### Properties

##### addressJson

```ts
addressJson: string;
```

Address document as the patron entered it; at most
`maxNotifierAddressBytes`.

***

### ObjectChange

One entry of a change feed.

#### Properties

##### info

```ts
info: ObjectInfo;
```

Listing fields after the change when the feed carries them (signature
fields may be empty); empty for `removed`.

##### key

```ts
key: string;
```

Object key, relative to the destination root.

##### kind

```ts
kind: "changed" | "upserted" | "removed";
```

What happened to it.

***

### ObjectInfo

Compact object entry in a list page.

#### Properties

##### etag

```ts
etag: string;
```

Backend entity tag; empty when unsupported or not requested
(`ListOptions.withSignatures`).

##### key

```ts
key: string;
```

Object key.

##### modifiedAtMs

```ts
modifiedAtMs: number;
```

Last modification time, UTC unix milliseconds; zero when unknown or not
requested.

##### signature

```ts
signature: string;
```

Opaque change signature, as `ObjectMetadata.signature`; empty when
unsupported or not requested.

##### size

```ts
size: number;
```

Object size in bytes.

##### versionId

```ts
versionId: string;
```

Backend object version id; empty when none or not requested.

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

##### modifiedAtMs

```ts
modifiedAtMs: number;
```

Last modification time, UTC unix milliseconds; zero when unknown.

##### sha256

```ts
sha256: Uint8Array;
```

Raw SHA-256 digest (32 bytes) or empty when unknown.

##### signature

```ts
signature: string;
```

Opaque change signature: an unchanged value means the object has not
changed. Backend-defined (a local file's inode, size, mtime and ctime;
an S3 ETag and version id). Empty when the backend has none.

##### size

```ts
size: number;
```

Object size in bytes.

##### versionId

```ts
versionId: string;
```

Backend object version id (versioned buckets); empty when none.

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

### Origin

The library, as a relay `ingress` plugin reaches it (`Bindings.origin`,
`ORIGIN` in the Workers idiom). The plugin turns each request its provider
delivers into a `fetch` and streams the response back. The host treats
every request as the `tunnel` transport: public `https`, never loopback,
with the client address the plugin reports.

#### Methods

##### connect()

```ts
connect(request, attachment): Promise<OriginConnectReply>;
```

Open a WebSocket to the library for one upgrade request the provider
delivers. An accepted upgrade becomes a socket in the instance's
`Bindings.sockets`: its messages arrive on `Entrypoints.socketHandler`
and the plugin answers with `Sockets.send` and `Sockets.close`, as on
any socket it dialed. Needs `[sockets]`. These sockets count toward
`maxOriginSocketsPerInstance`, not `maxSocketsPerInstance`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`OriginRequest`](#originrequest) | A `GET` with `upgrade: websocket`. |
| `attachment` | `Uint8Array` | Initial attachment, at most `maxSocketAttachmentBytes`. |

###### Returns

`Promise`\<[`OriginConnectReply`](#originconnectreply)\>

[OriginConnectReply](#originconnectreply)

##### fetch()

```ts
fetch(request, body): Promise<OriginFetchReply>;
```

Serve one request; the response body streams as the library writes it.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`OriginRequest`](#originrequest) | Method, target, headers, client address. |
| `body` | [`ByteSource`](#bytesource) | Request body; null when the request has none. |

###### Returns

`Promise`\<[`OriginFetchReply`](#originfetchreply)\>

[OriginFetchReply](#originfetchreply)

***

### OriginHeader

One header field on an `Origin.fetch` request or response.

#### Properties

##### name

```ts
name: string;
```

Field name, an HTTP token; the host refuses hop-by-hop names.

##### value

```ts
value: string;
```

Field value, visible ASCII, spaces and tabs. May carry a credential (a
cookie, an authorization): the host never logs it.

***

### OriginRefusal

The library's answer to an `Origin.connect` it did not upgrade: an
ordinary response (`401`, `403`, `404`, ...) for the plugin to hand back
to the client.

#### Properties

##### body

```ts
body: Uint8Array;
```

Response body, at most `maxOriginRefusalBodyBytes`.

##### headers

```ts
headers: OriginHeader[];
```

Response header fields in order, at most `maxOriginHeaders` and
`maxOriginHeaderBytes`.

##### status

```ts
status: number;
```

HTTP status code, 200 to 599 and never `101`.

***

### OriginRequest

The head of one public request a relay `ingress` plugin carries
(`Origin.fetch`). Its scheme is `https` and its authority is the library's
public origin; neither is the plugin's to choose.

#### Properties

##### clientAddress?

```ts
clientAddress?: string;
```

IP address of the client, from the provider's own connection metadata;
empty when the provider gives none.
Omitted when absent (wire zero value).

##### headers

```ts
headers: OriginHeader[];
```

Request header fields in order, at most `maxOriginHeaders` and
`maxOriginHeaderBytes`. `host` and forwarding headers are ignored.

##### method

```ts
method: string;
```

Request method, an HTTP token (`GET`, `PUT`, ...).

##### target

```ts
target: string;
```

Path and query in origin form (`/api/status?x=1`), at most
`maxOriginTargetBytes`.

***

### OriginResponse

Success payload of `Origin.fetch`: the library's response head and its
streamed body.

#### Properties

##### body?

```ts
body?: ByteSource | null;
```

Response body; read to `done`. Null when the response has no body.
Null when the guest does not export this capability.

##### headers

```ts
headers: OriginHeader[];
```

Response header fields in order, at most `maxOriginHeaders` and
`maxOriginHeaderBytes`; repeated names (`set-cookie`) stay separate.

##### status

```ts
status: number;
```

HTTP status code, 100 to 599.

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

##### chapterKind

```ts
chapterKind: "unknown" | "chapters" | "tracks";
```

What `chapters` marks. The host ranks literary chapters above track
boundaries when it decides an edition's chapters.

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
  | "mediaDownload"
  | "metadataFiles"
  | "notifier"
  | "ingress"
  | "ai"
  | "provisioning")[];
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

##### sockets

```ts
sockets: boolean;
```

True when the guest declared `[sockets]`: it receives the host-brokered
`Sockets` binding and may export a `SocketHandler` for pushed messages.

##### webhooks

```ts
webhooks: boolean;
```

True when the guest declared `[[webhooks]]`: it receives the host-served
`Webhooks` binding and may export a `WebhookHandler` the host wakes when
it stores deliveries.

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

### PricedRecord

One priced listing returned by a title search.

#### Properties

##### hint

```ts
hint: PurchaseHint;
```

Its purchase link and prices, as `PurchaseHint` carries them.

##### record

```ts
record: CatalogHit;
```

The listing's catalog record: identifiers, credits, kinds, language.

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

### Provisioned

Success payload of `Provisioning.provision`.

#### Properties

##### secrets

```ts
secrets: ProvisionedSecret[];
```

Credentials to seal; each key at most once.

##### settingsJson

```ts
settingsJson: string;
```

Non-secret settings to store, as a JSON object keyed by setting keys
listed in `[provisioning] outputs`; at most `maxProvisionSettingsBytes`.

##### summary

```ts
summary: string[];
```

Short lines for the setup screen ("Created bucket books"), at most
`maxProvisionSummaryLines` of `maxProvisionSummaryLineBytes`. Never a
credential.

***

### ProvisionedSecret

One credential a guided setup returns for the host to seal.

#### Properties

##### key

```ts
key: string;
```

A `secret` setting key listed in `[provisioning] outputs`.

##### value

```ts
value: string;
```

The credential, at most `maxProvisionSecretBytes`.

***

### ProvisionGrant

A short-lived credential for one guided setup run. The host never stores
or logs it; the plugin uses it only during the call.

#### Properties

##### expiresAtUnixMs?

```ts
expiresAtUnixMs?: number;
```

When the token expires (UTC Unix milliseconds); zero when unknown or it
does not expire.
Omitted when absent (wire zero value).

##### kind

```ts
kind: "none" | "apiToken" | "oauthAccessToken";
```

Kind of grant.

##### token?

```ts
token?: string;
```

The token; empty for `none`. At most `maxProvisionTokenBytes`.
Omitted when absent (wire zero value).

***

### Provisioning

Guided setup (`Entrypoints.provisioning`). The host collects a
short-lived grant from the operator (a pasted API token, or an OAuth access
token), opens the instance with `Invocation.purpose = provision` and hands
the grant over. The plugin creates what it needs at the provider and
returns settings and narrowly scoped credentials; the host checks them
against `[provisioning] outputs` and the pinned settings schema, writes the
settings, seals the credentials and drops the grant.

#### Methods

##### deprovision()

```ts
deprovision(request): Promise<EmptyReply>;
```

Remove what `provision` created. Removing what is already gone is not an
error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`DeprovisionRequest`](#deprovisionrequest) | A grant, or none to use stored secrets. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### provision()

```ts
provision(request): Promise<ProvisionReply>;
```

Create (or, when `CONFIG` holds earlier outputs, update or rotate) what
the instance needs.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`ProvisionRequest`](#provisionrequest) | The grant and the operator's choices. |

###### Returns

`Promise`\<[`ProvisionReply`](#provisionreply)\>

[ProvisionReply](#provisionreply)

***

### ProvisionRequest

Arguments of `Provisioning.provision`.

#### Properties

##### grant

```ts
grant: ProvisionGrant;
```

The operator's grant; never `none`.

##### inputsJson

```ts
inputsJson: string;
```

The operator's choices as a JSON object keyed by `[provisioning] inputs`
setting keys (`{"bucket": "books"}`); `{}` when none were asked for.

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

##### lookup

```ts
lookup: "unspecified" | "isbn" | "asin" | "title" | "product";
```

How the host asks the store to find the title. `unspecified` (an older
host) leaves the guest free to use any field, as before.

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

##### records

```ts
records: PricedRecord[];
```

Every priced listing a title search matched (`lookup = title`), each
with its full catalog record, so the host can tell recordings apart by
identifiers, credits and kinds rather than by title. Empty for other
lookups and for guests that do not return records; `hint` then stays
the answer.

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

##### modifiedAtMs

```ts
modifiedAtMs: number;
```

Modification time of the stored object, UTC unix milliseconds; zero when
unknown.

##### sha256

```ts
sha256: Uint8Array;
```

Raw SHA-256 digest of the stored bytes; empty when not computed.

##### signature

```ts
signature: string;
```

Change signature of the stored object (`ObjectMetadata.signature`), so
the host can recognise its own write in a later listing; empty when none.

##### versionId

```ts
versionId: string;
```

Version id of the stored object; empty when none.

***

### QualityOffer

The stream one account would download for a title today.

#### Properties

##### aacProfile

```ts
aacProfile: "unknown" | "lc" | "he" | "heV2" | "xhe";
```

AAC profile when `codec` is `aac`; `unknown` otherwise.

##### bitrateKbps?

```ts
bitrateKbps?: number;
```

Average bitrate in kbit/s; zero when unknown.
Omitted when absent (wire zero value).

##### channels?

```ts
channels?: number;
```

Channel count; zero when unknown.
Omitted when absent (wire zero value).

##### codec

```ts
codec: 
  | "unknown"
  | "aac"
  | "alac"
  | "flac"
  | "mp3"
  | "opus"
  | "vorbis"
  | "eac3"
  | "ac4";
```

Codec of the offered stream.

##### sampleRateHz?

```ts
sampleRateHz?: number;
```

Sample rate in Hz; zero when unknown.
Omitted when absent (wire zero value).

***

### QualityOffersParams

Params of `ContentSource.qualityOffers`: the titles one account holds.

#### Properties

##### accountId

```ts
accountId: string;
```

Account whose download would be made.

##### credentials?

```ts
credentials?: Uint8Array<ArrayBufferLike>;
```

Decrypted credential blob for this account; never log it. Empty when
none is stored.
Omitted when absent (wire zero value).

##### fetch

```ts
fetch: FetchOptions;
```

The fetch options a download would carry (DRM path, xHE-AAC), so the
answer names what that download would get.

##### sourceConfig

```ts
sourceConfig: ExtensibleConfig;
```

The instance's settings as `application/json` (bitrate tier and the
like, which change what a download gets).

##### titleIds

```ts
titleIds: string[];
```

Store product ids, as the account's scan listed them; at most
`maxListPage`.

***

### QualityOffersResult

Success payload of `ContentSource.qualityOffers`.

#### Properties

##### offers

```ts
offers: TitleQualityOffer[];
```

One answer per title id, in the order they were sent. A short list
leaves the rest unanswered.

##### refreshedCredentials?

```ts
refreshedCredentials?: Uint8Array<ArrayBufferLike>;
```

The account's credentials as the guest holds them after refreshing them
during the call, as in `ScanPage.refreshedCredentials`. Empty when
nothing changed.
Omitted when absent (wire zero value).

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

##### ackEvents()

```ts
ackEvents(params): Promise<EmptyReply>;
```

Acknowledge users a `pollEvents` reply handed over: the host has passed
them on, so the guest stops returning them.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`EventAckParams`](#eventackparams) | The acknowledged users. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

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
pollEvents(params): Promise<EventPollReply>;
```

External users the remote side observed and the host has not
acknowledged. Without `params.ackFollows` (hosts that predate
`ackEvents`) a returned user counts as delivered.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | [`EventPollParams`](#eventpollparams) | Whether the host acknowledges what it receives. |

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

##### explicitness

```ts
explicitness: "unknown" | "clean" | "explicit";
```

Whether the storefront marks the edition explicit (adult) content.

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
scheduled(controller, cancel): Promise<EmptyReply>;
```

Run one cron fire to completion.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `controller` | [`ScheduledController`](#scheduledcontroller) | The fire being delivered. |
| `cancel` | [`Cancellation`](#cancellation) | Fires once the host cancels the fire (fence loss or operator cancel); null from hosts that predate it. |

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

### SettingNetworkGrant

One network setting and the destination its value names.

#### Properties

##### destination?

```ts
destination?: string;
```

The destination as the consent form shows it: `scheme://host:port` for
a fetch, `host:port` for TCP (IPv6 in brackets); empty when `state` is
`unset` or `invalid`.
Omitted when absent (wire zero value).

##### error?

```ts
error?: string;
```

Why the value names no destination, when `state` is `invalid`.
Omitted when absent (wire zero value).

##### key

```ts
key: string;
```

The setting key.

##### network

```ts
network: "fetch" | "tcp";
```

Whether the plugin fetches the destination or opens a socket to it.

##### state

```ts
state: "approved" | "pending" | "unset" | "invalid";
```

Where the destination stands.

***

### SocketAutoResponse

A heartbeat the broker answers itself: when `request` arrives byte for byte,
it sends `response` and does not wake the guest.

#### Properties

##### request

```ts
request: SocketFrame;
```

Frame to match, at most `maxSocketAutoResponseBytes`.

##### response

```ts
response: SocketFrame;
```

Frame to send back, at most `maxSocketAutoResponseBytes`.

***

### SocketClosed

Argument of `SocketHandler.closed`.

#### Properties

##### attachment

```ts
attachment: Uint8Array;
```

The socket's last attachment.

##### clean

```ts
clean: boolean;
```

True when the close handshake completed.

##### code

```ts
code: number;
```

WebSocket close code; 1006 when the connection dropped without one.

##### reason?

```ts
reason?: string;
```

Close reason the peer sent; empty when none.
Omitted when absent (wire zero value).

##### socket

```ts
socket: string;
```

Socket that closed; its id is not reused.

***

### SocketConnectRequest

Arguments of `Sockets.connect`.

#### Properties

##### attachment

```ts
attachment: Uint8Array;
```

Initial attachment, at most `maxSocketAttachmentBytes`.

##### autoResponses

```ts
autoResponses: SocketAutoResponse[];
```

Heartbeats the broker answers, at most `maxSocketAutoResponses`; none on
`tls://`.

##### headers

```ts
headers: SocketHeader[];
```

Extra request headers, at most `maxSocketHeaders`; none on `tls://`.

##### protocols

```ts
protocols: string[];
```

Requested subprotocols, at most `maxSocketHeaders`; on a `tls://`
stream, the ALPN protocol ids to offer (visible ASCII, 1 to 255 bytes).

##### serverName

```ts
serverName: string;
```

`tls://` only: the DNS name to send as SNI and to check the server's
certificate against, for an endpoint reached at one name that answers
for another (a Cloudflare Tunnel edge, dialed at `region1.v2.argotunnel.com`
and serving `h2.cftunnel.com`). It must be on a `tcp` grant for the
URL's port too, so it can't route past the grant by name. At most
`maxSocketServerNameBytes`; empty uses the URL's host.

##### trustAnchors

```ts
trustAnchors: Uint8Array<ArrayBufferLike>[];
```

`tls://` only: DER root certificates, at most `maxSocketTrustAnchors` of
at most `maxSocketTrustAnchorBytes` each, that the server's chain must
end in instead of Mozilla's roots, for a service that signs its own
(an ngrok or Cloudflare Tunnel agent endpoint). Empty uses Mozilla's.

##### url

```ts
url: string;
```

`wss://` or `ws://` URL, at most `maxSocketUrlBytes`, or
`tls://host:port` for a raw TLS byte stream (port required, nothing
after it). The host must be on the plugin's network grant; a `tls://`
stream needs a `tcp` grant for that host and port.

***

### SocketHandler

Guest handler for host-brokered sockets (`Entrypoints.socketHandler`).

#### Methods

##### closed()

```ts
closed(event): Promise<EmptyReply>;
```

A socket closed, by the remote side, the network, or the host.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `event` | [`SocketClosed`](#socketclosed) | Socket, close code, and attachment. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### message()

```ts
message(event): Promise<SocketMessageReply>;
```

One message arrived on an open socket.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `event` | [`SocketMessage`](#socketmessage) | Socket, frame, and attachment. |

###### Returns

`Promise`\<[`SocketMessageReply`](#socketmessagereply)\>

[SocketMessageReply](#socketmessagereply)

***

### SocketHeader

One request header on `Sockets.connect`.

#### Properties

##### name

```ts
name: string;
```

Header name; the broker refuses hop-by-hop and WebSocket handshake names.

##### value

```ts
value: string;
```

Header value. May carry a credential: the host never logs it.

***

### SocketInfo

One open host-brokered socket.

#### Properties

##### attachment

```ts
attachment: Uint8Array;
```

Current attachment.

##### protocol?

```ts
protocol?: string;
```

Subprotocol the server chose (on `tls://`, the negotiated ALPN id);
empty when none.
Omitted when absent (wire zero value).

##### socket

```ts
socket: string;
```

Host-minted socket id, stable for the socket's life.

##### url

```ts
url: string;
```

URL the socket was opened to, without credentials.

***

### SocketList

Success payload of `Sockets.list`.

#### Properties

##### sockets

```ts
sockets: SocketInfo[];
```

Open sockets, oldest first.

***

### SocketMessage

Argument of `SocketHandler.message`.

#### Properties

##### attachment

```ts
attachment: Uint8Array;
```

The socket's attachment when the message arrived.

##### frame

```ts
frame: SocketFrame;
```

The message.

##### socket

```ts
socket: string;
```

Socket the message arrived on.

***

### SocketMessageOk

What a guest asks of the host after handling one pushed message.

#### Properties

##### pollEvents

```ts
pollEvents: boolean;
```

True asks the host to call `Integration.pollEvents` now, while the
guest is awake, instead of at the next scheduled poll: the message
announced something the poll hands over (a new user). The host ignores
it for a plugin whose host schedules no poll.

***

### Sockets

Host-brokered outbound WebSockets, granted through `Bindings.sockets`.

The guest asks for a socket; a broker the host runs for the plugin
instance dials it under the plugin's network grant and keeps it open while
the guest sleeps, in the manner of hibernatable Workers WebSockets. Each
incoming message wakes the guest and arrives on
`Entrypoints.socketHandler`. Heartbeat pairs declared at connect are
answered by the broker without waking the guest.

#### Methods

##### close()

```ts
close(
   socket, 
   code, 
   reason
): Promise<EmptyReply>;
```

Close a socket; closing an unknown or closed socket is not an error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `socket` | `string` | Socket id. |
| `code` | `number` | WebSocket close code; zero sends 1000. |
| `reason` | `string` | Close reason, at most 123 UTF-8 bytes. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### connect()

```ts
connect(request): Promise<SocketConnectReply>;
```

Open a socket; it is owned by the plugin instance, not this invocation.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`SocketConnectRequest`](#socketconnectrequest) | Target, subprotocols, headers, heartbeats. |

###### Returns

`Promise`\<[`SocketConnectReply`](#socketconnectreply)\>

[SocketConnectReply](#socketconnectreply)

##### list()

```ts
list(): Promise<SocketListReply>;
```

The instance's open sockets, for a guest that just woke.

###### Returns

`Promise`\<[`SocketListReply`](#socketlistreply)\>

[SocketListReply](#socketlistreply)

##### send()

```ts
send(socket, frame): Promise<EmptyReply>;
```

Send one message on an open socket.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `socket` | `string` | Socket id from `connect` or `list`. |
| `frame` | [`SocketFrame`](#socketframe) | Message, at most `maxSocketFrameBytes`. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### setAttachment()

```ts
setAttachment(socket, attachment): Promise<EmptyReply>;
```

Replace the attachment the host keeps for a socket.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `socket` | `string` | Socket id. |
| `attachment` | `Uint8Array` | Opaque bytes, at most `maxSocketAttachmentBytes`. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

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

### TitleQualityOffer

One title's answer in `QualityOffersResult`.

#### Properties

##### found

```ts
found: boolean;
```

False when the store could not say for this title (`offer` is empty).

##### offer

```ts
offer: QualityOffer;
```

What a download would get, when `found`.

***

### TranslateKeysResult

Success payload of `Destination.translateKeys`.

#### Properties

##### keys

```ts
keys: string[];
```

One translated key per requested host key, in request order: the key
the guest stores that host key under. Each is a relative key with no
empty, `.` or `..` segment.

***

### WebhookAckRequest

Arguments of `Webhooks.ack`.

#### Properties

##### acks

```ts
acks: string[];
```

Leases whose deliveries are done and are deleted.

##### name

```ts
name: string;
```

`[[webhooks]]` name.

##### retries

```ts
retries: string[];
```

Leases whose deliveries the next pull offers again.

***

### WebhookDelivered

Argument of `WebhookHandler.delivered`.

#### Properties

##### name

```ts
name: string;
```

`[[webhooks]]` name.

##### waiting

```ts
waiting: number;
```

Deliveries waiting for this webhook when the host called.

***

### WebhookDelivery

One stored request, leased to the guest.

#### Properties

##### attempt

```ts
attempt: number;
```

How many times this delivery has been pulled, counting this one.

##### body

```ts
body: Uint8Array;
```

Request body as received, at most the row's `max_body_bytes`.

##### headers

```ts
headers: WebhookHeader[];
```

Kept request headers, in the order the row names them.

##### leaseId

```ts
leaseId: string;
```

Lease id for `ack`, at most `maxWebhookLeaseIdBytes`; a new one each
time the delivery is pulled.

##### receivedAtUnixMs

```ts
receivedAtUnixMs: number;
```

When the host stored the request, in Unix milliseconds.

***

### WebhookHandler

Guest handler for inbound webhooks (`Entrypoints.webhookHandler`).

#### Methods

##### delivered()

```ts
delivered(event): Promise<EmptyReply>;
```

The host stored deliveries for one webhook; the guest pulls them.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `event` | [`WebhookDelivered`](#webhookdelivered) | Webhook and how many wait. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

***

### WebhookHeader

One request header kept with a delivery.

#### Properties

##### name

```ts
name: string;
```

Lowercase header name, one the row's `headers` names or `content-type`.

##### value

```ts
value: string;
```

Header value as received.

***

### WebhookPullPage

Success payload of `Webhooks.pull`.

#### Properties

##### deliveries

```ts
deliveries: WebhookDelivery[];
```

Leased deliveries, oldest first.

##### missed

```ts
missed: boolean;
```

True when the host dropped deliveries for this webhook since the last
pull (the waiting cap was full or a delivery expired unacknowledged):
the guest should treat what it follows as incomplete.

***

### WebhookPullRequest

Arguments of `Webhooks.pull`.

#### Properties

##### batchSize

```ts
batchSize: number;
```

Most deliveries to lease, 1 to `maxWebhookPullBatch`.

##### name

```ts
name: string;
```

`[[webhooks]]` name.

##### visibilityTimeoutMs

```ts
visibilityTimeoutMs: number;
```

How long leased deliveries stay hidden from the next pull, in
milliseconds, at most `maxWebhookVisibilityMs`; zero means one minute.

***

### Webhooks

Host-served inbound webhooks, granted through `Bindings.webhooks`.

The host serves each `[[webhooks]]` row at a secret URL under the
library's public origin, checks every request against the row's body cap,
rate limit and signature rule, stores it and answers at once. The guest
takes stored deliveries with `pull` and settles them with `ack`, in the
manner of a Cloudflare Queues HTTP pull consumer: a pulled delivery stays
leased until acknowledged, retried, or its visibility timeout ends.

#### Methods

##### ack()

```ts
ack(request): Promise<EmptyReply>;
```

Acknowledge finished deliveries and offer retried ones again at once.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`WebhookAckRequest`](#webhookackrequest) | Webhook and lease ids. |

###### Returns

`Promise`\<[`EmptyReply`](#emptyreply)\>

[EmptyReply](#emptyreply)

##### pull()

```ts
pull(request): Promise<WebhookPullReply>;
```

Lease up to `batchSize` stored deliveries, oldest first.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `request` | [`WebhookPullRequest`](#webhookpullrequest) | Webhook, batch size, visibility timeout. |

###### Returns

`Promise`\<[`WebhookPullReply`](#webhookpullreply)\>

[WebhookPullReply](#webhookpullreply)

##### url()

```ts
url(name): Promise<WebhookUrlReply>;
```

The URL an outside service should call for one webhook.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |

###### Returns

`Promise`\<[`WebhookUrlReply`](#webhookurlreply)\>

[WebhookUrlReply](#webhookurlreply)

***

### WebhookUrl

Success payload of `Webhooks.url`.

#### Properties

##### url?

```ts
url?: string;
```

Absolute `https://` URL; empty when the library cannot be reached from
the Internet or webhooks are turned off, and the plugin should use its
fallback.
Omitted when absent (wire zero value).

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

##### ifMatch

```ts
ifMatch: string;
```

Conditional write (`storage.conditionalWrite`): publish only while the
stored object's `etag` equals this value, else fail with `conflict`.
Empty means unconditional. A staged put checks it at `commit`.

##### ifNoneMatch

```ts
ifNoneMatch: boolean;
```

Conditional write (`storage.conditionalWrite`): publish only when no
object exists at the key, else fail with `conflict`.

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

### AiModelsReply

```ts
type AiModelsReply = 
  | {
  kind: "ok";
  value: AiModelList;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AiModels.models`.

***

### AiRunReply

```ts
type AiRunReply = 
  | {
  kind: "ok";
  value: AiRunOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `AiModels.run` and `AiRunner.run`. A model the plugin
does not serve is `not_found`; an input for another task is
`invalid_params`; a caller without the grant gets `forbidden`.

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

### ChangesReply

```ts
type ChangesReply = 
  | {
  kind: "ok";
  value: ChangesPage;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.changes`.

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

### DeleteManyReply

```ts
type DeleteManyReply = 
  | {
  kind: "ok";
  value: DeleteManyResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.deleteMany`.

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

### DownloadResultsReply

```ts
type DownloadResultsReply = 
  | {
  kind: "ok";
  value: DownloadResults;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `MediaDownloader.downloadMedia`.

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

### HeadManyReply

```ts
type HeadManyReply = 
  | {
  kind: "ok";
  value: HeadManyResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.headMany`.

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

### IngressHealthReply

```ts
type IngressHealthReply = 
  | {
  kind: "ok";
  value: IngressHealth;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Ingress.health`.

***

### IngressStartReply

```ts
type IngressStartReply = 
  | {
  kind: "ok";
  value: IngressStarted;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Ingress.start`.

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

### MetadataFileParseReply

```ts
type MetadataFileParseReply = 
  | {
  kind: "ok";
  value: MetadataFileParsed;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `MetadataFiles.parse`. A file the dialect cannot read is
`invalid_params`, so the host records it as unreadable and moves on.

***

### MetadataFileRenderReply

```ts
type MetadataFileRenderReply = 
  | {
  kind: "ok";
  value: MetadataFileRendered;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `MetadataFiles.render`.

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

### MoveManyReply

```ts
type MoveManyReply = 
  | {
  kind: "ok";
  value: MoveManyPage;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.moveMany`.

***

### MoveManyTarget

```ts
type MoveManyTarget = 
  | {
  kind: "pairs";
  value: KeyPair[];
}
  | {
  kind: "prefix";
  value: KeyPair;
};
```

What `Destination.moveMany` moves.

***

### MoveReply

```ts
type MoveReply = 
  | {
  kind: "ok";
  value: MoveResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.move`.

***

### NotifyDeliverabilityReply

```ts
type NotifyDeliverabilityReply = 
  | {
  kind: "ok";
  value: NotifyDeliverabilityReport;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Notifier.checkDeliverability`.

***

### NotifyRaiseReply

```ts
type NotifyRaiseReply = 
  | {
  kind: "ok";
  value: NotifyRaised;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `NotificationRaiser.raise`.

***

### NotifySendReply

```ts
type NotifySendReply = 
  | {
  kind: "ok";
  value: NotifySent;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Notifier.send`.

***

### NotifyValidateReply

```ts
type NotifyValidateReply = 
  | {
  kind: "ok";
  value: NotifyValidated;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Notifier.validateDestination`. An address the plugin
refuses is `invalid_params`.

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

### OriginConnected

```ts
type OriginConnected = 
  | {
  kind: "socket";
  value: SocketInfo;
}
  | {
  kind: "refused";
  value: OriginRefusal;
};
```

Success payload of `Origin.connect`.

***

### OriginConnectReply

```ts
type OriginConnectReply = 
  | {
  kind: "ok";
  value: OriginConnected;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Origin.connect`.

***

### OriginFetchReply

```ts
type OriginFetchReply = 
  | {
  kind: "ok";
  value: OriginResponse;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Origin.fetch`.

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

### ProvisionReply

```ts
type ProvisionReply = 
  | {
  kind: "ok";
  value: Provisioned;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Provisioning.provision`.

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

### QualityOffersReply

```ts
type QualityOffersReply = 
  | {
  kind: "ok";
  value: QualityOffersResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `ContentSource.qualityOffers`. A whole-batch `err` (for
example `rateLimited`) applies to every title in it.

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

### SocketConnectReply

```ts
type SocketConnectReply = 
  | {
  kind: "ok";
  value: SocketInfo;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Sockets.connect`.

***

### SocketFrame

```ts
type SocketFrame = 
  | {
  kind: "text";
  value: string;
}
  | {
  kind: "binary";
  value: Uint8Array;
};
```

One WebSocket message, or one chunk of a `tls://` stream (always
`binary`).

***

### SocketListReply

```ts
type SocketListReply = 
  | {
  kind: "ok";
  value: SocketList;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Sockets.list`.

***

### SocketMessageReply

```ts
type SocketMessageReply = 
  | {
  kind: "ok";
  value: SocketMessageOk;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Reply of `SocketHandler.message`.

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

### TranslateKeysReply

```ts
type TranslateKeysReply = 
  | {
  kind: "ok";
  value: TranslateKeysResult;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Destination.translateKeys`.

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

***

### WebhookPullReply

```ts
type WebhookPullReply = 
  | {
  kind: "ok";
  value: WebhookPullPage;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Webhooks.pull`.

***

### WebhookUrlReply

```ts
type WebhookUrlReply = 
  | {
  kind: "ok";
  value: WebhookUrl;
}
  | {
  kind: "err";
  value: PluginError;
};
```

Result union of `Webhooks.url`.

## References

### Abridgement

Re-exports [Abridgement](abi.md#abridgement)

***

### AiTask

Re-exports [AiTask](abi.md#aitask)

***

### AiTextRole

Re-exports [AiTextRole](abi.md#aitextrole)

***

### CatalogField

Re-exports [CatalogField](abi.md#catalogfield)

***

### CatalogSort

Re-exports [CatalogSort](abi.md#catalogsort)

***

### ChapterKind

Re-exports [ChapterKind](abi.md#chapterkind)

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

### Explicitness

Re-exports [Explicitness](abi.md#explicitness)

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

### InvocationPurpose

Re-exports [InvocationPurpose](abi.md#invocationpurpose)

***

### IsolationReq

Re-exports [IsolationReq](abi.md#isolationreq)

***

### MediaKind

Re-exports [MediaKind](abi.md#mediakind)

***

### MetadataField

Re-exports [MetadataField](abi.md#metadatafield)

***

### NotificationAlertAction

Re-exports [NotificationAlertAction](abi.md#notificationalertaction)

***

### NotificationAudience

Re-exports [NotificationAudience](abi.md#notificationaudience)

***

### NotificationSeverity

Re-exports [NotificationSeverity](abi.md#notificationseverity)

***

### NotifyCheckOutcome

Re-exports [NotifyCheckOutcome](abi.md#notifycheckoutcome)

***

### NotifyDestinationKind

Re-exports [NotifyDestinationKind](abi.md#notifydestinationkind)

***

### ObjectChangeKind

Re-exports [ObjectChangeKind](abi.md#objectchangekind)

***

### OfferedAacProfile

Re-exports [OfferedAacProfile](abi.md#offeredaacprofile)

***

### OfferedCodec

Re-exports [OfferedCodec](abi.md#offeredcodec)

***

### PluginErrorCode

Re-exports [PluginErrorCode](abi.md#pluginerrorcode)

***

### PortalAuthMode

Re-exports [PortalAuthMode](abi.md#portalauthmode)

***

### PriceLookup

Re-exports [PriceLookup](abi.md#pricelookup)

***

### ProvisionGrantKind

Re-exports [ProvisionGrantKind](abi.md#provisiongrantkind)

***

### RemoteMedia

Re-exports [RemoteMedia](abi.md#remotemedia)

***

### ResolvedSqlType

Re-exports [ResolvedSqlType](abi.md#resolvedsqltype)

***

### SettingGrantState

Re-exports [SettingGrantState](abi.md#settinggrantstate)

***

### SettingNetworkKind

Re-exports [SettingNetworkKind](abi.md#settingnetworkkind)

***

### WorkKind

Re-exports [WorkKind](abi.md#workkind)
