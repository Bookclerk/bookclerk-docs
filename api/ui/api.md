# api

## Classes

### ApiError

HTTP failure from Bookclerk JSON APIs, including optional machine `code`.

#### Extends

- `Error`

#### Constructors

##### Constructor

```ts
new ApiError(
   status, 
   message, 
   extras?
): ApiError;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `status` | `number` |
| `message` | `string` |
| `extras?` | \{ `code?`: `string`; `pluginId?`: `string`; `summary?`: `string`[]; \} |
| `extras.code?` | `string` |
| `extras.pluginId?` | `string` |
| `extras.summary?` | `string`[] |

###### Returns

[`ApiError`](#apierror)

###### Overrides

```ts
Error.constructor
```

#### Properties

##### code?

```ts
optional code?: string;
```

Machine slug from JSON error bodies when present.

##### pluginId?

```ts
optional pluginId?: string;
```

##### status

```ts
status: number;
```

##### summary?

```ts
optional summary?: string[];
```

## Interfaces

### ActionResponse

Ack payload for scan / acquire job triggers.

#### Properties

##### error?

```ts
optional error?: string | null;
```

##### job\_id

```ts
job_id: string;
```

##### message

```ts
message: string;
```

##### ok

```ts
ok: boolean;
```

***

### AuthMeImpersonating

Active impersonation target when an administrator is viewing as another user.

#### Properties

##### display\_name

```ts
display_name: string | null;
```

##### user\_id

```ts
user_id: number;
```

***

### AuthMeUser

Signed-in administrator or member profile from `/api/auth/me`.

#### Properties

##### avatar\_source?

```ts
optional avatar_source?: string | null;
```

`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`.

##### display\_name

```ts
display_name: string | null;
```

##### email?

```ts
optional email?: string | null;
```

##### gravatar\_hash?

```ts
optional gravatar_hash?: string | null;
```

SHA-256 hex of the contact email for Gravatar.

##### has\_avatar?

```ts
optional has_avatar?: boolean;
```

True when a profile picture file is stored for this user.

##### has\_password?

```ts
optional has_password?: boolean;
```

##### id

```ts
id: number;
```

##### login\_name?

```ts
optional login_name?: string | null;
```

##### role

```ts
role: string;
```

##### sso\_pictures?

```ts
optional sso_pictures?: object[];
```

IdP-supplied pictures the user can pick.

###### identity\_id

```ts
identity_id: number;
```

###### last\_used\_at?

```ts
optional last_used_at?: string | null;
```

###### picture\_url

```ts
picture_url: string;
```

###### provider

```ts
provider: string;
```

##### status?

```ts
optional status?: string;
```

***

### AuthSecondFactor

Host MFA policy plus this user's TOTP / passkey enrollment.

#### Properties

##### enrolled

```ts
enrolled: boolean;
```

##### passkey\_count

```ts
passkey_count: number;
```

##### required

```ts
required: boolean;
```

##### totp

```ts
totp: boolean;
```

***

### AuthSession

Normalized client session used to gate SPA views and actions.

#### Properties

##### authenticated

```ts
authenticated: boolean;
```

##### can\_acquire

```ts
can_acquire: boolean;
```

##### default\_view

```ts
default_view: AppView;
```

##### elevated?

```ts
optional elevated?: boolean;
```

##### impersonating?

```ts
optional impersonating?: AuthMeImpersonating;
```

##### portal?

```ts
optional portal?: PortalInfo;
```

##### role?

```ts
optional role?: AuthRole;
```

##### second\_factor?

```ts
optional second_factor?: AuthSecondFactor;
```

##### user?

```ts
optional user?: AuthMeUser;
```

***

### BookRecord

Library book row from `/api/library/books`.

Field names match the daemon JSON (`snake_case`). Use `uuid` for acquire /
cover URLs; `asin` / `isbn` are store product ids when known.

#### Properties

##### account\_id

```ts
account_id: string;
```

Sealed account id that owns this title.

##### acquire\_status

```ts
acquire_status: AcquireStatus;
```

Pipeline state for the main audio acquire job.

##### asin

```ts
asin: string | null;
```

Audible ASIN when the source provides one.

##### authors

```ts
authors: string | null;
```

Semicolon- or comma-separated author list.

##### categories

```ts
categories: string | null;
```

Genre / category list from the store (often `;`- or `,`-separated).

##### content\_kind

```ts
content_kind: string;
```

Content kind slug (`audiobook`, `podcast`, …).

##### cover\_url

```ts
cover_url: string | null;
```

Remote or proxied cover image URL.

##### created\_at

```ts
created_at: string;
```

Row insert time (ISO-8601).

##### description

```ts
description: string | null;
```

Long-form description / blurb (may contain light HTML).

##### enrich\_confidence

```ts
enrich_confidence: number | null;
```

Enrichment confidence score (0–1) when present.

##### enrich\_source

```ts
enrich_source: string | null;
```

Enrichment provider id that last wrote metadata.

##### enrich\_updated\_at

```ts
enrich_updated_at: string | null;
```

When enrichment last ran (ISO-8601).

##### error\_message

```ts
error_message: string | null;
```

Last acquire / scan error message when status is `error`.

##### id

```ts
id: number;
```

Library row primary key.

##### is\_abridged

```ts
is_abridged: boolean;
```

True when the edition is abridged.

##### is\_finished

```ts
is_finished: boolean;
```

Whether the listener marked the title finished.

##### isbn

```ts
isbn: string | null;
```

ISBN-13 / ISBN-10 when the source provides one.

##### language

```ts
language: string | null;
```

BCP-47-ish language code from the store.

##### length\_minutes

```ts
length_minutes: number | null;
```

Runtime length in minutes when known.

##### marketplace

```ts
marketplace: string;
```

Marketplace / locale code (`us`, `uk`, …).

##### narrators

```ts
narrators: string | null;
```

Semicolon- or comma-separated narrator list.

##### pdf\_status

```ts
pdf_status: AcquireStatus;
```

Pipeline state for companion PDF acquire.

##### pdf\_storage\_key

```ts
pdf_storage_key: string | null;
```

Destination key for an acquired companion PDF.

##### product\_id

```ts
product_id: string;
```

Store-native product id (ASIN, ISBN, UUID, …).

##### published\_at

```ts
published_at: string | null;
```

Original publication date (ISO-8601 date or datetime).

##### publisher

```ts
publisher: string | null;
```

Publisher name when known.

##### purchased\_at

```ts
purchased_at: string | null;
```

Purchase / library-add timestamp from the store (ISO-8601).

##### rating\_overall

```ts
rating_overall: number | null;
```

Overall community rating (0–5) when enriched.

##### rating\_performance

```ts
rating_performance: number | null;
```

Performance / narrator rating when enriched.

##### rating\_story

```ts
rating_story: number | null;
```

Story rating when enriched.

##### series

```ts
series: string | null;
```

Series name when known.

##### series\_asin

```ts
series_asin: string | null;
```

Audible series / podcast-parent ASIN when known.

##### series\_index

```ts
series_index: string | null;
```

Position within the series (string to preserve `#1.5` styles).

##### source

```ts
source: string;
```

Storefront plugin id (`audible`, `libro`, …).

##### storage\_key

```ts
storage_key: string | null;
```

Destination object key after a successful acquire.

##### subjects

```ts
subjects: string | null;
```

Subject / BISAC-style tags when enriched.

##### subtitle

```ts
subtitle: string | null;
```

Subtitle when the store provides one.

##### tags

```ts
tags: string | null;
```

Space-separated user tags.

##### title

```ts
title: string;
```

Display title.

##### updated\_at

```ts
updated_at: string;
```

Row last update time (ISO-8601).

##### uuid

```ts
uuid: string;
```

Stable book uuid used by acquire / cover / detail APIs.

***

### BooksResponse

Paginated library books list payload.

#### Properties

##### books

```ts
books: BookRecord[];
```

##### limit

```ts
limit: number;
```

##### offset

```ts
offset: number;
```

##### total

```ts
total: number;
```

***

### BootstrapAdministratorResponse

Bootstrap response for the first owner.

#### Properties

##### claim\_ticket

```ts
claim_ticket: string;
```

##### has\_password

```ts
has_password: boolean;
```

##### invite\_url?

```ts
optional invite_url?: string | null;
```

##### login\_name

```ts
login_name: string | null;
```

##### ok

```ts
ok: boolean;
```

##### user\_id

```ts
user_id: number;
```

***

### CatalogSearchHit

One hit from `/api/discover/search`.

#### Properties

##### asin

```ts
asin: string | null;
```

##### authors

```ts
authors: string | null;
```

##### cover\_url?

```ts
optional cover_url?: string | null;
```

##### description?

```ts
optional description?: string | null;
```

##### genres?

```ts
optional genres?: string | null;
```

##### is\_abridged?

```ts
optional is_abridged?: boolean | null;
```

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
optional language?: string | null;
```

##### length\_minutes?

```ts
optional length_minutes?: number | null;
```

##### narrators

```ts
narrators: string | null;
```

##### price\_cents?

```ts
optional price_cents?: number | null;
```

##### published\_at?

```ts
optional published_at?: string | null;
```

##### publisher?

```ts
optional publisher?: string | null;
```

##### purchase\_hints?

```ts
optional purchase_hints?: PurchaseHint[];
```

##### rating\_count?

```ts
optional rating_count?: number | null;
```

##### rating\_overall?

```ts
optional rating_overall?: number | null;
```

##### series

```ts
series: string | null;
```

##### series\_index?

```ts
optional series_index?: string | null;
```

##### sources

```ts
sources: string[];
```

##### store\_editions

```ts
store_editions: StoreEdition[];
```

##### subtitle?

```ts
optional subtitle?: string | null;
```

##### title

```ts
title: string;
```

##### work\_key

```ts
work_key: string;
```

***

### CatalogSearchPage

Cursor-paginated catalog search page.

#### Properties

##### has\_more

```ts
has_more: boolean;
```

##### items

```ts
items: CatalogSearchHit[];
```

##### next\_cursor?

```ts
optional next_cursor?: string | null;
```

##### page\_size

```ts
page_size: number;
```

##### sort

```ts
sort: string;
```

##### sort\_dir?

```ts
optional sort_dir?: string;
```

***

### ClusterConsentsResponse

`GET /api/config/cluster-consents`.

#### Properties

##### clusters

```ts
clusters: ClusterLinkView[];
```

Every owner cluster link this install holds, in any state.

##### consents

```ts
consents: RemoteConsentView[];
```

Every recorded owner plugin request, approved or not.

***

### ClusterLinkView

An owner cluster this install is enrolled in as a host (no key material).

#### Properties

##### cluster\_id

```ts
cluster_id: string;
```

Id of the owner cluster; one link per cluster.

##### enrolled\_at

```ts
enrolled_at: string;
```

RFC 3339 time of the latest enrollment (a re-enrollment resets it).

##### include\_branches?

```ts
optional include_branches?: boolean;
```

"Include books from my own branches" (on by default).

##### node\_id

```ts
node_id: string;
```

Node id the owner minted for this install.

##### owner\_origin

```ts
owner_origin: string;
```

Canonical origin (`https://host[:port]`) this host dials the owner at.

##### pinned

```ts
pinned: boolean;
```

True when the link pins the owner's self-signed certificate.

##### share\_books?

```ts
optional share_books?: boolean;
```

"Share this branch's books with <library>" (on by default).

##### state

```ts
state: string;
```

`active`, `revoked` (by the owner), or `left` (by this host).

***

### ClusterStatusResponse

`GET /api/cluster/status`: cluster health plus the nodes the caller may see.

#### Extends

- [`NodesResponse`](#nodesresponse)

#### Properties

##### capabilities

```ts
capabilities: NodeCapabilities;
```

###### Inherited from

[`NodesResponse`](#nodesresponse).[`capabilities`](#capabilities-1)

##### cluster

```ts
cluster: ClusterSummary;
```

##### node\_id

```ts
node_id: string;
```

###### Inherited from

[`NodesResponse`](#nodesresponse).[`node_id`](#node_id-3)

##### nodes

```ts
nodes: ConfigNodeView[];
```

###### Inherited from

[`NodesResponse`](#nodesresponse).[`nodes`](#nodes-2)

***

### ClusterSummary

Cluster-wide health from `GET /api/cluster/status`.

#### Properties

##### active\_jobs

```ts
active_jobs: number;
```

##### cpu\_budget\_millicores

```ts
cpu_budget_millicores: number;
```

##### cpu\_reserved\_millicores

```ts
cpu_reserved_millicores: number;
```

##### deliveries

```ts
deliveries: object;
```

###### dead\_letter

```ts
dead_letter: number;
```

###### oldest\_pending\_age\_secs

```ts
oldest_pending_age_secs: number | null;
```

###### pending

```ts
pending: number;
```

###### running

```ts
running: number;
```

##### drained

```ts
drained: number;
```

##### host\_nodes

```ts
host_nodes: number;
```

##### live

```ts
live: number;
```

##### memory\_budget\_bytes

```ts
memory_budget_bytes: number;
```

##### memory\_reserved\_bytes

```ts
memory_reserved_bytes: number;
```

##### nodes

```ts
nodes: number;
```

##### running\_guests

```ts
running_guests: number;
```

***

### ConfigNodeView

One cluster node as `GET /api/config/nodes` returns it.

#### Properties

##### implicit\_labels

```ts
implicit_labels: string[];
```

Labels every node carries (`node=<id>`, `target=<target>`).

##### kind

```ts
kind: "owner" | "host";
```

`owner` (shares the database) or `host` (its own database, enrolled to contribute).

##### labels

```ts
labels: string[];
```

Operator labels.

##### live

```ts
live: boolean;
```

True while the node's daemon heartbeats.

##### node\_id

```ts
node_id: string;
```

##### owner

```ts
owner: 
  | {
  display_name: string | null;
  role: string;
  user_id: number;
}
  | null;
```

User who administers this node, when one is assigned.

##### runtime

```ts
runtime: NodeRuntimeView;
```

##### software\_version?

```ts
optional software_version?: string;
```

##### status

```ts
status: NodeStatusView | null;
```

Latest heartbeat; null when the node's daemon never ran.

##### target

```ts
target: string;
```

##### this\_node

```ts
this_node: boolean;
```

True for the node serving this request.

***

### DiscoverFeed

Discover recommendations feed payload.

#### Properties

##### shelf\_kinds?

```ts
optional shelf_kinds?: ShelfKindInfo[];
```

##### shelves

```ts
shelves: DiscoverShelf[];
```

***

### DiscoverShelf

One Discover feed shelf (id, copy, items).

#### Properties

##### id

```ts
id: string;
```

##### items

```ts
items: Recommendation[];
```

##### subtitle

```ts
subtitle: string | null;
```

##### title

```ts
title: string;
```

***

### EventDeliveryInfo

Operator delivery row from `/api/events/deliveries`.

#### Properties

##### attemptCount

```ts
attemptCount: number;
```

##### cancelRequested

```ts
cancelRequested: boolean;
```

##### errorMessage

```ts
errorMessage: string | null;
```

##### eventId

```ts
eventId: string;
```

##### hasCheckpoint

```ts
hasCheckpoint: boolean;
```

##### id

```ts
id: string;
```

##### invocationSequence

```ts
invocationSequence: number;
```

##### leaseGeneration

```ts
leaseGeneration: number;
```

##### outcome

```ts
outcome: string | null;
```

##### pluginId

```ts
pluginId: string;
```

##### resourceClass

```ts
resourceClass: string;
```

##### resumePending

```ts
resumePending: boolean;
```

##### runAfter

```ts
runAfter: string;
```

##### state

```ts
state: string;
```

##### updatedAt

```ts
updatedAt: string;
```

***

### EventQueueStatus

Durable domain-event delivery-queue counters from `/api/status`.

#### Properties

##### acked

```ts
acked: number;
```

##### dead\_letter

```ts
dead_letter: number;
```

##### dead\_letters\_total?

```ts
optional dead_letters_total?: number;
```

##### dispatch\_latency\_ms\_avg?

```ts
optional dispatch_latency_ms_avg?: number | null;
```

##### handler\_latency\_ms\_avg?

```ts
optional handler_latency_ms_avg?: number | null;
```

##### oldest\_pending\_age\_secs

```ts
oldest_pending_age_secs: number | null;
```

##### pending

```ts
pending: number;
```

##### retries\_total?

```ts
optional retries_total?: number;
```

##### running

```ts
running: number;
```

##### suspended

```ts
suspended: number;
```

##### suspensions\_total?

```ts
optional suspensions_total?: number;
```

***

### GlobalQueueEntry

Aggregated global request-queue entry (shared wishlist demand).

#### Properties

##### asin

```ts
asin: string | null;
```

##### authors

```ts
authors: string | null;
```

##### cover\_url?

```ts
optional cover_url?: string | null;
```

##### description?

```ts
optional description?: string | null;
```

##### first\_requested\_at

```ts
first_requested_at: string;
```

##### genres?

```ts
optional genres?: string | null;
```

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
optional language?: string | null;
```

##### last\_requested\_at

```ts
last_requested_at: string;
```

##### length\_minutes?

```ts
optional length_minutes?: number | null;
```

##### narrators?

```ts
optional narrators?: string | null;
```

##### published\_at?

```ts
optional published_at?: string | null;
```

##### publisher?

```ts
optional publisher?: string | null;
```

##### purchase\_hints?

```ts
optional purchase_hints?: PurchaseHint[];
```

##### reasons?

```ts
optional reasons?: string[];
```

##### sample\_uuids

```ts
sample_uuids: string[];
```

##### score?

```ts
optional score?: number;
```

Final rank (`taste_score + wish_count` boost).

##### series?

```ts
optional series?: string | null;
```

##### series\_index?

```ts
optional series_index?: string | null;
```

##### store\_editions?

```ts
optional store_editions?: StoreEdition[];
```

##### subtitle?

```ts
optional subtitle?: string | null;
```

##### taste\_score?

```ts
optional taste_score?: number;
```

##### title

```ts
title: string;
```

##### wish\_count

```ts
wish_count: number;
```

##### wishers?

```ts
optional wishers?: QueueWisher[];
```

Distinct people who have this title on an open wishlist.

##### work\_key

```ts
work_key: string;
```

***

### JobInfo

Background job row from `/api/jobs`.

#### Properties

##### attempt\_count?

```ts
optional attempt_count?: number;
```

##### created\_at?

```ts
optional created_at?: string | null;
```

##### detail

```ts
detail: string | null;
```

##### error\_kind?

```ts
optional error_kind?: string | null;
```

##### finished\_at?

```ts
optional finished_at?: string | null;
```

##### id

```ts
id: string;
```

##### kind

```ts
kind: string;
```

##### max\_attempts?

```ts
optional max_attempts?: number;
```

##### progress?

```ts
optional progress?: string | null;
```

##### status

```ts
status: string;
```

***

### LinkedIdentity

Linked portal identity (local or `oidc:{id}`).

#### Properties

##### external\_user\_id

```ts
external_user_id: string;
```

##### label

```ts
label: string | null;
```

##### provider

```ts
provider: string;
```

***

### ListedPasskey

Registered passkey row.

#### Properties

##### credential\_id

```ts
credential_id: string;
```

##### id

```ts
id: number;
```

##### name?

```ts
optional name?: string | null;
```

***

### ListedSession

Active auth session row visible to the current principal.

#### Properties

##### client\_label?

```ts
optional client_label?: string | null;
```

##### created\_at

```ts
created_at: string;
```

##### device\_type?

```ts
optional device_type?: string | null;
```

##### elevated?

```ts
optional elevated?: boolean;
```

##### expires\_at

```ts
expires_at: string;
```

##### id

```ts
id: number;
```

##### impersonating\_user\_id?

```ts
optional impersonating_user_id?: number | null;
```

##### is\_current?

```ts
optional is_current?: boolean;
```

##### kind

```ts
kind: string;
```

##### last\_used\_at?

```ts
optional last_used_at?: string | null;
```

##### user\_agent?

```ts
optional user_agent?: string | null;
```

***

### ListedUser

User row from `GET /api/users` (administrator settings).

#### Properties

##### avatar\_source?

```ts
optional avatar_source?: string | null;
```

`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`.

##### display\_name

```ts
display_name: string | null;
```

##### email?

```ts
optional email?: string | null;
```

##### gravatar\_hash?

```ts
optional gravatar_hash?: string | null;
```

SHA-256 hex of the contact email for Gravatar.

##### has\_avatar?

```ts
optional has_avatar?: boolean;
```

True when a profile picture file is stored for this user.

##### has\_password

```ts
has_password: boolean;
```

##### id

```ts
id: number;
```

##### identities?

```ts
optional identities?: object[];
```

Linked login identities (local + OIDC).

###### external\_user\_id

```ts
external_user_id: string;
```

###### label

```ts
label: string | null;
```

###### provider

```ts
provider: string;
```

##### integrations?

```ts
optional integrations?: object[];
```

Linked storefront / integration accounts.

###### account\_id

```ts
account_id: string;
```

###### label

```ts
label: string | null;
```

###### source

```ts
source: string;
```

##### last\_active\_at?

```ts
optional last_active_at?: string | null;
```

Most recent unexpired portal session activity (RFC 3339). Active vs idle is ~5 minutes.

##### last\_seen\_at?

```ts
optional last_seen_at?: string | null;
```

Durable last portal use (RFC 3339); survives logout. Null means never signed in.

##### listening?

```ts
optional listening?: 
  | {
  last_listened_at: string;
  provider: string;
  title: string | null;
}
  | null;
```

Unfinished listen within ~30 minutes (waveform uses the last ~5 minutes).

##### login\_name

```ts
login_name: string | null;
```

##### online?

```ts
optional online?: boolean;
```

Non-expired portal session present (`true` even when idle).

##### role

```ts
role: string;
```

##### sso\_pictures?

```ts
optional sso_pictures?: object[];
```

IdP-supplied pictures the user can pick.

###### identity\_id

```ts
identity_id: number;
```

###### last\_used\_at?

```ts
optional last_used_at?: string | null;
```

###### picture\_url

```ts
picture_url: string;
```

###### provider

```ts
provider: string;
```

##### status

```ts
status: string;
```

***

### NodeCapabilities

What the caller may do on the nodes it was sent.

#### Properties

##### can\_assign\_owner

```ts
can_assign_owner: boolean;
```

##### can\_edit\_labels

```ts
can_edit_labels: boolean;
```

##### can\_edit\_runtime

```ts
can_edit_runtime: boolean;
```

##### scope

```ts
scope: "all" | "owned";
```

`all` for an operator; `owned` for a user who owns nodes.

***

### NodeRuntimeView

A node's runtime document (`node.runtime`): its label, the share of its
detected capacity the scheduler may commit, and the drain switch.

#### Properties

##### cpu\_budget\_percent

```ts
cpu_budget_percent: number;
```

Share of detected CPU the scheduler may commit (1..=100).

##### label

```ts
label: string;
```

##### memory\_budget\_percent

```ts
memory_budget_percent: number;
```

Share of detected memory the scheduler may commit (1..=100).

##### schedulable

```ts
schedulable: boolean;
```

False drains the node: nothing new is placed there.

***

### NodesResponse

`GET /api/config/nodes`.

#### Extended by

- [`ClusterStatusResponse`](#clusterstatusresponse)

#### Properties

##### capabilities

```ts
capabilities: NodeCapabilities;
```

##### node\_id

```ts
node_id: string;
```

##### nodes

```ts
nodes: ConfigNodeView[];
```

***

### NodeStatusView

A node's latest heartbeat (every 15 s while its daemon runs).

#### Properties

##### availability\_ppm

```ts
availability_ppm: number;
```

Recent online fraction in parts per million (decays over about a day).

##### cpu\_budget\_millicores

```ts
cpu_budget_millicores: number;
```

##### cpu\_millicores

```ts
cpu_millicores: number;
```

Detected CPU capacity in millicores.

##### cpu\_used\_millicores

```ts
cpu_used_millicores: number | null;
```

Measured CPU in use since the previous heartbeat.

##### files\_dir\_free\_bytes

```ts
files_dir_free_bytes: number | null;
```

##### heartbeat\_at

```ts
heartbeat_at: string;
```

##### interval\_ms

```ts
interval_ms: number;
```

##### memory\_available\_bytes

```ts
memory_available_bytes: number | null;
```

Measured memory available to new work.

##### memory\_budget\_bytes

```ts
memory_budget_bytes: number | null;
```

##### memory\_bytes

```ts
memory_bytes: number | null;
```

Detected memory in bytes, when the node could read it.

##### reserved

```ts
reserved: object;
```

Sum of the running guests' jail ceilings.

###### cpu\_millicores

```ts
cpu_millicores: number;
```

###### memory\_bytes

```ts
memory_bytes: number;
```

##### running\_guests

```ts
running_guests: number;
```

##### schedulable

```ts
schedulable: boolean;
```

##### started\_at

```ts
started_at: string;
```

***

### OidcAsClient

Bookclerk-as-IdP OAuth client (secrets never listed after create/rotate).

#### Properties

##### allowed\_scopes

```ts
allowed_scopes: string[];
```

##### client\_id

```ts
client_id: string;
```

##### client\_secret?

```ts
optional client_secret?: string | null;
```

Plaintext secret, returned only on create or rotate.

##### confidential

```ts
confidential: boolean;
```

##### enabled

```ts
enabled: boolean;
```

##### has\_secret

```ts
has_secret: boolean;
```

##### issue\_refresh\_token

```ts
issue_refresh_token: boolean;
```

##### name?

```ts
optional name?: string | null;
```

##### plugin\_id?

```ts
optional plugin_id?: string | null;
```

Set when a plugin owns this client (redirects are read-only).

##### redirect\_uris

```ts
redirect_uris: string[];
```

***

### OidcAsClientWrite

Body for creating or updating a Bookclerk-as-IdP client.

#### Properties

##### allowed\_scopes

```ts
allowed_scopes: string[];
```

##### client\_id?

```ts
optional client_id?: string;
```

##### confidential

```ts
confidential: boolean;
```

##### current\_password?

```ts
optional current_password?: string;
```

##### enabled

```ts
enabled: boolean;
```

##### issue\_refresh\_token

```ts
issue_refresh_token: boolean;
```

##### name?

```ts
optional name?: string | null;
```

##### redirect\_uris

```ts
redirect_uris: string[];
```

***

### OidcBrokerConfigUpdate

Body for `PUT /api/auth/oidc/config`.

#### Properties

##### allowed\_email\_domains

```ts
allowed_email_domains: string[];
```

##### current\_password?

```ts
optional current_password?: string;
```

##### enabled

```ts
enabled: boolean;
```

##### providers

```ts
providers: OidcProviderConfigUpdate[];
```

##### public\_origin?

```ts
optional public_origin?: string | null;
```

Omit to leave unchanged; empty string clears `integrations.public_origin`.

***

### OidcBrokerConfigView

Owner/operator identity-broker settings payload.

#### Properties

##### allowed\_email\_domains

```ts
allowed_email_domains: string[];
```

##### authorization\_endpoint?

```ts
optional authorization_endpoint?: string;
```

##### callback\_url?

```ts
optional callback_url?: string | null;
```

##### detected\_origin?

```ts
optional detected_origin?: string;
```

Origin detected from this request when it is a bound loopback listen address.

##### discovery\_url?

```ts
optional discovery_url?: string;
```

Absolute OpenID discovery URL.

##### enabled

```ts
enabled: boolean;
```

##### issuer\_url?

```ts
optional issuer_url?: string;
```

Effective issuer origin (`public_origin`, else bound loopback, else localhost).

##### providers

```ts
providers: OidcProviderConfigView[];
```

##### public\_origin?

```ts
optional public_origin?: string | null;
```

Configured `integrations.public_origin` (issuer / callback / invite base).

##### revocation\_endpoint?

```ts
optional revocation_endpoint?: string;
```

##### token\_endpoint?

```ts
optional token_endpoint?: string;
```

##### userinfo\_endpoint?

```ts
optional userinfo_endpoint?: string;
```

***

### OidcProvider

Enabled upstream OIDC/OAuth login providers.

#### Properties

##### id

```ts
id: string;
```

##### name

```ts
name: string;
```

##### preset?

```ts
optional preset?: string | null;
```

Built-in social preset (`google`, `github`, `apple`, `discord`) when set.

***

### OidcProviderConfigUpdate

Provider fields accepted by `PUT /api/auth/oidc/config`.

#### Properties

##### allowed\_email\_domains

```ts
allowed_email_domains: string[];
```

##### allowed\_emails

```ts
allowed_emails: string[];
```

##### allowed\_subjects

```ts
allowed_subjects: string[];
```

##### apple\_key\_id?

```ts
optional apple_key_id?: string | null;
```

##### apple\_private\_key?

```ts
optional apple_private_key?: string;
```

##### apple\_team\_id?

```ts
optional apple_team_id?: string | null;
```

##### client\_id

```ts
client_id: string;
```

##### client\_secret?

```ts
optional client_secret?: string;
```

Omit to keep; empty string to clear; non-empty to store.

##### default\_role

```ts
default_role: string;
```

##### id

```ts
id: string;
```

##### issuer?

```ts
optional issuer?: string | null;
```

##### link\_by\_email

```ts
link_by_email: boolean;
```

##### name

```ts
name: string;
```

##### preset?

```ts
optional preset?: string | null;
```

##### provision

```ts
provision: OidcProvisionMode;
```

##### role\_claim

```ts
role_claim: string;
```

##### role\_map

```ts
role_map: Record<string, string>;
```

##### scopes

```ts
scopes: string[];
```

***

### OidcProviderConfigView

Owner/operator view of one `[auth.oidc]` provider (secrets redacted).

#### Properties

##### allowed\_email\_domains

```ts
allowed_email_domains: string[];
```

##### allowed\_emails

```ts
allowed_emails: string[];
```

##### allowed\_subjects

```ts
allowed_subjects: string[];
```

##### apple\_key\_id?

```ts
optional apple_key_id?: string | null;
```

##### apple\_team\_id?

```ts
optional apple_team_id?: string | null;
```

##### client\_id

```ts
client_id: string;
```

##### default\_role

```ts
default_role: string;
```

##### has\_apple\_private\_key?

```ts
optional has_apple_private_key?: boolean;
```

##### has\_client\_secret

```ts
has_client_secret: boolean;
```

##### id

```ts
id: string;
```

##### issuer?

```ts
optional issuer?: string | null;
```

##### link\_by\_email

```ts
link_by_email: boolean;
```

##### name

```ts
name: string;
```

##### preset?

```ts
optional preset?: string | null;
```

##### provision

```ts
provision: OidcProvisionMode;
```

##### role\_claim

```ts
role_claim: string;
```

##### role\_map

```ts
role_map: Record<string, string>;
```

##### scopes

```ts
scopes: string[];
```

##### secret\_source

```ts
secret_source: OidcSecretSource;
```

***

### PluginConsentBrand

Branded display metadata for a plugin consent dialog.

#### Properties

##### accent?

```ts
optional accent?: string | null;
```

##### bg?

```ts
optional bg?: string | null;
```

##### fg?

```ts
optional fg?: string | null;
```

##### logo?

```ts
optional logo?: string | null;
```

##### name

```ts
name: string;
```

***

### PluginConsentLimits

Host-capped consent limit defaults (workerd budgets + shared jail/disk).

Jail CPU fields are cores (2 d.p.) from the daemon — no percent conversion.

#### Properties

##### cpu\_cores

```ts
cpu_cores: number;
```

Manifest / default jail CPU in cores.

##### cpu\_ms

```ts
cpu_ms: number;
```

##### disk\_mib

```ts
disk_mib: number;
```

##### extra\_processes

```ts
extra_processes: number;
```

Default extra process/thread budget (native).

##### jail\_cpu\_cores?

```ts
optional jail_cpu_cores?: number | null;
```

Optional Settings global jail CPU ceiling in cores.

##### known\_bindings

```ts
known_bindings: string[];
```

##### max\_cpu\_cores

```ts
max_cpu_cores: number;
```

Host max jail CPU in cores.

##### max\_cpu\_ms

```ts
max_cpu_ms: number;
```

##### max\_disk\_mib

```ts
max_disk_mib: number;
```

##### max\_extra\_processes

```ts
max_extra_processes: number;
```

Host hard cap for extra process budget.

##### max\_memory\_mib

```ts
max_memory_mib: number;
```

##### max\_subrequests

```ts
max_subrequests: number;
```

##### memory\_mib

```ts
memory_mib: number;
```

##### subrequests

```ts
subrequests: number;
```

***

### PluginConsentResponse

Network/capability consent status for a plugin.

#### Properties

##### brand?

```ts
optional brand?: PluginConsentBrand;
```

##### covered

```ts
covered: boolean;
```

##### existing?

```ts
optional existing?: PluginGrant;
```

##### limits

```ts
limits: PluginConsentLimits;
```

##### plugin\_id

```ts
plugin_id: string;
```

##### plugin\_key?

```ts
optional plugin_key?: string;
```

##### provenance?

```ts
optional provenance?: string;
```

##### request

```ts
request: PluginGrant;
```

##### runtime

```ts
runtime: string;
```

Guest runtime: `native` or `workerd`.

##### summary

```ts
summary: string[];
```

***

### PluginDeploymentView

This node's deployment of a plugin instance, with its latest observation.

#### Properties

##### applied\_config\_revision?

```ts
optional applied_config_revision?: number;
```

##### deployment\_id

```ts
deployment_id: string;
```

##### desired

```ts
desired: string;
```

`present` / `absent` when an operator manages the instance (the Enabled
switch), `placed` / `unplaced` when the cluster scheduler does. The
host runs it for `present` and `placed`.

##### detail?

```ts
optional detail?: string;
```

##### node\_id

```ts
node_id: string;
```

##### observed\_at?

```ts
optional observed_at?: string;
```

##### revision

```ts
revision: number;
```

##### status?

```ts
optional status?: string;
```

`materialized`, `running`, `healthy`, `error`, or `retired` when observed.

***

### PluginGrant

Plugin consent grant shape serialized by the daemon.

#### Properties

##### addressCidrs?

```ts
optional addressCidrs?: string[];
```

##### allowUndeclaredPublicRedirects?

```ts
optional allowUndeclaredPublicRedirects?: boolean;
```

##### approvedAt

```ts
approvedAt: string;
```

##### authorityRevision?

```ts
optional authorityRevision?: string;
```

##### bindings

```ts
bindings: string[];
```

##### compatibilityFlags

```ts
compatibilityFlags: string[];
```

##### consumers?

```ts
optional consumers?: object[];
```

Approved event-consumer subscriptions (type + schemas + suspend + filter).

###### eventType

```ts
eventType: string;
```

###### filter?

```ts
optional filter?: string | null;
```

###### schemaVersions

```ts
schemaVersions: number[];
```

###### supportsSuspend

```ts
supportsSuspend: boolean;
```

##### cpuCores?

```ts
optional cpuCores?: number;
```

Jail CPU in cores (2 d.p.) from the daemon.

##### cpuMs?

```ts
optional cpuMs?: number;
```

##### diskMib?

```ts
optional diskMib?: number;
```

##### domains

```ts
domains: string[];
```

##### entrypoints

```ts
entrypoints: string[];
```

Exported entrypoints the operator approved (`storefront`, `cli`, …).

##### extraProcesses?

```ts
optional extraProcesses?: number;
```

Extra process/thread budget beyond launcher overhead (native).

##### jobs?

```ts
optional jobs?: string[];
```

Approved `[triggers].jobs` types.

##### manifestDomains?

```ts
optional manifestDomains?: string[];
```

##### memoryMib?

```ts
optional memoryMib?: number;
```

##### networkMode

```ts
networkMode: string;
```

##### operatorAddedCidrs?

```ts
optional operatorAddedCidrs?: string[];
```

##### operatorAddedDomains?

```ts
optional operatorAddedDomains?: string[];
```

##### operatorAddedTcp?

```ts
optional operatorAddedTcp?: object[];
```

###### host

```ts
host: string;
```

###### ports

```ts
ports: number[];
```

##### operatorDeniedCidrs?

```ts
optional operatorDeniedCidrs?: string[];
```

##### operatorDeniedDomains?

```ts
optional operatorDeniedDomains?: string[];
```

##### operatorDeniedTcp?

```ts
optional operatorDeniedTcp?: object[];
```

###### host

```ts
host: string;
```

###### ports

```ts
ports: number[];
```

##### pluginId

```ts
pluginId: string;
```

##### pluginKey?

```ts
optional pluginKey?: string;
```

##### producers

```ts
producers: string[];
```

Event types the operator approved the plugin to publish.

##### subrequests?

```ts
optional subrequests?: number;
```

##### tcp?

```ts
optional tcp?: object[];
```

###### host

```ts
host: string;
```

###### ports

```ts
ports: number[];
```

***

### PluginInstanceConfig

Instance document as `GET /api/config/plugin-instances/{id}/config` returns it.

#### Properties

##### effective

```ts
effective: Record<string, unknown>;
```

Schema defaults under the stored settings.

##### plugin\_instance\_id

```ts
plugin_instance_id: string;
```

##### plugin\_key

```ts
plugin_key: string;
```

##### revision

```ts
revision: number;
```

##### secret\_refs

```ts
secret_refs: object[];
```

Secret refs (names only).

###### key

```ts
key: string;
```

###### name

```ts
name: string;
```

##### settings

```ts
settings: Record<string, string | number | boolean>;
```

Stored settings (operator writes only; defaults are not stored).

***

### PluginInstanceConfigWrite

Body for `PUT /api/config/plugin-instances/{id}/config`.

#### Properties

##### expected\_revision

```ts
expected_revision: number;
```

##### secret\_refs

```ts
secret_refs: object[];
```

###### key

```ts
key: string;
```

###### name

```ts
name: string;
```

##### secrets?

```ts
optional secrets?: Record<string, string>;
```

Secret values to seal; never echoed back.

##### settings

```ts
settings: Record<string, string | number | boolean>;
```

***

### PluginPlacementView

Scheduler placement of a plugin instance: the operator's intent and the
last scheduler pass's result.

#### Properties

##### constraints

```ts
constraints: string[];
```

Label constraints (`nas`, `zone=eu`, `!slow`, `host=<id>`).

##### detail

```ts
detail: string;
```

Why fewer than requested are placed; empty when satisfied.

##### placed

```ts
placed: number;
```

Replicas the last scheduler pass placed.

##### replicas

```ts
replicas: number | "all";
```

Nodes that should run the instance: a count (0 stops every replica) or
`"all"` (every live, schedulable host with budget room).

##### revision

```ts
revision: number;
```

Compare-and-swap revision of the intent.

##### scheduled\_at

```ts
scheduled_at: string;
```

##### updated\_at

```ts
updated_at: string;
```

***

### PluginSettingChoice

Select option for a plugin setting.

#### Properties

##### label

```ts
label: string;
```

##### value

```ts
value: string;
```

***

### PluginSettingOption

Editable plugin setting descriptor from the settings API.

For a plugin instance group the key is the bare `[[settings]]` key from the
plugin's manifest and `value` is the effective value (document over schema
default). A `secret` option never carries its value.

#### Properties

##### choices?

```ts
optional choices?: PluginSettingChoice[];
```

##### key

```ts
key: string;
```

##### label

```ts
label: string;
```

##### secret?

```ts
optional secret?: boolean;
```

True for a `secret` setting; `value` is always empty.

##### secret\_set?

```ts
optional secret_set?: boolean;
```

For a secret setting, whether a sealed value is stored for the instance.

##### value

```ts
value: string;
```

##### value\_type

```ts
value_type: "string" | "number" | "boolean" | "secret";
```

***

### PluginSettingsGroup

Settings group for one plugin instance, or for a bootstrap database adapter.

#### Properties

##### config\_revision?

```ts
optional config_revision?: number;
```

Instance document revision for compare-and-swap writes.

##### deployment?

```ts
optional deployment?: PluginDeploymentView;
```

This node's deployment of the instance, when one exists.

##### display\_name?

```ts
optional display_name?: string;
```

Operator-facing instance name, when set.

##### family

```ts
family: string;
```

Handler family (`source`, `integration`, `output`, `database`).

##### id

```ts
id: string;
```

##### is\_default?

```ts
optional is_default?: boolean;
```

True when a bare plugin key or alias routes to this instance.

##### logo?

```ts
optional logo?: string;
```

Google favicon (or portal brand) URL for Settings list rows.

##### placement?

```ts
optional placement?: PluginPlacementView;
```

Scheduler placement; absent when the instance is managed by hand.

##### plugin\_instance\_id?

```ts
optional plugin_instance_id?: string;
```

Instance this group edits; absent for a `config.toml`-backed database adapter.

##### plugin\_key?

```ts
optional plugin_key?: string;
```

Provenance-qualified PluginKey when this row came from a discovered install.

##### present\_on?

```ts
optional present_on?: string[];
```

Nodes whose deployment of the instance is `present`.

##### provenance?

```ts
optional provenance?: string;
```

Host-evaluated provenance (`platform_bundled`, `verified_staged`, `installed`, …).

##### settings

```ts
settings: PluginSettingOption[];
```

***

### PortalBrand

Storefront brand colors and logo URL for Accounts UI.

#### Properties

##### accent

```ts
accent: string;
```

##### bg

```ts
bg: string;
```

##### fg

```ts
fg: string;
```

##### logo

```ts
logo: string;
```

***

### PortalConnection

Linked store account for the current portal user.

#### Properties

##### account\_id

```ts
account_id: string;
```

##### brand?

```ts
optional brand?: PortalBrand;
```

##### connection\_status

```ts
connection_status: string;
```

##### label

```ts
label: string | null;
```

##### plugin\_key

```ts
plugin_key: string;
```

Provenance-qualified PluginKey that owns the account.

##### source

```ts
source: string;
```

##### source\_enabled

```ts
source_enabled: boolean;
```

***

### PortalInfo

Linked portal identity attached to a user session.

#### Properties

##### external\_user\_id

```ts
external_user_id: string;
```

##### identity\_id

```ts
identity_id: number;
```

##### label

```ts
label: string | null;
```

##### provider

```ts
provider: string;
```

***

### PortalMe

Portal identity from `/api/portal/me`.

#### Properties

##### external\_user\_id

```ts
external_user_id: string;
```

##### label

```ts
label: string | null;
```

##### provider

```ts
provider: string;
```

***

### PortalSource

Connectable storefront advertised by `/api/portal/sources`.

#### Properties

##### auth

```ts
auth: string;
```

##### brand

```ts
brand: PortalBrand;
```

##### id

```ts
id: string;
```

##### name

```ts
name: string;
```

***

### PurchaseHint

Live or seeded storefront price / buy URL for a title.

#### Properties

##### currency?

```ts
optional currency?: string | null;
```

##### list\_price\_cents?

```ts
optional list_price_cents?: number | null;
```

##### list\_price\_label?

```ts
optional list_price_label?: string | null;
```

##### member\_price\_cents?

```ts
optional member_price_cents?: number | null;
```

##### member\_price\_label?

```ts
optional member_price_label?: string | null;
```

##### price\_cents?

```ts
optional price_cents?: number | null;
```

##### price\_label?

```ts
optional price_label?: string | null;
```

##### product\_id

```ts
product_id: string;
```

##### source

```ts
source: string;
```

##### title

```ts
title: string | null;
```

##### url

```ts
url: string | null;
```

***

### PurchaseHintsQuery

Query used to resolve purchase hints for a title.

#### Properties

##### asin?

```ts
optional asin?: string | null;
```

##### authors?

```ts
optional authors?: string | null;
```

##### candidate\_product\_id?

```ts
optional candidate_product_id?: string | null;
```

##### candidate\_source?

```ts
optional candidate_source?: string | null;
```

##### isbn?

```ts
optional isbn?: string | null;
```

##### region?

```ts
optional region?: string;
```

##### store\_editions?

```ts
optional store_editions?: StoreEdition[];
```

##### title

```ts
title: string;
```

***

### PurchaseHintsResponse

Purchase hint list plus optional best pick.

#### Properties

##### best

```ts
best: PurchaseHint | null;
```

##### hints

```ts
hints: PurchaseHint[];
```

***

### QueueWisher

Person who has a title on an open wishlist (global queue avatars).

#### Properties

##### avatar\_source?

```ts
optional avatar_source?: string | null;
```

Explicit picture choice (`auto` / `monogram` / `gravatar` / `upload` / `sso:{id}`).

##### display\_name?

```ts
optional display_name?: string | null;
```

Display name, identity label, or Operator.

##### gravatar\_hash?

```ts
optional gravatar_hash?: string | null;
```

SHA-256 hex of the contact email for Gravatar.

##### has\_avatar?

```ts
optional has_avatar?: boolean;
```

True when an uploaded avatar file exists.

##### identity\_id?

```ts
optional identity_id?: number | null;
```

Portal identity that created the wish.

##### login\_name?

```ts
optional login_name?: string | null;
```

Local login name when set.

##### operator?

```ts
optional operator?: boolean;
```

True when the wish came from the operator token.

##### picture\_url?

```ts
optional picture_url?: string | null;
```

HTTPS picture from the wishing identity, when any.

##### user\_id?

```ts
optional user_id?: number | null;
```

First-party user id when the portal identity is linked.

***

### Recommendation

Discover shelf recommendation item.

#### Properties

##### asin

```ts
asin: string | null;
```

##### authors

```ts
authors: string | null;
```

##### candidate\_product\_id

```ts
candidate_product_id: string | null;
```

##### candidate\_source

```ts
candidate_source: string | null;
```

##### categories?

```ts
optional categories?: string[];
```

Stable shelf-kind tags (`finish_series`, `author`, `requests`, …).

##### cover\_url?

```ts
optional cover_url?: string | null;
```

##### description?

```ts
optional description?: string | null;
```

##### from\_request

```ts
from_request: boolean;
```

##### genres?

```ts
optional genres?: string | null;
```

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
optional language?: string | null;
```

##### length\_minutes?

```ts
optional length_minutes?: number | null;
```

##### narrators

```ts
narrators: string | null;
```

##### published\_at?

```ts
optional published_at?: string | null;
```

##### publisher?

```ts
optional publisher?: string | null;
```

##### purchase\_hints

```ts
purchase_hints: PurchaseHint[];
```

##### reasons

```ts
reasons: string[];
```

##### request\_uuid

```ts
request_uuid: string | null;
```

##### score

```ts
score: number;
```

##### seed\_categories?

```ts
optional seed_categories?: string | null;
```

##### series

```ts
series: string | null;
```

##### series\_index

```ts
series_index: string | null;
```

##### store\_editions?

```ts
optional store_editions?: StoreEdition[];
```

##### subtitle?

```ts
optional subtitle?: string | null;
```

##### title

```ts
title: string;
```

##### wish\_count?

```ts
optional wish_count?: number;
```

Distinct wisher count (may exceed `wishers.length`).

##### wishers?

```ts
optional wishers?: QueueWisher[];
```

People who have this title on an open wishlist.

##### work\_id

```ts
work_id: string | null;
```

##### work\_key?

```ts
optional work_key?: string;
```

Stable bibliographic key (`isbn:` / `asin:` / `soft:`…).

***

### RemoteApprovalBody

Body of an approval: the reviewed request and what of it to leave out.
An approval can only narrow the request, never widen it.

#### Properties

##### cluster\_id

```ts
cluster_id: string;
```

Owner cluster of the request.

##### deny\_bindings?

```ts
optional deny_bindings?: string[];
```

Host bindings to remove (`secrets`, `oauth`, `kv`, …).

##### deny\_domains?

```ts
optional deny_domains?: string[];
```

Outbound domain patterns to remove from the grant.

##### deny\_network?

```ts
optional deny_network?: boolean;
```

True turns outbound network off altogether (no domains, TCP or CIDRs).

##### plugin\_key

```ts
plugin_key: string;
```

Plugin key of the request.

##### request\_digest

```ts
request_digest: string;
```

The `request_digest` reviewed; refused when the recorded request is another one.

***

### RemoteConsentView

One owner plugin's request to run on this host, and this host's answer.

#### Properties

##### approved\_at

```ts
approved_at: string | null;
```

RFC 3339 time of the approval, if approved.

##### approved\_by

```ts
approved_by: string | null;
```

Who approved: the operator's audit identity, or `platform` for an automatic approval.

##### approved\_version

```ts
approved_version: string | null;
```

Version this host approved, if any.

##### cluster\_id

```ts
cluster_id: string;
```

Owner cluster that asks; matches a [ClusterLinkView.cluster\_id](#cluster_id).

##### grant

```ts
grant: RemoteGrantView | null;
```

What this host granted (the request, narrowed), if approved.

##### plugin\_key

```ts
plugin_key: string;
```

Canonical plugin key the request is for.

##### request\_digest

```ts
request_digest: string;
```

Digest of the whole request (version, grant, archive); approve by it.

##### requested\_artifact?

```ts
optional requested_artifact?: object;
```

The archive this host would fetch for that version (hex SHA-256 digests).

###### archive\_sha256

```ts
archive_sha256: string;
```

Digest of the archive file.

###### payload\_root\_sha256

```ts
payload_root_sha256: string;
```

Digest of the unpacked payload tree.

###### url

```ts
url: string;
```

`blob:` (served by the owner) or `https:` archive URL.

##### requested\_grant

```ts
requested_grant: RemoteGrantView | null;
```

What that version asks for; `null` when the stored request does not parse.

##### requested\_version

```ts
requested_version: string;
```

Version the owner now asks to run.

##### revoked\_at?

```ts
optional revoked_at?: string | null;
```

RFC 3339 time this host's operator revoked it; a revoke holds until approved again.

##### state

```ts
state: RemoteConsentState;
```

Whether the guest may run here now, or what it waits on.

##### updated\_at?

```ts
optional updated_at?: string;
```

RFC 3339 time the row last changed.

***

### RemoteGrantView

The parts of a plugin grant the consent screen shows. Every field is
optional because the screen renders whatever the host sent; a missing set
means the grant asks for none of it, and a missing or `null` limit leaves
that limit to the version's manifest.

#### Properties

##### addressCidrs?

```ts
optional addressCidrs?: string[];
```

CIDR ranges beyond the public Internet (private or local addresses).

##### allowUndeclaredPublicRedirects?

```ts
optional allowUndeclaredPublicRedirects?: boolean;
```

Follow a redirect to a public host it did not declare.

##### bindings?

```ts
optional bindings?: string[];
```

Host bindings (`config`, `secrets`, `kv`, …).

##### compatibilityFlags?

```ts
optional compatibilityFlags?: string[];
```

workerd compatibility flags the isolate runs with.

##### consumers?

```ts
optional consumers?: unknown[];
```

Events and queues it consumes (shape owned by the host).

##### cpuMs?

```ts
optional cpuMs?: number | null;
```

workerd CPU budget per request, in milliseconds.

##### cpuRatePercent?

```ts
optional cpuRatePercent?: number | null;
```

CPU rate as a percent of one logical CPU (above 100 asks for more than one core).

##### diskMib?

```ts
optional diskMib?: number | null;
```

Disk budget in MiB for each of its `data/` and `tmp/` directories.

##### domains?

```ts
optional domains?: string[];
```

Outbound domain patterns the guest may dial first (redirect hops aside).

##### entrypoints?

```ts
optional entrypoints?: string[];
```

Named entrypoints it exports (`storefront`, `storage`, `cli`, …).

##### extraProcesses?

```ts
optional extraProcesses?: number | null;
```

Processes or threads allowed beyond the launcher's own (native guests).

##### jobs?

```ts
optional jobs?: string[];
```

`[triggers].jobs` types it may handle.

##### memoryMib?

```ts
optional memoryMib?: number | null;
```

Memory ceiling in MiB.

##### networkMode?

```ts
optional networkMode?: string;
```

`deny` (no egress) or `outbound`; anything else is treated as `deny`.

##### pluginId?

```ts
optional pluginId?: string;
```

Display alias from `plugin.toml` (not security authoritative).

##### producers?

```ts
optional producers?: string[];
```

Event types it may publish and queue bindings it may send to.

##### subrequests?

```ts
optional subrequests?: number | null;
```

workerd outbound fetch (subrequest) budget per request.

##### tcp?

```ts
optional tcp?: object[];
```

Raw TCP destinations: a host and the ports allowed on it.

###### host

```ts
host: string;
```

###### ports

```ts
ports: number[];
```

***

### SettingsResponse

Flat config map plus per-plugin setting groups.

#### Properties

##### effective?

```ts
optional effective?: Record<string, string>;
```

##### host\_cpu\_cores\_max?

```ts
optional host_cpu_cores_max?: number;
```

Host max jail CPU in cores (2 d.p.; equals logical CPU count).

##### jail\_cpu\_cores?

```ts
optional jail_cpu_cores?: number | null;
```

Optional global per-jail CPU ceiling in cores.

##### output\_revision?

```ts
optional output_revision?: number;
```

`core.output` document revision the `output.*` values were read from; send it
as `expected_output_revision`. `effective["output.revision"]` is the revision
this host has applied, which can lag the document. Absent when the server
could not read `core.output`; the `output.*` values are then missing too and
must not be edited.

##### plugins

```ts
plugins: PluginSettingsGroup[];
```

##### settings

```ts
settings: Record<string, string>;
```

***

### SettingsUpdate

One config key/value pair for `PATCH /api/settings`.

#### Properties

##### key

```ts
key: string;
```

##### value

```ts
value: string;
```

***

### ShelfKindInfo

Shelf kind id/label for preference toggles.

#### Properties

##### id

```ts
id: string;
```

##### label

```ts
label: string;
```

***

### StatusResponse

Aggregate library counters from `/api/status`.

#### Properties

##### accounts

```ts
accounts: number;
```

##### acquired

```ts
acquired: number;
```

##### books

```ts
books: number;
```

##### error

```ts
error: number;
```

##### events?

```ts
optional events?: EventQueueStatus;
```

##### in\_progress

```ts
in_progress: number;
```

##### listen

```ts
listen: string;
```

##### pending

```ts
pending: number;
```

##### storage\_backend

```ts
storage_backend: string;
```

***

### StoreEdition

Storefront product identity for a catalog work.

#### Properties

##### product\_id

```ts
product_id: string;
```

##### source

```ts
source: string;
```

***

### TitleMeta

Public Audnexus / Audible catalog fields for detail dialogs.

#### Properties

##### asin?

```ts
optional asin?: string | null;
```

##### authors?

```ts
optional authors?: string | null;
```

##### categories?

```ts
optional categories?: string | null;
```

##### cover\_url?

```ts
optional cover_url?: string | null;
```

##### description?

```ts
optional description?: string | null;
```

##### is\_abridged?

```ts
optional is_abridged?: boolean | null;
```

`true` abridged / `false` unabridged when the storefront said so.

##### isbn?

```ts
optional isbn?: string | null;
```

##### language?

```ts
optional language?: string | null;
```

##### length\_minutes?

```ts
optional length_minutes?: number | null;
```

##### narrators?

```ts
optional narrators?: string | null;
```

##### published\_at?

```ts
optional published_at?: string | null;
```

##### publisher?

```ts
optional publisher?: string | null;
```

##### rating\_count?

```ts
optional rating_count?: number | null;
```

##### rating\_overall?

```ts
optional rating_overall?: number | null;
```

Audible community ratings (detail fetch only).

##### rating\_performance?

```ts
optional rating_performance?: number | null;
```

##### rating\_story?

```ts
optional rating_story?: number | null;
```

##### review\_count?

```ts
optional review_count?: number | null;
```

##### ~~reviews?~~

```ts
optional reviews?: TitleReview[];
```

###### Deprecated

Prefer [`fetchTitleReviews`]. Title-meta no longer embeds reviews.

##### series?

```ts
optional series?: string | null;
```

##### series\_index?

```ts
optional series_index?: string | null;
```

##### subtitle?

```ts
optional subtitle?: string | null;
```

##### title?

```ts
optional title?: string | null;
```

***

### TitleRequest

Wishlist / title-request row from `/api/wishlist`.

Represents a user ask for a title (and optional acquire queue linkage via
`resolved_book_uuid` / `work_key`).

#### Properties

##### asin

```ts
asin: string | null;
```

##### authors

```ts
authors: string | null;
```

##### cover\_url?

```ts
optional cover_url?: string | null;
```

##### created\_at

```ts
created_at: string;
```

##### description?

```ts
optional description?: string | null;
```

##### genres?

```ts
optional genres?: string | null;
```

##### id

```ts
id: number;
```

##### identity\_id

```ts
identity_id: number | null;
```

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
optional language?: string | null;
```

##### length\_minutes?

```ts
optional length_minutes?: number | null;
```

##### narrators?

```ts
optional narrators?: string | null;
```

##### notes

```ts
notes: string | null;
```

##### published\_at?

```ts
optional published_at?: string | null;
```

##### publisher?

```ts
optional publisher?: string | null;
```

##### purchase\_hints?

```ts
optional purchase_hints?: PurchaseHint[];
```

##### resolved\_book\_uuid

```ts
resolved_book_uuid: string | null;
```

##### series?

```ts
optional series?: string | null;
```

##### series\_index?

```ts
optional series_index?: string | null;
```

##### status

```ts
status: string;
```

##### store\_editions?

```ts
optional store_editions?: StoreEdition[];
```

##### subtitle?

```ts
optional subtitle?: string | null;
```

##### title

```ts
title: string;
```

##### updated\_at

```ts
updated_at: string;
```

##### uuid

```ts
uuid: string;
```

##### work\_id

```ts
work_id: string | null;
```

##### work\_key

```ts
work_key: string;
```

***

### TitleReview

One Audible customer review for title detail.

#### Properties

##### author\_name?

```ts
optional author_name?: string | null;
```

##### body

```ts
body: string;
```

##### id?

```ts
optional id?: string | null;
```

##### overall\_rating?

```ts
optional overall_rating?: number | null;
```

##### performance\_rating?

```ts
optional performance_rating?: number | null;
```

##### story\_rating?

```ts
optional story_rating?: number | null;
```

##### submitted\_at?

```ts
optional submitted_at?: string | null;
```

##### title?

```ts
optional title?: string | null;
```

***

### UserMutationResponse

Created / patched user response from RBAC provisioning APIs.

#### Properties

##### claim\_ticket?

```ts
optional claim_ticket?: string | null;
```

##### invite\_url?

```ts
optional invite_url?: string | null;
```

Magic-link URL for `/invite?ticket=…` when an invite was minted.

##### ok

```ts
ok: boolean;
```

##### user

```ts
user: ListedUser;
```

***

### UserPreferences

Per-user Discover / default-view preferences.

#### Properties

##### default\_view

```ts
default_view: AppView;
```

##### disabled\_shelves

```ts
disabled_shelves: string[];
```

##### discover\_excluded\_sources

```ts
discover_excluded_sources: string[];
```

Store ids hidden in Discover. Empty = all sources including future.

##### discover\_language

```ts
discover_language: string | null;
```

`null` / omitted = browser language; `__all__` = no hard filter.

##### discover\_sort

```ts
discover_sort: CatalogSearchSort;
```

##### discover\_sort\_dir

```ts
discover_sort_dir: "desc" | "asc";
```

##### theme

```ts
theme: ThemePreference;
```

Appearance: `system` follows the OS (fallback light); `light` / `dark` pin it.

## Type Aliases

### AcquireStatus

```ts
type AcquireStatus = "not_acquired" | "queued" | "downloading" | "acquired" | "error";
```

Pipeline acquire state for a library book row.

***

### AppView

```ts
type AppView = "discover" | "library" | "accounts" | "wishlist" | "status" | "settings";
```

Primary authenticated SPA views mirrored in the URL path.

***

### AuthRole

```ts
type AuthRole = "operator" | "owner" | "administrator" | "host" | "member";
```

Session role returned by the daemon auth APIs.

***

### CatalogSearchFilters

```ts
type CatalogSearchFilters = object;
```

Server-side include/exclude filters for catalog search.

#### Properties

##### authors?

```ts
optional authors?: string[];
```

##### exclude\_narrators?

```ts
optional exclude_narrators?: string[];
```

##### exclude\_sources?

```ts
optional exclude_sources?: string[];
```

##### genres?

```ts
optional genres?: string[];
```

##### languages?

```ts
optional languages?: string[];
```

##### max\_length\_minutes?

```ts
optional max_length_minutes?: number;
```

##### min\_length\_minutes?

```ts
optional min_length_minutes?: number;
```

##### min\_rating?

```ts
optional min_rating?: number;
```

##### narrators?

```ts
optional narrators?: string[];
```

##### series?

```ts
optional series?: string[];
```

##### sources?

```ts
optional sources?: string[];
```

***

### CatalogSearchSort

```ts
type CatalogSearchSort = 
  | "relevance"
  | "popularity"
  | "rating"
  | "title"
  | "author"
  | "price"
  | "length";
```

Server-supported Discover catalog sort keys.

***

### CatalogSortDir

```ts
type CatalogSortDir = "asc" | "desc";
```

Ascending or descending catalog sort direction.

***

### OidcProvisionMode

```ts
type OidcProvisionMode = "mapped_role" | "any" | "allowlist" | "invite_only";
```

How an upstream IdP decides who may become a first-party User.

***

### OidcSecretSource

```ts
type OidcSecretSource = "env" | "config" | "store" | "none";
```

Where a provider client secret is stored (never returned in plaintext).

***

### RemoteConsentState

```ts
type RemoteConsentState = 
  | "consented"
  | "awaiting_host_consent"
  | "awaiting_host_reapproval"
  | "invalid_grant";
```

Where an owner plugin's request to run here stands.

***

### TitleMetaQuery

```ts
type TitleMetaQuery = object;
```

Query for public title metadata enrichment.

#### Properties

##### asin?

```ts
optional asin?: string | null;
```

##### authors?

```ts
optional authors?: string | null;
```

##### isbn?

```ts
optional isbn?: string | null;
```

##### length\_minutes?

```ts
optional length_minutes?: number | null;
```

##### narrators?

```ts
optional narrators?: string | null;
```

##### region?

```ts
optional region?: string;
```

##### title

```ts
title: string;
```

***

### TitleReviewsPage

```ts
type TitleReviewsPage = object;
```

One page of Audible customer reviews.

#### Properties

##### asin

```ts
asin: string;
```

##### has\_more

```ts
has_more: boolean;
```

##### page

```ts
page: number;
```

##### page\_size

```ts
page_size: number;
```

##### reviews

```ts
reviews: TitleReview[];
```

##### sort\_by?

```ts
optional sort_by?: string;
```

***

### TitleReviewsQuery

```ts
type TitleReviewsQuery = object;
```

Paginated title-reviews request body.

#### Properties

##### asin

```ts
asin: string;
```

##### page?

```ts
optional page?: number;
```

##### page\_size?

```ts
optional page_size?: number;
```

##### region?

```ts
optional region?: string;
```

##### sort\_by?

```ts
optional sort_by?: TitleReviewsSort;
```

***

### TitleReviewsSort

```ts
type TitleReviewsSort = "MostHelpful" | "MostRecent";
```

Audible review sort modes for title detail.

## Variables

### endElevate

```ts
const endElevate: () => Promise<void> = endElevation;
```

Alias used by Settings RBAC controls.

Ends an elevated operator window on a user session.

#### Returns

`Promise`\<`void`\>

Resolves when elevation is cleared.

## Functions

### ackEventDelivery()

```ts
function ackEventDelivery(id): Promise<ActionResponse>;
```

Acknowledges a dead-lettered delivery (no further retry).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Delivery id (`{eventId}:{pluginId}`). |

#### Returns

`Promise`\<[`ActionResponse`](#actionresponse)\>

Action ack from the daemon.

***

### approveClusterConsent()

```ts
function approveClusterConsent(body): Promise<{
  consent: RemoteConsentView;
}>;
```

Approves an owner plugin's request, optionally narrowed. Only what the
request asks for can be approved.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | [`RemoteApprovalBody`](#remoteapprovalbody) | The request reviewed and what to leave out. |

#### Returns

`Promise`\<\{
  `consent`: [`RemoteConsentView`](#remoteconsentview);
\}\>

The consent row after approval.

***

### approvePluginConsent()

```ts
function approvePluginConsent(
   id, 
   grant?, 
   pluginKey?
): Promise<PluginConsentResponse>;
```

Approves network consent for a plugin.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Display alias from Settings. |
| `grant?` | \{ `addressCidrs?`: `string`[]; `allowUndeclaredPublicRedirects?`: `boolean`; `bindings`: `string`[]; `compatibilityFlags`: `string`[]; `cpuCores?`: `number`; `cpuMs?`: `number`; `diskMib?`: `number`; `domains`: `string`[]; `extraProcesses?`: `number`; `memoryMib?`: `number`; `networkMode`: `string`; `subrequests?`: `number`; `tcp?`: `object`[]; \} | Operator-approved grant overlay; omitted fields keep defaults. |
| `grant.addressCidrs?` | `string`[] | - |
| `grant.allowUndeclaredPublicRedirects?` | `boolean` | - |
| `grant.bindings` | `string`[] | - |
| `grant.compatibilityFlags` | `string`[] | - |
| `grant.cpuCores?` | `number` | - |
| `grant.cpuMs?` | `number` | - |
| `grant.diskMib?` | `number` | - |
| `grant.domains` | `string`[] | - |
| `grant.extraProcesses?` | `number` | - |
| `grant.memoryMib?` | `number` | - |
| `grant.networkMode` | `string` | - |
| `grant.subrequests?` | `number` | - |
| `grant.tcp?` | `object`[] | - |
| `pluginKey?` | `string` | Provenance-qualified PluginKey when the alias is ambiguous. |

#### Returns

`Promise`\<[`PluginConsentResponse`](#pluginconsentresponse)\>

Updated consent response after approval.

***

### authMe()

```ts
function authMe(): Promise<AuthSession>;
```

Loads the current session from `GET /api/auth/me`.

#### Returns

`Promise`\<[`AuthSession`](#authsession)\>

Anonymous session when the response is 401.

***

### bootstrapAdministrator()

```ts
function bootstrapAdministrator(body): Promise<BootstrapAdministratorResponse>;
```

Creates the first Owner account when no owners (or legacy administrators) exist.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `display_name?`: `string`; `email?`: `string`; `login_name?`: `string`; `password?`: `string`; \} | Optional display/login/email/password fields for the new owner. |
| `body.display_name?` | `string` | - |
| `body.email?` | `string` | - |
| `body.login_name?` | `string` | - |
| `body.password?` | `string` | - |

#### Returns

`Promise`\<[`BootstrapAdministratorResponse`](#bootstrapadministratorresponse)\>

Bootstrap result including the one-time invite magic link.

***

### cancelEventDelivery()

```ts
function cancelEventDelivery(id): Promise<ActionResponse>;
```

Cancels a pending delivery or flags a running one.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Delivery id (`{eventId}:{pluginId}`). |

#### Returns

`Promise`\<[`ActionResponse`](#actionresponse)\>

Action ack from the daemon.

***

### cancelJob()

```ts
function cancelJob(id): Promise<ActionResponse>;
```

Requests cancel of a pending job or cooperative stop of a running job.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Durable job id from `/api/jobs`. |

#### Returns

`Promise`\<[`ActionResponse`](#actionresponse)\>

Action ack including the job id.

***

### coverUrl()

```ts
function coverUrl(uuid): string;
```

Builds the authenticated cover image URL for a library book.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `uuid` | `string` | Book uuid. |

#### Returns

`string`

Relative URL under `/api/library/books/.../cover`.

***

### createOidcClient()

```ts
function createOidcClient(body): Promise<OidcAsClient>;
```

Creates an OIDC client. Confidential clients return `client_secret` once.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | [`OidcAsClientWrite`](#oidcasclientwrite) | Client id, redirects, token policy. |

#### Returns

`Promise`\<[`OidcAsClient`](#oidcasclient)\>

Created client; confidential clients include `client_secret` once.

***

### createUser()

```ts
function createUser(body): Promise<UserMutationResponse>;
```

Creates a first-party Bookclerk user.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `display_name?`: `string`; `email?`: `string`; `login_name?`: `string`; `mint_invite?`: `boolean`; `password?`: `string`; `role?`: `string`; \} | Role, profile fields, optional password, and invite choice. |
| `body.display_name?` | `string` | - |
| `body.email?` | `string` | - |
| `body.login_name?` | `string` | - |
| `body.mint_invite?` | `boolean` | - |
| `body.password?` | `string` | - |
| `body.role?` | `string` | - |

#### Returns

`Promise`\<[`UserMutationResponse`](#usermutationresponse)\>

Created user plus optional one-time claim ticket.

***

### createWishlistItem()

```ts
function createWishlistItem(body): Promise<TitleRequest>;
```

Adds a title to the current user's wishlist.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `asin?`: `string`; `authors?`: `string`; `cover_url?`: `string` \| `null`; `description?`: `string` \| `null`; `genres?`: `string` \| `null`; `isbn?`: `string`; `language?`: `string` \| `null`; `length_minutes?`: `number` \| `null`; `narrators?`: `string` \| `null`; `notes?`: `string`; `published_at?`: `string` \| `null`; `publisher?`: `string` \| `null`; `purchase_hints?`: [`PurchaseHint`](#purchasehint)[]; `series?`: `string` \| `null`; `series_index?`: `string` \| `null`; `store_editions?`: [`StoreEdition`](#storeedition)[]; `subtitle?`: `string` \| `null`; `title`: `string`; `work_key?`: `string`; \} | Title fields and optional catalog enrichment. |
| `body.asin?` | `string` | - |
| `body.authors?` | `string` | - |
| `body.cover_url?` | `string` \| `null` | - |
| `body.description?` | `string` \| `null` | - |
| `body.genres?` | `string` \| `null` | - |
| `body.isbn?` | `string` | - |
| `body.language?` | `string` \| `null` | - |
| `body.length_minutes?` | `number` \| `null` | - |
| `body.narrators?` | `string` \| `null` | - |
| `body.notes?` | `string` | - |
| `body.published_at?` | `string` \| `null` | - |
| `body.publisher?` | `string` \| `null` | - |
| `body.purchase_hints?` | [`PurchaseHint`](#purchasehint)[] | - |
| `body.series?` | `string` \| `null` | - |
| `body.series_index?` | `string` \| `null` | - |
| `body.store_editions?` | [`StoreEdition`](#storeedition)[] | - |
| `body.subtitle?` | `string` \| `null` | - |
| `body.title` | `string` | - |
| `body.work_key?` | `string` | - |

#### Returns

`Promise`\<[`TitleRequest`](#titlerequest)\>

Created title-request row.

***

### deleteOidcClient()

```ts
function deleteOidcClient(clientId, currentPassword?): Promise<void>;
```

Deletes a registered OIDC client.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `clientId` | `string` | Client to remove. |
| `currentPassword?` | `string` | Owner re-auth when the session is older than 15 minutes. |

#### Returns

`Promise`\<`void`\>

Resolves when the client is removed.

***

### deletePasskey()

```ts
function deletePasskey(id, currentPassword?): Promise<void>;
```

Delete a passkey owned by the current User.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | - |
| `currentPassword?` | `string` | - |

#### Returns

`Promise`\<`void`\>

***

### deletePluginPlacement()

```ts
function deletePluginPlacement(instanceId): Promise<{
  deleted: boolean;
  plugin_instance_id: string;
}>;
```

Drops an instance's placement: its deployments stay as they are and the
Enabled switch manages them again.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `instanceId` | `string` | Plugin instance id. |

#### Returns

`Promise`\<\{
  `deleted`: `boolean`;
  `plugin_instance_id`: `string`;
\}\>

Whether a placement was removed.

***

### deleteProfileAvatar()

```ts
function deleteProfileAvatar(): Promise<void>;
```

Removes the signed-in user's stored profile picture.

#### Returns

`Promise`\<`void`\>

***

### deleteUser()

```ts
function deleteUser(id): Promise<{
  ok: boolean;
}>;
```

Deletes a first-party user (operator / administrator provisioner).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | User id. |

#### Returns

`Promise`\<\{
  `ok`: `boolean`;
\}\>

Ack payload.

***

### disableTotp()

```ts
function disableTotp(currentPassword?): Promise<void>;
```

Disables TOTP for the current user.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `currentPassword?` | `string` | Required when the portal session is older than the reauth window. |

#### Returns

`Promise`\<`void`\>

Resolves when TOTP is disabled.

***

### elevate()

```ts
function elevate(password): Promise<void>;
```

Temporarily elevates an Owner session after password re-authentication.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `password` | `string` | The Owner's account password. |

#### Returns

`Promise`\<`void`\>

Resolves when elevation succeeds.

***

### endElevation()

```ts
function endElevation(): Promise<void>;
```

Ends an elevated operator window on a user session.

#### Returns

`Promise`\<`void`\>

Resolves when elevation is cleared.

***

### fetchBooks()

```ts
function fetchBooks(params): Promise<BooksResponse>;
```

Fetches one page of library books for the Library table.

Omits `status` (or passes `all`) to include every acquire state. Default page
size is 40 when `limit` is omitted.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `params` | \{ `limit?`: `number`; `offset?`: `number`; `q?`: `string`; `status?`: `string`; \} | Optional search string, acquire-status filter, and pagination. |
| `params.limit?` | `number` | - |
| `params.offset?` | `number` | - |
| `params.q?` | `string` | - |
| `params.status?` | `string` | - |

#### Returns

`Promise`\<[`BooksResponse`](#booksresponse)\>

Paginated books plus total count for the pager.

***

### fetchClusterStatus()

```ts
function fetchClusterStatus(): Promise<ClusterStatusResponse>;
```

Cluster health every node administrator may see, with detail for the
caller's visible nodes.

#### Returns

`Promise`\<[`ClusterStatusResponse`](#clusterstatusresponse)\>

Cluster summary, visible nodes, and the caller's capabilities.

***

### fetchDiscoverFeed()

```ts
function fetchDiscoverFeed(limit?): Promise<DiscoverFeed>;
```

Loads Discover recommendation shelves.

#### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `limit?` | `number` | `36` | Max items per shelf (default 36). Seed URLs only; live prices load later. |

#### Returns

`Promise`\<[`DiscoverFeed`](#discoverfeed)\>

Discover feed payload.

***

### fetchEventDeliveries()

```ts
function fetchEventDeliveries(state?): Promise<EventDeliveryInfo[]>;
```

Lists recent event deliveries for the Library strip.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `state?` | `string` | Optional lifecycle filter (`dead_letter`, `running`, …). |

#### Returns

`Promise`\<[`EventDeliveryInfo`](#eventdeliveryinfo)[]\>

Delivery rows from `/api/events/deliveries`.

***

### fetchJobs()

```ts
function fetchJobs(): Promise<JobInfo[]>;
```

Lists recent background jobs for the jobs strip / status polling.

#### Returns

`Promise`\<[`JobInfo`](#jobinfo)[]\>

Job rows from `/api/jobs` (newest first as returned by the daemon).

***

### fetchMfaPolicy()

```ts
function fetchMfaPolicy(): Promise<{
  require_second_factor: boolean;
}>;
```

Host-wide password second-factor policy (`daemon.auth.require_second_factor`).

#### Returns

`Promise`\<\{
  `require_second_factor`: `boolean`;
\}\>

Whether password login requires a second factor.

***

### fetchOidcConfig()

```ts
function fetchOidcConfig(): Promise<OidcBrokerConfigView>;
```

Owner/operator identity-broker configuration (secrets redacted).

#### Returns

`Promise`\<[`OidcBrokerConfigView`](#oidcbrokerconfigview)\>

Enabled flag, callback URL, and provider rows.

***

### fetchPluginConsent()

```ts
function fetchPluginConsent(id, pluginKey?): Promise<PluginConsentResponse>;
```

Loads network/capability consent status for a plugin before enable.

Call from Settings when an operator reviews domains/bindings that still need
approval; `covered` is false until approvePluginConsent succeeds.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Display alias from Settings. |
| `pluginKey?` | `string` | Provenance-qualified PluginKey when the alias is ambiguous. |

#### Returns

`Promise`\<[`PluginConsentResponse`](#pluginconsentresponse)\>

Current consent request payload and whether it already covers the ask.

***

### fetchPluginInstanceConfig()

```ts
function fetchPluginInstanceConfig(instanceId): Promise<PluginInstanceConfig>;
```

Reads a plugin instance's stored document.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `instanceId` | `string` | Plugin instance id. |

#### Returns

`Promise`\<[`PluginInstanceConfig`](#plugininstanceconfig)\>

Stored settings, secret refs, and the document revision.

***

### fetchPreferences()

```ts
function fetchPreferences(): Promise<UserPreferences>;
```

Loads the signed-in user's preferences.

#### Returns

`Promise`\<[`UserPreferences`](#userpreferences)\>

Normalized [UserPreferences](#userpreferences).

***

### fetchPurchaseHints()

```ts
function fetchPurchaseHints(body): Promise<PurchaseHintsResponse>;
```

Resolves purchase hints, coalescing viewport-gated cards into a short batch window.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | [`PurchaseHintsQuery`](#purchasehintsquery) | Title identity and optional candidate edition. |

#### Returns

`Promise`\<[`PurchaseHintsResponse`](#purchasehintsresponse)\>

Hints list plus optional best pick (client-cached).

***

### fetchRequestQueue()

```ts
function fetchRequestQueue(): Promise<GlobalQueueEntry[]>;
```

Loads the global request queue (shared demand).

#### Returns

`Promise`\<[`GlobalQueueEntry`](#globalqueueentry)[]\>

Aggregated queue entries.

***

### fetchSettings()

```ts
function fetchSettings(): Promise<SettingsResponse>;
```

Loads operator settings groups and plugin option schemas for the Settings page.

#### Returns

`Promise`\<[`SettingsResponse`](#settingsresponse)\>

Core settings plus per-plugin option groups.

***

### fetchSigninMethods()

```ts
function fetchSigninMethods(): Promise<{
  integrations: object[];
  oidc: OidcProvider[];
  operator_token: boolean;
  require_second_factor: boolean;
}>;
```

Public sign-in picker (operator-token availability, SSO, integration logins).

#### Returns

`Promise`\<\{
  `integrations`: `object`[];
  `oidc`: [`OidcProvider`](#oidcprovider)[];
  `operator_token`: `boolean`;
  `require_second_factor`: `boolean`;
\}\>

Whether SPA operator-token login is offered, plus IdP and integration buttons.

***

### fetchStatus()

```ts
function fetchStatus(): Promise<StatusResponse>;
```

Loads aggregate library status counters.

#### Returns

`Promise`\<[`StatusResponse`](#statusresponse)\>

Status payload for the jobs strip / library header.

***

### fetchTitleMeta()

```ts
function fetchTitleMeta(body): Promise<TitleMeta | null>;
```

Fetches title metadata, coalescing viewport-gated cards into a short batch window.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | [`TitleMetaQuery`](#titlemetaquery) | Title identity fields. |

#### Returns

`Promise`\<[`TitleMeta`](#titlemeta) \| `null`\>

Metadata or `null` when none matched (client-cached).

***

### fetchTitleMetaBatch()

```ts
function fetchTitleMetaBatch(queries): Promise<(TitleMeta | null)[]>;
```

Batch title-meta fetch (order preserved); shares the client TTL cache with singles.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `queries` | [`TitleMetaQuery`](#titlemetaquery)[] | Parallel title-meta queries. |

#### Returns

`Promise`\<([`TitleMeta`](#titlemeta) \| `null`)[]\>

Metadata (or `null`) aligned with `queries`.

***

### fetchTitleReviews()

```ts
function fetchTitleReviews(body): Promise<TitleReviewsPage>;
```

Loads a page of Audible customer reviews for title detail infinite scroll.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | [`TitleReviewsQuery`](#titlereviewsquery) | ASIN, region, page, and sort. |

#### Returns

`Promise`\<[`TitleReviewsPage`](#titlereviewspage)\>

Reviews page payload.

***

### fetchTotpStatus()

```ts
function fetchTotpStatus(): Promise<{
  enabled: boolean;
}>;
```

Whether TOTP is confirmed for the current user.

#### Returns

`Promise`\<\{
  `enabled`: `boolean`;
\}\>

`{ enabled }` for the signed-in user.

***

### fetchWishlist()

```ts
function fetchWishlist(): Promise<TitleRequest[]>;
```

Loads the current user's wishlist items.

#### Returns

`Promise`\<[`TitleRequest`](#titlerequest)[]\>

Title-request rows.

***

### isApiError()

```ts
function isApiError(error): error is ApiError;
```

Narrows `unknown` to [ApiError](#apierror).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `error` | `unknown` | Caught value from a failed API call. |

#### Returns

`error is ApiError`

Whether `error` is an [ApiError](#apierror) instance.

***

### listClusterConsents()

```ts
function listClusterConsents(): Promise<ClusterConsentsResponse>;
```

Lists the owner clusters this host is enrolled in and what each asks to
run here. Operator only.

#### Returns

`Promise`\<[`ClusterConsentsResponse`](#clusterconsentsresponse)\>

Links and consent rows.

***

### listConfigNodes()

```ts
function listConfigNodes(): Promise<NodesResponse>;
```

Lists the nodes the caller may see: every node for an operator, the
owned ones for a node owner.

#### Returns

`Promise`\<[`NodesResponse`](#nodesresponse)\>

The serving node's id, the visible nodes, and what the caller may edit.

***

### listOidcClients()

```ts
function listOidcClients(): Promise<OidcAsClient[]>;
```

Lists Bookclerk-as-IdP clients (no plaintext secrets).

#### Returns

`Promise`\<[`OidcAsClient`](#oidcasclient)[]\>

Registered OAuth clients.

***

### listOidcIdentities()

```ts
function listOidcIdentities(): Promise<LinkedIdentity[]>;
```

Linked login identities for the current User.

#### Returns

`Promise`\<[`LinkedIdentity`](#linkedidentity)[]\>

***

### listOidcProviders()

```ts
function listOidcProviders(): Promise<{
  enabled: boolean;
  providers: OidcProvider[];
}>;
```

Public list of SSO providers for the login page.

#### Returns

`Promise`\<\{
  `enabled`: `boolean`;
  `providers`: [`OidcProvider`](#oidcprovider)[];
\}\>

***

### listPasskeys()

```ts
function listPasskeys(): Promise<ListedPasskey[]>;
```

Passkeys registered to the current User.

#### Returns

`Promise`\<[`ListedPasskey`](#listedpasskey)[]\>

***

### listSessions()

```ts
function listSessions(): Promise<ListedSession[]>;
```

Lists sessions for the current principal.

#### Returns

`Promise`\<[`ListedSession`](#listedsession)[]\>

Active session rows.

***

### listUsers()

```ts
function listUsers(): Promise<ListedUser[]>;
```

Lists users for administrator settings.

#### Returns

`Promise`\<[`ListedUser`](#listeduser)[]\>

User rows (empty array when none).

***

### login()

```ts
function login(token): Promise<AuthSession>;
```

Signs in with the operator token via `POST /api/auth/login`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `token` | `string` | Operator bearer token from `bookclerk daemon token`. |

#### Returns

`Promise`\<[`AuthSession`](#authsession)\>

Authenticated operator session (acquire enabled).

***

### logout()

```ts
function logout(): Promise<void>;
```

Clears the operator cookie session via `POST /api/auth/logout`.

#### Returns

`Promise`\<`void`\>

Resolves when the logout request finishes.

***

### mintUserClaimTicket()

```ts
function mintUserClaimTicket(id): Promise<{
  claim_ticket: string;
  invite_url?: string | null;
  ok: boolean;
}>;
```

Mints a fresh one-time claim ticket for an existing active user.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | User id. |

#### Returns

`Promise`\<\{
  `claim_ticket`: `string`;
  `invite_url?`: `string` \| `null`;
  `ok`: `boolean`;
\}\>

One-time claim ticket.

***

### passkeyElevateBegin()

```ts
function passkeyElevateBegin(): Promise<{
  challenge_id: string;
  publicKey: Record<string, unknown>;
}>;
```

Begin passkey Owner elevation.

#### Returns

`Promise`\<\{
  `challenge_id`: `string`;
  `publicKey`: `Record`\<`string`, `unknown`\>;
\}\>

***

### passkeyElevateFinish()

```ts
function passkeyElevateFinish(body): Promise<void>;
```

Finish passkey Owner elevation.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `challenge_id`: `string`; `credential`: `Record`\<`string`, `unknown`\>; \} | - |
| `body.challenge_id` | `string` | - |
| `body.credential` | `Record`\<`string`, `unknown`\> | - |

#### Returns

`Promise`\<`void`\>

***

### passkeyLoginBegin()

```ts
function passkeyLoginBegin(login): Promise<{
  challenge_id: string;
  publicKey: Record<string, unknown>;
}>;
```

Begin identifier-first passkey login.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `login` | `string` | - |

#### Returns

`Promise`\<\{
  `challenge_id`: `string`;
  `publicKey`: `Record`\<`string`, `unknown`\>;
\}\>

***

### passkeyLoginFinish()

```ts
function passkeyLoginFinish(body): Promise<void>;
```

Finish passkey login (sets the portal session cookie).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `challenge_id`: `string`; `credential`: `Record`\<`string`, `unknown`\>; \} | - |
| `body.challenge_id` | `string` | - |
| `body.credential` | `Record`\<`string`, `unknown`\> | - |

#### Returns

`Promise`\<`void`\>

***

### passkeyRegisterBegin()

```ts
function passkeyRegisterBegin(currentPassword?): Promise<{
  challenge_id: string;
  publicKey: Record<string, unknown>;
}>;
```

Begin WebAuthn registration (authenticated).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `currentPassword?` | `string` | - |

#### Returns

`Promise`\<\{
  `challenge_id`: `string`;
  `publicKey`: `Record`\<`string`, `unknown`\>;
\}\>

***

### passkeyRegisterFinish()

```ts
function passkeyRegisterFinish(body): Promise<void>;
```

Finish WebAuthn registration.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `challenge_id`: `string`; `credential`: `Record`\<`string`, `unknown`\>; `name?`: `string`; \} | - |
| `body.challenge_id` | `string` | - |
| `body.credential` | `Record`\<`string`, `unknown`\> | - |
| `body.name?` | `string` | - |

#### Returns

`Promise`\<`void`\>

***

### passwordLogin()

```ts
function passwordLogin(login, password): Promise<{
  mfa?: {
     challenge_id: string;
     method: string;
  };
  ok: boolean;
}>;
```

Signs in with username/password via `POST /api/auth/password`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `login` | `string` | Username or email. |
| `password` | `string` | Account password. |

#### Returns

`Promise`\<\{
  `mfa?`: \{
     `challenge_id`: `string`;
     `method`: `string`;
  \};
  `ok`: `boolean`;
\}\>

Session cookie on success, or an MFA challenge when TOTP is enabled.

***

### patchPreferences()

```ts
function patchPreferences(body): Promise<UserPreferences>;
```

Partially updates the signed-in user's preferences.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `default_view?`: [`AppView`](#appview); `disabled_shelves?`: `string`[]; `discover_excluded_sources?`: `string`[]; `discover_language?`: `string` \| `null`; `discover_sort?`: [`CatalogSearchSort`](#catalogsearchsort); `discover_sort_dir?`: `"desc"` \| `"asc"`; `theme?`: `ThemePreference`; \} | Fields to patch (`discover_language: null` clears to browser default). |
| `body.default_view?` | [`AppView`](#appview) | - |
| `body.disabled_shelves?` | `string`[] | - |
| `body.discover_excluded_sources?` | `string`[] | - |
| `body.discover_language?` | `string` \| `null` | Pass `null` to clear to browser default. |
| `body.discover_sort?` | [`CatalogSearchSort`](#catalogsearchsort) | - |
| `body.discover_sort_dir?` | `"desc"` \| `"asc"` | - |
| `body.theme?` | `ThemePreference` | - |

#### Returns

`Promise`\<[`UserPreferences`](#userpreferences)\>

Updated preferences after save.

***

### patchProfile()

```ts
function patchProfile(body): Promise<{
  ok: boolean;
  user: AuthMeUser;
}>;
```

Updates the signed-in user's display name and/or email.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `avatar_source?`: `string`; `display_name?`: `string`; `email?`: `string`; \} | Fields to change; omitted keys are left unchanged. |
| `body.avatar_source?` | `string` | - |
| `body.display_name?` | `string` | - |
| `body.email?` | `string` | - |

#### Returns

`Promise`\<\{
  `ok`: `boolean`;
  `user`: [`AuthMeUser`](#authmeuser);
\}\>

Updated profile snapshot from the daemon.

***

### patchSettings()

```ts
function patchSettings(body): Promise<SettingsResponse>;
```

Writes bootstrap settings (`daemon`, `jail`, `[database]` keys) through
`PATCH /api/settings`. Plugin settings are not accepted here: they are
instance documents, written with [putPluginInstanceConfig](#putplugininstanceconfig).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `expected_output_revision?`: `number`; `settings`: [`SettingsUpdate`](#settingsupdate)[]; \} | Settings updates to apply. |
| `body.expected_output_revision?` | `number` | `core.output` revision the caller read; a stale value is rejected with 409. |
| `body.settings` | [`SettingsUpdate`](#settingsupdate)[] | - |

#### Returns

`Promise`\<[`SettingsResponse`](#settingsresponse)\>

The settings view after the write.

***

### patchUser()

```ts
function patchUser(id, body): Promise<UserMutationResponse>;
```

Updates a first-party user role, status, or profile fields.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | User id to patch. |
| `body` | \{ `display_name?`: `string`; `email?`: `string`; `login_name?`: `string`; `role?`: `string`; `status?`: `string`; \} | Partial user fields accepted by the daemon. |
| `body.display_name?` | `string` | - |
| `body.email?` | `string` | - |
| `body.login_name?` | `string` | - |
| `body.role?` | `string` | - |
| `body.status?` | `string` | - |

#### Returns

`Promise`\<[`UserMutationResponse`](#usermutationresponse)\>

Patched user row.

***

### portalConnections()

```ts
function portalConnections(): Promise<PortalConnection[]>;
```

Lists linked store accounts for the current portal user.

#### Returns

`Promise`\<[`PortalConnection`](#portalconnection)[]\>

Connection rows.

***

### portalLoginIntegration()

```ts
function portalLoginIntegration(body): Promise<void>;
```

Signs in via an integration provider username/password.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `password`: `string`; `provider`: `string`; `username`: `string`; \} | Provider id plus credentials. |
| `body.password` | `string` | - |
| `body.provider` | `string` | - |
| `body.username` | `string` | - |

#### Returns

`Promise`\<`void`\>

Resolves when the portal session is established.

***

### portalLogout()

```ts
function portalLogout(): Promise<void>;
```

Clears the portal / user session cookie.

#### Returns

`Promise`\<`void`\>

Resolves when logout finishes (best-effort).

***

### portalMe()

```ts
function portalMe(): Promise<PortalMe>;
```

Loads the linked portal identity for the signed-in SPA session.

#### Returns

`Promise`\<[`PortalMe`](#portalme)\>

Portal identity metadata used by Accounts / Settings headers.

***

### portalRedeem()

```ts
function portalRedeem(ticket, password?): Promise<void>;
```

Redeems a claim ticket into a portal user session.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `ticket` | `string` | One-time claim ticket string. |
| `password?` | `string` | Optional password when claiming invite/reset tickets. |

#### Returns

`Promise`\<`void`\>

Resolves when the session cookie is set.

***

### portalRevokeConnection()

```ts
function portalRevokeConnection(source, accountId): Promise<void>;
```

Disconnects a storefront account from the current portal user.

Does not delete library rows; only revokes the sealed credentials / link.
Accounts belong to a plugin, so the source (alias or PluginKey) is part of
the address: the same account id may exist on two stores.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `source` | `string` | Source id or PluginKey from [portalConnections](#portalconnections). |
| `accountId` | `string` | Portal connection account id from [portalConnections](#portalconnections). |

#### Returns

`Promise`\<`void`\>

Resolves when the revoke POST succeeds.

***

### portalSourceLogin()

```ts
function portalSourceLogin(id, body): Promise<void>;
```

Password-connects a storefront account.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Source plugin id. |
| `body` | \{ `email`: `string`; `password`: `string`; \} | Store email and password. |
| `body.email` | `string` | - |
| `body.password` | `string` | - |

#### Returns

`Promise`\<`void`\>

Resolves when the connection is stored.

***

### portalSourceOauthStart()

```ts
function portalSourceOauthStart(id): Promise<{
  url: string;
}>;
```

Starts OAuth for a storefront and returns the authorize URL.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Source plugin id. |

#### Returns

`Promise`\<\{
  `url`: `string`;
\}\>

Object with browser `url` to open.

***

### portalSources()

```ts
function portalSources(): Promise<PortalSource[]>;
```

Lists connectable storefronts for Accounts.

#### Returns

`Promise`\<[`PortalSource`](#portalsource)[]\>

Source descriptors including brand metadata.

***

### putMfaPolicy()

```ts
function putMfaPolicy(requireSecondFactor, currentPassword?): Promise<{
  require_second_factor: boolean;
}>;
```

Persists whether password login requires TOTP or a passkey.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `requireSecondFactor` | `boolean` | When true, password-only sign-in is not enough once enrolled. |
| `currentPassword?` | `string` | Owner step-up; operators omit this. |

#### Returns

`Promise`\<\{
  `require_second_factor`: `boolean`;
\}\>

Saved `{ require_second_factor }` flag.

***

### putNodeLabels()

```ts
function putNodeLabels(nodeId, labels): Promise<unknown>;
```

Replaces a node's operator labels.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `nodeId` | `string` | Node to edit. |
| `labels` | `string`[] | The complete label set (`nas`, `zone=eu`). |

#### Returns

`Promise`\<`unknown`\>

The updated node.

***

### putNodeOwner()

```ts
function putNodeOwner(nodeId, userId): Promise<{
  node_id: string;
  owner_user_id: number | null;
}>;
```

Assigns a node to the user who administers it, or clears its owner.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `nodeId` | `string` | Node to assign. |
| `userId` | `number` \| `null` | Owning user, or `null` to clear. |

#### Returns

`Promise`\<\{
  `node_id`: `string`;
  `owner_user_id`: `number` \| `null`;
\}\>

The node id and its owner.

***

### putNodeRuntime()

```ts
function putNodeRuntime(nodeId, body): Promise<{
  node_id: string;
  revision: number;
  runtime: NodeRuntimeView;
}>;
```

Edits a node's label, budgets, or drain switch. The node applies it at
its next heartbeat.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `nodeId` | `string` | Node to edit. |
| `body` | `Partial`\<[`NodeRuntimeView`](#noderuntimeview)\> | Fields to change; omitted fields keep their value. |

#### Returns

`Promise`\<\{
  `node_id`: `string`;
  `revision`: `number`;
  `runtime`: [`NodeRuntimeView`](#noderuntimeview);
\}\>

The stored runtime document.

***

### putOidcConfig()

```ts
function putOidcConfig(body): Promise<OidcBrokerConfigView>;
```

Replaces `[auth.oidc]` providers. Omit `client_secret` to keep the stored value.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | [`OidcBrokerConfigUpdate`](#oidcbrokerconfigupdate) | Broker enablement and provider list. |

#### Returns

`Promise`\<[`OidcBrokerConfigView`](#oidcbrokerconfigview)\>

Saved configuration with redacted secrets.

***

### putPluginDeployment()

```ts
function putPluginDeployment(
   instanceId, 
   nodeId, 
   desired, 
   expectedRevision?
): Promise<PluginDeploymentView>;
```

Sets whether a plugin instance should run on a host (the enable switch).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `instanceId` | `string` | Plugin instance id. |
| `nodeId` | `string` | Target node id. |
| `desired` | `"present"` \| `"absent"` | `present` to run it there, `absent` to retire it. |
| `expectedRevision?` | `number` | Deployment revision last observed, for compare-and-swap. |

#### Returns

`Promise`\<[`PluginDeploymentView`](#plugindeploymentview)\>

The deployment row after the write.

***

### putPluginInstanceConfig()

```ts
function putPluginInstanceConfig(instanceId, body): Promise<{
  plugin_instance_id: string;
  revision: number;
}>;
```

Replaces a plugin instance's document (compare-and-swap on `expected_revision`).

Secrets in `body.secrets` are sealed server-side and referenced by name;
the response never carries them.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `instanceId` | `string` | Plugin instance id. |
| `body` | [`PluginInstanceConfigWrite`](#plugininstanceconfigwrite) | New document plus secrets to seal. |

#### Returns

`Promise`\<\{
  `plugin_instance_id`: `string`;
  `revision`: `number`;
\}\>

The committed revision.

***

### putPluginPlacement()

```ts
function putPluginPlacement(instanceId, body): Promise<{
  placement: PluginPlacementView;
  plugin_instance_id: string;
}>;
```

Hands an instance's deployments to the cluster scheduler, or changes its
placement (compare-and-swap on `expected_revision`; 0 creates).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `instanceId` | `string` | Plugin instance id. |
| `body` | \{ `constraints`: `string`[]; `expected_revision`: `number`; `replicas`: `number` \| `"all"`; \} | Replica count, label constraints, and the revision last seen. |
| `body.constraints` | `string`[] | - |
| `body.expected_revision` | `number` | - |
| `body.replicas` | `number` \| `"all"` | - |

#### Returns

`Promise`\<\{
  `placement`: [`PluginPlacementView`](#pluginplacementview);
  `plugin_instance_id`: `string`;
\}\>

The stored placement.

***

### removeWishlistItem()

```ts
function removeWishlistItem(uuid): Promise<TitleRequest>;
```

Deletes a wishlist title-request owned by the current user.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `uuid` | `string` | Wishlist title-request uuid from [fetchWishlist](#fetchwishlist). |

#### Returns

`Promise`\<[`TitleRequest`](#titlerequest)\>

The removed title-request row as echoed by the API.

***

### resetUserPassword()

```ts
function resetUserPassword(id): Promise<{
  claim_ticket: string;
  invite_url?: string | null;
  ok: boolean;
  revoked_sessions: number;
}>;
```

Mints a password-reset claim ticket and revokes the user's portal sessions.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | User id. |

#### Returns

`Promise`\<\{
  `claim_ticket`: `string`;
  `invite_url?`: `string` \| `null`;
  `ok`: `boolean`;
  `revoked_sessions`: `number`;
\}\>

Reset ticket plus count of revoked sessions.

***

### resumeEventDelivery()

```ts
function resumeEventDelivery(id): Promise<ActionResponse>;
```

Wakes a suspended delivery for the next worker claim.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Delivery id (`{eventId}:{pluginId}`). |

#### Returns

`Promise`\<[`ActionResponse`](#actionresponse)\>

Action ack from the daemon.

***

### retryEventDelivery()

```ts
function retryEventDelivery(id): Promise<ActionResponse>;
```

Re-queues a dead-lettered delivery.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Delivery id (`{eventId}:{pluginId}`). |

#### Returns

`Promise`\<[`ActionResponse`](#actionresponse)\>

Action ack from the daemon.

***

### revokeClusterConsent()

```ts
function revokeClusterConsent(clusterId, pluginKey): Promise<{
  consent: RemoteConsentView;
}>;
```

Withdraws an approval: held jobs of the plugin are cancelled and given
back, and nothing of it is claimed until approved again.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `clusterId` | `string` | Owner cluster. |
| `pluginKey` | `string` | Canonical plugin key. |

#### Returns

`Promise`\<\{
  `consent`: [`RemoteConsentView`](#remoteconsentview);
\}\>

The consent row after revoking.

***

### revokeSession()

```ts
function revokeSession(id): Promise<void>;
```

Revokes one session visible to the current principal.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | Session id. |

#### Returns

`Promise`\<`void`\>

Resolves when the revoke succeeds.

***

### rotateOidcClientSecret()

```ts
function rotateOidcClientSecret(clientId, currentPassword?): Promise<OidcAsClient>;
```

Rotates a confidential client's secret; plaintext is returned once.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `clientId` | `string` | Confidential client to rotate. |
| `currentPassword?` | `string` | Owner re-auth when required. |

#### Returns

`Promise`\<[`OidcAsClient`](#oidcasclient)\>

Client row including the new plaintext secret once.

***

### searchCatalog()

```ts
function searchCatalog(q, opts?): Promise<CatalogSearchPage>;
```

Searches the Discover catalog across enabled storefronts.

Returns an empty page (without calling the network) when `q` trims to fewer
than 2 characters. Language defaults to the browser locale unless
`allLanguages` is set.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `q` | `string` | Free-text query (minimum 2 characters after trim). |
| `opts?` | \{ `allLanguages?`: `boolean`; `cursor?`: `string` \| `null`; `field?`: `"author"` \| `"narrator"` \| `"series"` \| `"genre"`; `filters?`: [`CatalogSearchFilters`](#catalogsearchfilters); `lang?`: `string`; `limit?`: `number`; `page_size?`: `number`; `sort?`: [`CatalogSearchSort`](#catalogsearchsort); `sort_dir?`: [`CatalogSortDir`](#catalogsortdir); \} | Pagination cursor, sort, language, and server-side facet filters. |
| `opts.allLanguages?` | `boolean` | When true, do not hard-filter by language. |
| `opts.cursor?` | `string` \| `null` | - |
| `opts.field?` | `"author"` \| `"narrator"` \| `"series"` \| `"genre"` | - |
| `opts.filters?` | [`CatalogSearchFilters`](#catalogsearchfilters) | - |
| `opts.lang?` | `string` | - |
| `opts.limit?` | `number` | **Deprecated** use page_size |
| `opts.page_size?` | `number` | - |
| `opts.sort?` | [`CatalogSearchSort`](#catalogsearchsort) | - |
| `opts.sort_dir?` | [`CatalogSortDir`](#catalogsortdir) | - |

#### Returns

`Promise`\<[`CatalogSearchPage`](#catalogsearchpage)\>

Cursor-paginated search page (or an empty stub when `q` is too short).

***

### setPassword()

```ts
function setPassword(body): Promise<{
  ok: boolean;
  revoked_sessions: number;
}>;
```

Sets a password for self or for a managed user.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `current_password?`: `string`; `password`: `string`; `user_id?`: `number`; \} | New password and optional target user id. |
| `body.current_password?` | `string` | - |
| `body.password` | `string` | - |
| `body.user_id?` | `number` | - |

#### Returns

`Promise`\<\{
  `ok`: `boolean`;
  `revoked_sessions`: `number`;
\}\>

Count of revoked sessions.

***

### signOut()

```ts
function signOut(role?): Promise<void>;
```

Signs out regardless of operator vs user/portal session.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `role?` | [`AuthRole`](#authrole) | Current session role; portal roles use portal logout. |

#### Returns

`Promise`\<`void`\>

Resolves when logout completes.

***

### startImpersonate()

```ts
function startImpersonate(userId): Promise<void>;
```

Starts operator (or elevated Owner) impersonation of another user.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `userId` | `number` | Target user id. |

#### Returns

`Promise`\<`void`\>

Resolves when impersonation is active.

***

### stopImpersonate()

```ts
function stopImpersonate(): Promise<void>;
```

Stops the current operator impersonation session.

#### Returns

`Promise`\<`void`\>

Resolves when impersonation ends.

***

### totpEnrollBegin()

```ts
function totpEnrollBegin(currentPassword?): Promise<{
  otpauth_url: string;
  qr_svg: string;
  secret: string;
}>;
```

Starts TOTP enrollment (pending secret, otpauth URL, QR SVG).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `currentPassword?` | `string` | Required when the portal session is older than the reauth window. |

#### Returns

`Promise`\<\{
  `otpauth_url`: `string`;
  `qr_svg`: `string`;
  `secret`: `string`;
\}\>

Pending secret, otpauth URL, and QR SVG.

***

### totpEnrollFinish()

```ts
function totpEnrollFinish(code): Promise<void>;
```

Confirms the pending TOTP secret with an authenticator code.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `code` | `string` | Six-digit code from the authenticator app. |

#### Returns

`Promise`\<`void`\>

Resolves when enrollment is confirmed.

***

### totpLogin()

```ts
function totpLogin(challengeId, code): Promise<void>;
```

Completes password+TOTP login via `POST /api/auth/totp/login`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `challengeId` | `string` | Challenge from the password-login MFA payload. |
| `code` | `string` | Six-digit authenticator code. |

#### Returns

`Promise`\<`void`\>

Resolves when the session cookie is set.

***

### triggerAcquire()

```ts
function triggerAcquire(body): Promise<ActionResponse>;
```

Queues acquire for a library book by uuid and/or ASIN.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `asin?`: `string`; `uuid?`: `string`; \} | Target identifiers (`uuid` and/or `asin`). |
| `body.asin?` | `string` | - |
| `body.uuid?` | `string` | - |

#### Returns

`Promise`\<[`ActionResponse`](#actionresponse)\>

Action ack including `job_id`.

***

### triggerScan()

```ts
function triggerScan(): Promise<ActionResponse>;
```

Queues a full-library scan job for accounts with scan enabled.

Prefer CLI one-shot scans with an explicit `--account` in cloud testing; this
SPA helper hits the authenticated daemon endpoint used by the Library UI.

#### Returns

`Promise`\<[`ActionResponse`](#actionresponse)\>

Action ack including `job_id` when accepted.

***

### updateOidcClient()

```ts
function updateOidcClient(clientId, body): Promise<OidcAsClient>;
```

Updates an OIDC client without echoing a secret unless flipping to confidential.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `clientId` | `string` | Existing client_id. |
| `body` | [`OidcAsClientWrite`](#oidcasclientwrite) | Redirects and token policy. |

#### Returns

`Promise`\<[`OidcAsClient`](#oidcasclient)\>

Updated client row (no plaintext secret unless flipping to confidential).

***

### uploadProfileAvatar()

```ts
function uploadProfileAvatar(file): Promise<void>;
```

Uploads a JPEG/PNG/WebP profile picture for the signed-in user.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `file` | `File` | Image selected from the file picker. |

#### Returns

`Promise`\<`void`\>

***

### userAvatarUrl()

```ts
function userAvatarUrl(userId, cacheKey?): string;
```

Authenticated URL for a user's stored profile picture.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `userId` | `number` | First-party `users.id`. |
| `cacheKey?` | `number` | Optional cache-buster after upload. |

#### Returns

`string`

Path used by `<img src>`.

***

### userFacingApiError()

```ts
function userFacingApiError(error, fallback): string;
```

Maps API / network failures to short copy for non-technical users.

Keeps branded daemon messages when they already read as plain language.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `error` | `unknown` | Caught failure from fetch / parse helpers. |
| `fallback` | `string` | Copy used when the error has no safe user-facing text. |

#### Returns

`string`

Message suitable for inline SPA error banners.
