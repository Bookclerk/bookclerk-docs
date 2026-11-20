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

`Error.constructor`

#### Properties

##### code?

```ts
code?: string;
```

Machine slug from JSON error bodies when present.

##### pluginId?

```ts
pluginId?: string;
```

##### status

```ts
status: number;
```

##### summary?

```ts
summary?: string[];
```

## Interfaces

### AccountHolder

A listing held by a store account.

#### Properties

##### account\_id

```ts
account_id: string;
```

The account's id on that storefront.

##### kind

```ts
kind: "account";
```

Discriminant.

##### plugin\_key

```ts
plugin_key: string;
```

The account's storefront plugin (`audible`, `libro`, ...).

***

### ActionResponse

Ack payload for scan / acquire job triggers.

#### Properties

##### error?

```ts
error?: string | null;
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
avatar_source?: string | null;
```

`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`.

##### display\_name

```ts
display_name: string | null;
```

##### email?

```ts
email?: string | null;
```

##### gravatar\_hash?

```ts
gravatar_hash?: string | null;
```

SHA-256 hex of the contact email for Gravatar.

##### has\_avatar?

```ts
has_avatar?: boolean;
```

True when a profile picture file is stored for this user.

##### has\_password?

```ts
has_password?: boolean;
```

##### id

```ts
id: number;
```

##### login\_name?

```ts
login_name?: string | null;
```

##### role

```ts
role: string;
```

##### sso\_pictures?

```ts
sso_pictures?: object[];
```

IdP-supplied pictures the user can pick.

###### identity\_id

```ts
identity_id: number;
```

###### last\_used\_at?

```ts
last_used_at?: string | null;
```

###### picture\_path?

```ts
picture_path?: string | null;
```

Same-origin 128 px copy of `picture_url`, when the server signed one.

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
status?: string;
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

##### administers\_nodes?

```ts
administers_nodes?: boolean;
```

Whether the session reaches node administration and the Status page:
the operator, an Owner or Administrator, or a Member who owns a node.

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
elevated?: boolean;
```

##### impersonating?

```ts
impersonating?: AuthMeImpersonating;
```

##### portal?

```ts
portal?: PortalInfo;
```

##### role?

```ts
role?: AuthRole;
```

##### second\_factor?

```ts
second_factor?: AuthSecondFactor;
```

##### see\_prices

```ts
see_prices: boolean;
```

Whether the caller may see prices and store links. When false the
library already drops them from its answers, and the app shows no
store links of its own either.

##### user?

```ts
user?: AuthMeUser;
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
invite_url?: string | null;
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

### BranchAvailability

One branch of this library that can supply a title (it answered a
catalog lookup for the title's work).

#### Properties

##### node\_id

```ts
node_id: string;
```

The branch node id.

##### owner\_name?

```ts
owner_name?: string | null;
```

The branch owner's display name, when the owner has one.

##### product\_id

```ts
product_id: string;
```

The store's product id of the branch's copy.

##### store

```ts
store: string;
```

The store the branch has the book from (`audible`).

##### work\_id

```ts
work_id: string;
```

This library's work the branch supplies; what [requestFromBranch](#requestfrombranch) asks for.

***

### BranchRequestQueued

`POST /api/branch-requests` answer.

#### Properties

##### already\_queued

```ts
already_queued: boolean;
```

`true` when a request for this work to this branch was already waiting.

##### job\_id

```ts
job_id: string;
```

The request's job, as `/api/jobs` lists it.

##### node\_id

```ts
node_id: string;
```

The branch the request went to.

***

### CatalogListingView

The signed-in user's catalog listing choice.

#### Properties

##### always\_listed

```ts
always_listed: boolean;
```

`true` for an Owner, whose books are always listed; `listed` then has no effect.

##### libraries

```ts
libraries: number;
```

Libraries this server is currently a branch of; the choice matters only above zero.

##### listed

```ts
listed: boolean;
```

`true` (the default) unless the user keeps their books out of catalog answers.

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

##### available\_through?

```ts
available_through?: BranchAvailability[];
```

Branches of this library that currently list this title.

##### cover\_path?

```ts
cover_path?: string | null;
```

Same-origin path of this cover's processed images; see [coverSrc](#coversrc).

##### cover\_url?

```ts
cover_url?: string | null;
```

##### description?

```ts
description?: string | null;
```

##### genres?

```ts
genres?: string | null;
```

##### is\_abridged?

```ts
is_abridged?: boolean | null;
```

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
language?: string | null;
```

##### length\_minutes?

```ts
length_minutes?: number | null;
```

##### narrators

```ts
narrators: string | null;
```

##### price\_cents?

```ts
price_cents?: number | null;
```

##### published\_at?

```ts
published_at?: string | null;
```

##### publisher?

```ts
publisher?: string | null;
```

##### purchase\_hints?

```ts
purchase_hints?: PurchaseHint[];
```

##### rating\_count?

```ts
rating_count?: number | null;
```

##### rating\_overall?

```ts
rating_overall?: number | null;
```

##### series

```ts
series: string | null;
```

##### series\_index?

```ts
series_index?: string | null;
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
subtitle?: string | null;
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
next_cursor?: string | null;
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
sort_dir?: string;
```

***

### ConfigNodeView

One library node as `GET /api/config/nodes` returns it.

#### Properties

##### implicit\_labels

```ts
implicit_labels: string[];
```

Labels every node carries (`node=<id>`, `target=<target>`).

##### kind

```ts
kind: "shared" | "branch";
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
software_version?: string;
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

### DiagnosticsReport

One diagnostics report this install sent (`GET /api/diagnostics/reports`).

#### Properties

##### report\_id

```ts
report_id: string;
```

The collector's ID for the report; the GitHub issue filed for it mentions this ID.

##### sent\_at\_unix\_ms

```ts
sent_at_unix_ms: number;
```

When the collector accepted it, Unix milliseconds.

##### trigger

```ts
trigger: string;
```

Why it was sent: `crash`, `error_burst`, `slow_burst`, `job_failed`, `cli_error`, ….

##### version

```ts
version: string;
```

The Bookclerk version that sent it.

***

### DiscoverFeed

Discover recommendations feed payload.

#### Properties

##### computed\_at?

```ts
computed_at?: string;
```

RFC 3339 time the stored shelves were ranked.

##### shelf\_kinds?

```ts
shelf_kinds?: ShelfKindInfo[];
```

##### shelves

```ts
shelves: DiscoverShelf[];
```

##### state?

```ts
state?: DiscoverFeedState;
```

How current the stored shelves are: `preparing` (none stored yet),
`refreshing` (newer ones queued) or `ready`.

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

### DiscoverStatus

Stored state of the caller's Discover shelves (`GET /api/discover/status`).

#### Properties

##### candidates\_fetched\_at?

```ts
candidates_fetched_at?: string | null;
```

##### feed\_computed\_at?

```ts
feed_computed_at?: string | null;
```

##### ready

```ts
ready: boolean;
```

##### scope?

```ts
scope?: string | null;
```

##### subject

```ts
subject: string;
```

##### updating

```ts
updating: boolean;
```

True while a fetch or ranking for the caller is waiting or running.

***

### Edition

One edition of a work, with its listings.

#### Properties

##### abridgement

```ts
abridgement: Abridgement;
```

Whether it is abridged.

##### acquire\_status

```ts
acquire_status: AcquireStatus;
```

Its acquire state.

##### cover\_path

```ts
cover_path: string | null;
```

Same-origin path of its processed cover; see [coverSrc](#coversrc).

##### duration\_minutes

```ts
duration_minutes: number | null;
```

Runtime in minutes, when known.

##### expression\_id

```ts
expression_id: string;
```

Its expression.

##### expression\_kind

```ts
expression_kind: ExpressionKind;
```

How the expression renders the work.

##### id

```ts
id: string;
```

Edition id; acquire takes it in `editions`.

##### language

```ts
language: string | null;
```

Language code, when known.

##### listings

```ts
listings: Listing[];
```

Store listings that sell it.

##### media\_kind

```ts
media_kind: MediaKind;
```

How it is consumed.

##### published\_at

```ts
published_at: string | null;
```

Publication date (ISO-8601), when known.

##### subtitle

```ts
subtitle: string | null;
```

Subtitle, when known.

##### title

```ts
title: string;
```

Edition title.

##### work\_id

```ts
work_id: string;
```

The work it belongs to (an episode for a podcast's editions).

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
dead_letters_total?: number;
```

##### dispatch\_latency\_ms\_avg?

```ts
dispatch_latency_ms_avg?: number | null;
```

##### handler\_latency\_ms\_avg?

```ts
handler_latency_ms_avg?: number | null;
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
retries_total?: number;
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
suspensions_total?: number;
```

***

### FacetValue

One value of a facet, from `GET /api/library/works/facets/{facet}`.

#### Properties

##### value

```ts
value: string;
```

The value, as [WorksQuery.value](#value-5) takes it.

##### works

```ts
works: number;
```

How many held works have it.

***

### FacetValuesPage

A page of a facet's values and the cursor of the next one.

#### Properties

##### next\_cursor?

```ts
next_cursor?: string | null;
```

Pass as `cursor` for the next page; absent on the last.

##### values

```ts
values: FacetValue[];
```

The page's values, by their lower-cased spelling.

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

##### cover\_path?

```ts
cover_path?: string | null;
```

Same-origin path of this cover's processed images; see [coverSrc](#coversrc).

##### cover\_url?

```ts
cover_url?: string | null;
```

##### description?

```ts
description?: string | null;
```

##### first\_requested\_at

```ts
first_requested_at: string;
```

##### genres?

```ts
genres?: string | null;
```

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
language?: string | null;
```

##### last\_requested\_at

```ts
last_requested_at: string;
```

##### length\_minutes?

```ts
length_minutes?: number | null;
```

##### narrators?

```ts
narrators?: string | null;
```

##### published\_at?

```ts
published_at?: string | null;
```

##### publisher?

```ts
publisher?: string | null;
```

##### purchase\_hints?

```ts
purchase_hints?: PurchaseHint[];
```

##### reasons?

```ts
reasons?: string[];
```

##### sample\_uuids

```ts
sample_uuids: string[];
```

##### score?

```ts
score?: number;
```

Final rank (`taste_score + wish_count` boost).

##### series?

```ts
series?: string | null;
```

##### series\_index?

```ts
series_index?: string | null;
```

##### store\_editions?

```ts
store_editions?: StoreEdition[];
```

##### subtitle?

```ts
subtitle?: string | null;
```

##### taste\_score?

```ts
taste_score?: number;
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
wishers?: QueueWisher[];
```

Distinct people who have this title on an open wishlist.

##### work\_id

```ts
work_id: string;
```

Library work the entry's wishes are for. Wishes for one work share an
entry even when their `work_key`s differ, so match on this first.

##### work\_key

```ts
work_key: string;
```

***

### Holding

One holding of a store listing.

#### Properties

##### added\_at

```ts
added_at: string;
```

When the library first saw it (ISO-8601).

##### entitlement

```ts
entitlement: Entitlement;
```

How it is held.

##### holder

```ts
holder: Holder;
```

Who holds it.

##### id

```ts
id: string;
```

Holding id.

##### purchased\_at

```ts
purchased_at: string | null;
```

When the store says it was bought (ISO-8601), when known.

***

### JobInfo

Background job row from `/api/jobs`.

#### Properties

##### attempt\_count?

```ts
attempt_count?: number;
```

##### created\_at?

```ts
created_at?: string | null;
```

##### detail

```ts
detail: string | null;
```

##### error\_kind?

```ts
error_kind?: string | null;
```

##### finished\_at?

```ts
finished_at?: string | null;
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
max_attempts?: number;
```

##### progress?

```ts
progress?: string | null;
```

##### status

```ts
status: string;
```

***

### LibraryConsentsResponse

`GET /api/config/library-consents`.

#### Properties

##### consents

```ts
consents: RemoteConsentView[];
```

Every recorded owner plugin request, approved or not.

##### libraries

```ts
libraries: LibraryLinkView[];
```

Every library link this install holds, in any state.

***

### LibraryLinkView

A library this install is enrolled in as a host (no key material).

#### Properties

##### enrolled\_at

```ts
enrolled_at: string;
```

RFC 3339 time of the latest enrollment (a re-enrollment resets it).

##### include\_branches?

```ts
include_branches?: boolean;
```

"Include books from my own branches" (on by default).

##### job\_share?

```ts
job_share?: number;
```

Jobs the library may have running here now; absent while its link is not running.

##### keep\_copies?

```ts
keep_copies?: boolean;
```

"Keep copies on this computer" (on by default): take the library's copies of this server's person's books.

##### keep\_member\_copies?

```ts
keep_member_copies?: boolean;
```

"Keep copies of other members' books" (off by default).

##### library\_id

```ts
library_id: string;
```

Id of the library; one link per library.

##### list\_members\_books?

```ts
list_members_books?: boolean;
```

"List my members' books" when the library asks (on by default).

##### max\_jobs?

```ts
max_jobs?: number | null;
```

The most jobs this host runs for the library at once; `null` is an equal share of the machine.

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
share_books?: boolean;
```

"Share this branch's books with <library>" (on by default).

##### state

```ts
state: string;
```

`active`, `revoked` (by the owner), or `left` (by this host).

##### work\_for\_members?

```ts
work_for_members?: boolean;
```

"Download for other members of <library>" (off by default).

***

### LibraryStatusResponse

`GET /api/library-status`: library health plus the nodes the caller may see.

#### Extends

- [`NodesResponse`](#nodesresponse)

#### Properties

##### capabilities

```ts
capabilities: NodeCapabilities;
```

###### Inherited from

[`NodesResponse`](#nodesresponse).[`capabilities`](#capabilities-1)

##### library

```ts
library: LibrarySummary;
```

##### node\_id

```ts
node_id: string;
```

###### Inherited from

[`NodesResponse`](#nodesresponse).[`node_id`](#node_id-6)

##### nodes

```ts
nodes: ConfigNodeView[];
```

###### Inherited from

[`NodesResponse`](#nodesresponse).[`nodes`](#nodes-2)

***

### LibrarySummary

Library-wide health from `GET /api/library-status`.

#### Properties

##### active\_jobs

```ts
active_jobs: number;
```

##### branch\_nodes

```ts
branch_nodes: number;
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
name?: string | null;
```

***

### ListedSession

Active auth session row visible to the current principal.

#### Properties

##### channel?

```ts
channel?: "browser" | "clerk" | null;
```

For a portal session: `browser`, or `clerk` for a clerk's library card.

##### client\_label?

```ts
client_label?: string | null;
```

##### created\_at

```ts
created_at: string;
```

##### device\_type?

```ts
device_type?: string | null;
```

##### elevated?

```ts
elevated?: boolean;
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
impersonating_user_id?: number | null;
```

##### is\_current?

```ts
is_current?: boolean;
```

##### kind

```ts
kind: string;
```

##### last\_used\_at?

```ts
last_used_at?: string | null;
```

##### user\_agent?

```ts
user_agent?: string | null;
```

***

### ListedUser

User row from `GET /api/users` (administrator settings).

#### Properties

##### avatar\_source?

```ts
avatar_source?: string | null;
```

`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`.

##### controls?

```ts
controls?: MemberControls;
```

What the user may do here: the member controls that apply to them.

##### display\_name

```ts
display_name: string | null;
```

##### email?

```ts
email?: string | null;
```

##### gravatar\_hash?

```ts
gravatar_hash?: string | null;
```

SHA-256 hex of the contact email for Gravatar.

##### has\_avatar?

```ts
has_avatar?: boolean;
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
identities?: object[];
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
integrations?: object[];
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
last_active_at?: string | null;
```

Most recent unexpired portal session activity (RFC 3339). Active vs idle is ~5 minutes.

##### last\_seen\_at?

```ts
last_seen_at?: string | null;
```

Durable last portal use (RFC 3339); survives logout. Null means never signed in.

##### listening?

```ts
listening?: 
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
online?: boolean;
```

Non-expired portal session present (`true` even when idle).

##### role

```ts
role: string;
```

##### sso\_pictures?

```ts
sso_pictures?: object[];
```

IdP-supplied pictures the user can pick.

###### identity\_id

```ts
identity_id: number;
```

###### last\_used\_at?

```ts
last_used_at?: string | null;
```

###### picture\_path?

```ts
picture_path?: string | null;
```

Same-origin 128 px copy of `picture_url`, when the server signed one.

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

### Listing

One store listing of an edition.

#### Properties

##### holdings

```ts
holdings: Holding[];
```

Who holds it.

##### id

```ts
id: string;
```

Listing id.

##### marketplace

```ts
marketplace: string;
```

Marketplace code (`us`, `uk`, …).

##### plugin\_key

```ts
plugin_key: string;
```

Storefront plugin (`audible`, `libro`, …).

##### product\_id

```ts
product_id: string;
```

The store's product id (ASIN, ISBN, …).

##### url

```ts
url: string | null;
```

The listing's store page, when known.

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

### NodeHolder

A listing held by a branch library's node.

#### Properties

##### kind

```ts
kind: "node";
```

Discriminant.

##### node\_id

```ts
node_id: string;
```

The holding node's id.

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

- [`LibraryStatusResponse`](#librarystatusresponse)

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
client_secret?: string | null;
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
name?: string | null;
```

##### plugin\_id?

```ts
plugin_id?: string | null;
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
client_id?: string;
```

##### confidential

```ts
confidential: boolean;
```

##### current\_password?

```ts
current_password?: string;
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
name?: string | null;
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
current_password?: string;
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
public_origin?: string | null;
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
authorization_endpoint?: string;
```

##### callback\_url?

```ts
callback_url?: string | null;
```

##### detected\_origin?

```ts
detected_origin?: string;
```

Origin detected from this request when it is a bound loopback listen address.

##### discovery\_url?

```ts
discovery_url?: string;
```

Absolute OpenID discovery URL.

##### enabled

```ts
enabled: boolean;
```

##### issuer\_url?

```ts
issuer_url?: string;
```

Effective issuer origin (`public_origin`, else bound loopback, else localhost).

##### providers

```ts
providers: OidcProviderConfigView[];
```

##### public\_origin?

```ts
public_origin?: string | null;
```

Configured `integrations.public_origin` (issuer / callback / invite base).

##### revocation\_endpoint?

```ts
revocation_endpoint?: string;
```

##### token\_endpoint?

```ts
token_endpoint?: string;
```

##### userinfo\_endpoint?

```ts
userinfo_endpoint?: string;
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
preset?: string | null;
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
apple_key_id?: string | null;
```

##### apple\_private\_key?

```ts
apple_private_key?: string;
```

##### apple\_team\_id?

```ts
apple_team_id?: string | null;
```

##### client\_id

```ts
client_id: string;
```

##### client\_secret?

```ts
client_secret?: string;
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
issuer?: string | null;
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
preset?: string | null;
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
apple_key_id?: string | null;
```

##### apple\_team\_id?

```ts
apple_team_id?: string | null;
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
has_apple_private_key?: boolean;
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
issuer?: string | null;
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
preset?: string | null;
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

### PatronScope

Which part of the library a patron sees.

#### Properties

##### rules

```ts
rules: ScopeRule[];
```

Visibility rules; empty means every book is visible.

***

### PluginConsentBrand

Branded display metadata for a plugin consent dialog.

#### Properties

##### accent?

```ts
accent?: string | null;
```

##### bg?

```ts
bg?: string | null;
```

##### fg?

```ts
fg?: string | null;
```

##### logo?

```ts
logo?: string | null;
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
jail_cpu_cores?: number | null;
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

### PluginConsentPending

The difference between a plugin's request and its stored grant.

#### Properties

##### addressCidrs

```ts
addressCidrs: string[];
```

CIDRs the request adds.

##### allowUndeclaredPublicRedirects

```ts
allowUndeclaredPublicRedirects: boolean;
```

Whether the request newly asks to follow undeclared public redirects.

##### domains

```ts
domains: string[];
```

Fetch host patterns the request adds (`*` is any public host).

##### lines

```ts
lines: string[];
```

Operator-facing lines naming every capability awaiting consent (the CLI prints the same).

##### tcp

```ts
tcp: object[];
```

TCP `host:ports` the request adds, narrowed to the new ports.

###### host

```ts
host: string;
```

###### ports

```ts
ports: number[];
```

***

### PluginConsentResponse

Network/capability consent status for a plugin.

#### Properties

##### brand?

```ts
brand?: PluginConsentBrand;
```

##### covered

```ts
covered: boolean;
```

##### existing?

```ts
existing?: PluginGrant;
```

##### host\_user\_agent?

```ts
host_user_agent?: string;
```

The `User-Agent` the host sends, except to the request's `ownUserAgentHosts`.

##### limits

```ts
limits: PluginConsentLimits;
```

##### pending?

```ts
pending?: PluginConsentPending;
```

What the request asks for beyond the stored grant (a plugin upgrade that
widened its capabilities); absent when no grant is stored. The plugin runs
under the stored grant until these are approved.

##### plugin\_id

```ts
plugin_id: string;
```

##### plugin\_key?

```ts
plugin_key?: string;
```

##### provenance?

```ts
provenance?: string;
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
applied_config_revision?: number;
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
switch), `placed` / `unplaced` when the library scheduler does. The
host runs it for `present` and `placed`.

##### detail?

```ts
detail?: string;
```

##### node\_id

```ts
node_id: string;
```

##### observed\_at?

```ts
observed_at?: string;
```

##### revision

```ts
revision: number;
```

##### status?

```ts
status?: string;
```

`materialized`, `running`, `healthy`, `error`, or `retired` when observed.

***

### PluginGrant

Plugin consent grant shape serialized by the daemon.

#### Properties

##### addressCidrs?

```ts
addressCidrs?: string[];
```

##### allowUndeclaredPublicRedirects?

```ts
allowUndeclaredPublicRedirects?: boolean;
```

##### approvedAt

```ts
approvedAt: string;
```

##### authorityRevision?

```ts
authorityRevision?: string;
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
consumers?: object[];
```

Approved event-consumer subscriptions (type + schemas + suspend + filter).

###### eventType

```ts
eventType: string;
```

###### filter?

```ts
filter?: string | null;
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
cpuCores?: number;
```

Jail CPU in cores (2 d.p.) from the daemon.

##### cpuMs?

```ts
cpuMs?: number;
```

##### diskMib?

```ts
diskMib?: number;
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
extraProcesses?: number;
```

Extra process/thread budget beyond launcher overhead (native).

##### jobs?

```ts
jobs?: string[];
```

Approved `[triggers].jobs` types.

##### manifestDomains?

```ts
manifestDomains?: string[];
```

##### memoryMib?

```ts
memoryMib?: number;
```

##### networkMode

```ts
networkMode: string;
```

##### operatorAddedCidrs?

```ts
operatorAddedCidrs?: string[];
```

##### operatorAddedDomains?

```ts
operatorAddedDomains?: string[];
```

##### operatorAddedTcp?

```ts
operatorAddedTcp?: object[];
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
operatorDeniedCidrs?: string[];
```

##### operatorDeniedDomains?

```ts
operatorDeniedDomains?: string[];
```

##### operatorDeniedTcp?

```ts
operatorDeniedTcp?: object[];
```

###### host

```ts
host: string;
```

###### ports

```ts
ports: number[];
```

##### ownUserAgentHosts?

```ts
ownUserAgentHosts?: string[];
```

Hosts that receive the plugin's own `User-Agent`; every other request
carries the host's. Absent on a grant approved before this permission
existed, which keeps whatever the plugin declares.

##### ownUserAgentReason?

```ts
ownUserAgentReason?: string;
```

The plugin's reason for `ownUserAgentHosts`, from `plugin.toml`.

##### pluginId

```ts
pluginId: string;
```

##### pluginKey?

```ts
pluginKey?: string;
```

##### producers

```ts
producers: string[];
```

Event types the operator approved the plugin to publish.

##### subrequests?

```ts
subrequests?: number;
```

##### tcp?

```ts
tcp?: object[];
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
settings: Record<string, InstanceSettingValue>;
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
secrets?: Record<string, string>;
```

Secret values to seal; never echoed back.

##### settings

```ts
settings: Record<string, InstanceSettingValue>;
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

### PluginSettingColumn

One column of a `table` setting: the record field each row fills.

#### Properties

##### choices?

```ts
choices?: PluginSettingChoice[];
```

Dropdown values of an `enum` column.

##### default?

```ts
default?: string;
```

String form of the value a new row starts with.

##### key

```ts
key: string;
```

Field name in each row object.

##### label

```ts
label: string;
```

Column heading.

##### required?

```ts
required?: boolean;
```

True when every row must have a value in this column.

##### value\_type

```ts
value_type: "string" | "number" | "boolean";
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
choices?: PluginSettingChoice[];
```

##### columns?

```ts
columns?: PluginSettingColumn[];
```

Columns of a `table` setting, in display order.

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
secret?: boolean;
```

True for a `secret` setting; `value` is always empty.

##### secret\_set?

```ts
secret_set?: boolean;
```

For a secret setting, whether a sealed value is stored for the instance.

##### value

```ts
value: string;
```

String form of the value; for a `table` setting, the rows as JSON text.

##### value\_type

```ts
value_type: "string" | "number" | "boolean" | "table" | "secret";
```

***

### PluginSettingsGroup

Settings group for one plugin instance, or for a bootstrap database adapter.

#### Properties

##### config\_revision?

```ts
config_revision?: number;
```

Instance document revision for compare-and-swap writes.

##### deployment?

```ts
deployment?: PluginDeploymentView;
```

This node's deployment of the instance, when one exists.

##### display\_name?

```ts
display_name?: string;
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
is_default?: boolean;
```

True when a bare plugin key or alias routes to this instance.

##### logo?

```ts
logo?: string;
```

Same-origin logo for Settings list rows (embedded, or a copy of the remote logo).

##### placement?

```ts
placement?: PluginPlacementView;
```

Scheduler placement; absent when the instance is managed by hand.

##### plugin\_instance\_id?

```ts
plugin_instance_id?: string;
```

Instance this group edits; absent for a `config.toml`-backed database adapter.

##### plugin\_key?

```ts
plugin_key?: string;
```

Provenance-qualified PluginKey when this row came from a discovered install.

##### present\_on?

```ts
present_on?: string[];
```

Nodes whose deployment of the instance is `present`.

##### provenance?

```ts
provenance?: string;
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
brand?: PortalBrand;
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
currency?: string | null;
```

##### list\_price\_cents?

```ts
list_price_cents?: number | null;
```

##### list\_price\_label?

```ts
list_price_label?: string | null;
```

##### member\_price\_cents?

```ts
member_price_cents?: number | null;
```

##### member\_price\_label?

```ts
member_price_label?: string | null;
```

##### price\_cents?

```ts
price_cents?: number | null;
```

##### price\_label?

```ts
price_label?: string | null;
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
asin?: string | null;
```

##### authors?

```ts
authors?: string | null;
```

##### candidate\_product\_id?

```ts
candidate_product_id?: string | null;
```

##### candidate\_source?

```ts
candidate_source?: string | null;
```

##### isbn?

```ts
isbn?: string | null;
```

##### region?

```ts
region?: string;
```

##### store\_editions?

```ts
store_editions?: StoreEdition[];
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

##### incomplete?

```ts
incomplete?: boolean;
```

True when the server's batch deadline cut the lookup short: the hints are
links without prices, so the answer is not cached and should be asked
for again later.

***

### QueueWisher

Person who has a title on an open wishlist (global queue avatars).

#### Properties

##### avatar\_source?

```ts
avatar_source?: string | null;
```

Explicit picture choice (`auto` / `monogram` / `gravatar` / `upload` / `sso:{id}`).

##### display\_name?

```ts
display_name?: string | null;
```

Display name, identity label, or Operator.

##### gravatar\_hash?

```ts
gravatar_hash?: string | null;
```

SHA-256 hex of the contact email for Gravatar.

##### has\_avatar?

```ts
has_avatar?: boolean;
```

True when an uploaded avatar file exists.

##### identity\_id?

```ts
identity_id?: number | null;
```

Portal identity that created the wish.

##### login\_name?

```ts
login_name?: string | null;
```

Local login name when set.

##### operator?

```ts
operator?: boolean;
```

True when the wish came from the operator token.

##### picture\_path?

```ts
picture_path?: string | null;
```

Same-origin 128 px copy of `picture_url`, when the server signed one.

##### picture\_url?

```ts
picture_url?: string | null;
```

HTTPS picture from the wishing identity, when any; never loaded directly.

##### user\_id?

```ts
user_id?: number | null;
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

##### available\_through?

```ts
available_through?: BranchAvailability[];
```

Branches of this library that currently list this title.

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
categories?: string[];
```

Stable shelf-kind tags (`finish_series`, `author`, `requests`, …).

##### cover\_path?

```ts
cover_path?: string | null;
```

Same-origin path of this cover's processed images; see [coverSrc](#coversrc).

##### cover\_url?

```ts
cover_url?: string | null;
```

##### description?

```ts
description?: string | null;
```

##### from\_request

```ts
from_request: boolean;
```

##### genres?

```ts
genres?: string | null;
```

##### isbn

```ts
isbn: string | null;
```

##### language?

```ts
language?: string | null;
```

##### length\_minutes?

```ts
length_minutes?: number | null;
```

##### narrators

```ts
narrators: string | null;
```

##### published\_at?

```ts
published_at?: string | null;
```

##### publisher?

```ts
publisher?: string | null;
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
seed_categories?: string | null;
```

##### seed\_title?

```ts
seed_title?: string | null;
```

Title of the book or wish the storefront was asked about to find this card.

##### series

```ts
series: string | null;
```

##### series\_index

```ts
series_index: string | null;
```

##### starter\_genre?

```ts
starter_genre?: string | null;
```

Starter genre this card was browsed from while the taste was thin.

##### store\_editions?

```ts
store_editions?: StoreEdition[];
```

##### subtitle?

```ts
subtitle?: string | null;
```

##### title

```ts
title: string;
```

##### wish\_count?

```ts
wish_count?: number;
```

Distinct wisher count (may exceed `wishers.length`).

##### wishers?

```ts
wishers?: QueueWisher[];
```

People who have this title on an open wishlist.

##### work\_id

```ts
work_id: string | null;
```

##### work\_key?

```ts
work_key?: string;
```

Stable bibliographic key (`isbn:` / `asin:` / `soft:`…).

***

### RemoteApprovalBody

Body of an approval: the reviewed request and what of it to leave out.
An approval can only narrow the request, never widen it.

#### Properties

##### deny\_bindings?

```ts
deny_bindings?: string[];
```

Host bindings to remove (`secrets`, `oauth`, `kv`, …).

##### deny\_domains?

```ts
deny_domains?: string[];
```

Outbound domain patterns to remove from the grant.

##### deny\_network?

```ts
deny_network?: boolean;
```

True turns outbound network off altogether (no domains, TCP or CIDRs).

##### deny\_own\_user\_agent\_hosts?

```ts
deny_own_user_agent_hosts?: string[];
```

Requested hosts refused the plugin's own User-Agent; they get Bookclerk's agent.

##### library\_id

```ts
library_id: string;
```

Library of the request.

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

##### grant

```ts
grant: RemoteGrantView | null;
```

What this host granted (the request, narrowed), if approved.

##### library\_id

```ts
library_id: string;
```

Library that asks; matches a [LibraryLinkView.library\_id](#library_id).

##### pending?

```ts
pending?: string[];
```

What the current request asks for beyond the request this host approved,
as the lines the CLI prints; empty when nothing was approved yet.

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
requested_artifact?: object;
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
revoked_at?: string | null;
```

RFC 3339 time this host's operator revoked it; a revoke holds until approved again.

##### state

```ts
state: RemoteConsentState;
```

Whether the guest may run here now, or what it waits on.

##### updated\_at?

```ts
updated_at?: string;
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
addressCidrs?: string[];
```

CIDR ranges beyond the public Internet (private or local addresses).

##### allowUndeclaredPublicRedirects?

```ts
allowUndeclaredPublicRedirects?: boolean;
```

Follow a redirect to a public host it did not declare.

##### bindings?

```ts
bindings?: string[];
```

Host bindings (`config`, `secrets`, `kv`, …).

##### compatibilityFlags?

```ts
compatibilityFlags?: string[];
```

workerd compatibility flags the isolate runs with.

##### consumers?

```ts
consumers?: unknown[];
```

Events and queues it consumes (shape owned by the host).

##### cpuMs?

```ts
cpuMs?: number | null;
```

workerd CPU budget per request, in milliseconds.

##### cpuRatePercent?

```ts
cpuRatePercent?: number | null;
```

CPU rate as a percent of one logical CPU (above 100 asks for more than one core).

##### diskMib?

```ts
diskMib?: number | null;
```

Disk budget in MiB for each of its `data/` and `tmp/` directories.

##### domains?

```ts
domains?: string[];
```

Outbound domain patterns the guest may dial first (redirect hops aside).

##### entrypoints?

```ts
entrypoints?: string[];
```

Named entrypoints it exports (`storefront`, `storage`, `cli`, …).

##### extraProcesses?

```ts
extraProcesses?: number | null;
```

Processes or threads allowed beyond the launcher's own (native guests).

##### jobs?

```ts
jobs?: string[];
```

`[triggers].jobs` types it may handle.

##### memoryMib?

```ts
memoryMib?: number | null;
```

Memory ceiling in MiB.

##### networkMode?

```ts
networkMode?: string;
```

`deny` (no egress) or `outbound`; anything else is treated as `deny`.

##### ownUserAgentHosts?

```ts
ownUserAgentHosts?: string[] | null;
```

Hosts that receive the plugin's own `User-Agent` instead of the host's.

##### ownUserAgentReason?

```ts
ownUserAgentReason?: string | null;
```

The plugin's reason for `ownUserAgentHosts`.

##### pluginId?

```ts
pluginId?: string;
```

Display alias from `plugin.toml` (not security authoritative).

##### producers?

```ts
producers?: string[];
```

Event types it may publish and queue bindings it may send to.

##### subrequests?

```ts
subrequests?: number | null;
```

workerd outbound fetch (subrequest) budget per request.

##### tcp?

```ts
tcp?: object[];
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

### ScopeRule

One rule of a patron scope.

#### Properties

##### effect

```ts
effect: RuleEffect;
```

Include or exclude.

##### field

```ts
field: ScopeField;
```

The field the rule matches.

##### value

```ts
value: string;
```

One value of the field, compared without case.

***

### SettingsResponse

Flat config map plus per-plugin setting groups.

#### Properties

##### effective?

```ts
effective?: Record<string, string>;
```

##### host\_cpu\_cores\_max?

```ts
host_cpu_cores_max?: number;
```

Host max jail CPU in cores (2 d.p.; equals logical CPU count).

##### jail\_cpu\_cores?

```ts
jail_cpu_cores?: number | null;
```

Optional global per-jail CPU ceiling in cores.

##### output\_revision?

```ts
output_revision?: number;
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
events?: EventQueueStatus;
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
asin?: string | null;
```

##### authors?

```ts
authors?: string | null;
```

##### categories?

```ts
categories?: string | null;
```

##### cover\_path?

```ts
cover_path?: string | null;
```

Same-origin path of this cover's processed images; see [coverSrc](#coversrc).

##### cover\_url?

```ts
cover_url?: string | null;
```

##### description?

```ts
description?: string | null;
```

##### is\_abridged?

```ts
is_abridged?: boolean | null;
```

`true` abridged / `false` unabridged when the storefront said so.

##### isbn?

```ts
isbn?: string | null;
```

##### language?

```ts
language?: string | null;
```

##### length\_minutes?

```ts
length_minutes?: number | null;
```

##### narrators?

```ts
narrators?: string | null;
```

##### published\_at?

```ts
published_at?: string | null;
```

##### publisher?

```ts
publisher?: string | null;
```

##### rating\_count?

```ts
rating_count?: number | null;
```

##### rating\_overall?

```ts
rating_overall?: number | null;
```

Audible community ratings (detail fetch only).

##### rating\_performance?

```ts
rating_performance?: number | null;
```

##### rating\_story?

```ts
rating_story?: number | null;
```

##### review\_count?

```ts
review_count?: number | null;
```

##### ~~reviews?~~

```ts
reviews?: TitleReview[];
```

###### Deprecated

Prefer [`fetchTitleReviews`]. Title-meta no longer embeds reviews.

##### series?

```ts
series?: string | null;
```

##### series\_index?

```ts
series_index?: string | null;
```

##### subtitle?

```ts
subtitle?: string | null;
```

##### title?

```ts
title?: string | null;
```

***

### TitleRequest

Wishlist / title-request row from `/api/wishlist`.

Represents a user ask for a work (and optional acquire queue linkage via
`resolved_edition_id`, the edition that fulfilled it).

#### Properties

##### asin

```ts
asin: string | null;
```

##### authors

```ts
authors: string | null;
```

##### cover\_path?

```ts
cover_path?: string | null;
```

Same-origin path of this cover's processed images; see [coverSrc](#coversrc).

##### cover\_url?

```ts
cover_url?: string | null;
```

##### created\_at

```ts
created_at: string;
```

##### description?

```ts
description?: string | null;
```

##### edition\_id?

```ts
edition_id?: string | null;
```

Edition the wish named, when its identifiers reached exactly one.

##### genres?

```ts
genres?: string | null;
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
language?: string | null;
```

##### length\_minutes?

```ts
length_minutes?: number | null;
```

##### narrators?

```ts
narrators?: string | null;
```

##### notes

```ts
notes: string | null;
```

##### published\_at?

```ts
published_at?: string | null;
```

##### publisher?

```ts
publisher?: string | null;
```

##### purchase\_hints?

```ts
purchase_hints?: PurchaseHint[];
```

##### resolved\_edition\_id

```ts
resolved_edition_id: string | null;
```

##### series?

```ts
series?: string | null;
```

##### series\_index?

```ts
series_index?: string | null;
```

##### status

```ts
status: string;
```

##### store\_editions?

```ts
store_editions?: StoreEdition[];
```

##### subtitle?

```ts
subtitle?: string | null;
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
work_id: string;
```

Library work the wish is for; every wish names one.

##### work\_key

```ts
work_key: string;
```

Bibliographic key derived from the wish's identifiers, for display matching.

***

### TitleReview

One Audible customer review for title detail.

#### Properties

##### author\_name?

```ts
author_name?: string | null;
```

##### body

```ts
body: string;
```

##### id?

```ts
id?: string | null;
```

##### overall\_rating?

```ts
overall_rating?: number | null;
```

##### performance\_rating?

```ts
performance_rating?: number | null;
```

##### story\_rating?

```ts
story_rating?: number | null;
```

##### submitted\_at?

```ts
submitted_at?: string | null;
```

##### title?

```ts
title?: string | null;
```

***

### UserControls

A patron's member controls as `GET /api/users/{id}/controls` reports them.

#### Properties

##### applies

```ts
applies: boolean;
```

False for an Owner, who may always do everything.

##### defaults

```ts
defaults: MemberControls;
```

The library's default for every patron.

##### effective

```ts
effective: MemberControls;
```

What applies to the user now.

##### follows\_default

```ts
follows_default: boolean;
```

True for a Member, who follows the library default; false for an
Administrator, whom only controls set for them alone limit.

##### overrides

```ts
overrides: ControlOverrides;
```

The controls set for this patron alone.

##### user\_id

```ts
user_id: number;
```

The user.

***

### UserMutationResponse

Created / patched user response from RBAC provisioning APIs.

#### Properties

##### claim\_ticket?

```ts
claim_ticket?: string | null;
```

##### invite\_url?

```ts
invite_url?: string | null;
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

***

### UserScope

A patron's scope as `GET`/`PUT /api/users/{id}/scope` report it.

#### Properties

##### applies

```ts
applies: boolean;
```

False for an Owner or Administrator, who always see everything.

##### own

```ts
own: boolean;
```

True when the user has their own scope.

##### scope

```ts
scope: PatronScope;
```

The scope that applies: their own, or the library default.

##### user\_id

```ts
user_id: number;
```

The user.

***

### WorkDetail

One work from `GET /api/library/works/{id}`.

#### Properties

##### authors

```ts
authors: string[];
```

Authors (or hosts).

##### can\_edit\_state?

```ts
can_edit_state?: boolean;
```

Whether the caller can keep their own state (false for the operator token).

##### description

```ts
description: string | null;
```

Description, when known (may contain light HTML).

##### editions

```ts
editions: Edition[];
```

Its editions.

##### id

```ts
id: string;
```

Work id.

##### kind

```ts
kind: WorkKind;
```

Book or podcast.

##### series

```ts
series: string | null;
```

Series name, when in one.

##### series\_position

```ts
series_position: string | null;
```

Position within the series.

##### state?

```ts
state?: WorkState | null;
```

The caller's own state; absent when they recorded none.

##### title

```ts
title: string;
```

Display title.

***

### WorksPage

A page of works and the cursor of the next one.

#### Properties

##### next\_cursor?

```ts
next_cursor?: string | null;
```

Pass as `cursor` for the next page; absent on the last.

##### works

```ts
works: WorkSummary[];
```

The page's works.

***

### WorksQuery

Filters and paging for [fetchWorks](#fetchworks).

#### Properties

##### account?

```ts
account?: string;
```

Only works held on this account id.

##### cursor?

```ts
cursor?: string;
```

The previous page's `next_cursor`.

##### facet?

```ts
facet?: WorkFacet;
```

Only works with [WorksQuery.value](#value-5) under this facet.

##### kind?

```ts
kind?: WorkKind;
```

Only books or only podcasts.

##### limit?

```ts
limit?: number;
```

Page size (the daemon caps it at 200).

##### media\_kind?

```ts
media_kind?: MediaKind;
```

Only works with an edition of this media kind.

##### q?

```ts
q?: string;
```

Search text; matches come best first and `sort` is ignored.

##### sort?

```ts
sort?: WorkSort;
```

Page order without `q`.

##### status?

```ts
status?: AcquireStatus | "all";
```

Only works in this acquire state (`all` or omitted for every state).

##### value?

```ts
value?: string;
```

The facet's value; an empty series value means works in no series.

***

### WorkState

The caller's own state for a work.

#### Properties

##### is\_finished

```ts
is_finished: boolean;
```

Whether they finished it.

##### rating\_overall

```ts
rating_overall: number | null;
```

Overall rating (0–5).

##### rating\_performance

```ts
rating_performance: number | null;
```

Performance rating (0–5).

##### rating\_story

```ts
rating_story: number | null;
```

Story rating (0–5).

##### tags

```ts
tags: string | null;
```

Space-separated tags.

***

### WorkStateEdit

A change to the caller's own state for a work. A field left out keeps
its value; `null` clears a rating or the tags.

#### Properties

##### is\_finished?

```ts
is_finished?: boolean;
```

Whether they finished it.

##### rating\_overall?

```ts
rating_overall?: number | null;
```

Overall rating (0–5).

##### rating\_performance?

```ts
rating_performance?: number | null;
```

Performance rating (0–5).

##### rating\_story?

```ts
rating_story?: number | null;
```

Story rating (0–5).

##### tags?

```ts
tags?: string | null;
```

Whitespace-separated tags (at most 32, each at most 64 bytes).

***

### WorkSummary

One work row from `GET /api/library/works`: a book or podcast once,
however many editions and accounts hold it.

#### Properties

##### acquire\_status

```ts
acquire_status: AcquireStatus;
```

The work's acquire state, summed over its editions.

##### added\_at

```ts
added_at: string;
```

When the newest holding was added (ISO-8601).

##### authors

```ts
authors: string | null;
```

Authors (or hosts), comma-separated, when known.

##### cover\_edition\_id

```ts
cover_edition_id: string | null;
```

Edition whose cover the row shows.

##### cover\_path

```ts
cover_path: string | null;
```

Same-origin path of that cover's processed images; see [coverSrc](#coversrc).

##### edition\_count

```ts
edition_count: number;
```

Editions of the work in the library.

##### holding\_count

```ts
holding_count: number;
```

Holdings of the work's listings.

##### id

```ts
id: string;
```

Work id; opens the detail at `/api/library/works/{id}`.

##### kind

```ts
kind: WorkKind;
```

Book or podcast.

##### length\_minutes?

```ts
length_minutes?: number | null;
```

Runtime in minutes of the longest held edition, when known.

##### media\_kinds

```ts
media_kinds: MediaKind[];
```

Media kinds the work's editions come in.

##### purchased\_at?

```ts
purchased_at?: string | null;
```

When the newest purchase a store reported happened (ISO-8601).

##### series

```ts
series: string | null;
```

Series name when the work is in one.

##### series\_position

```ts
series_position: string | null;
```

Position within the series (text, so `1.5` keeps its spelling).

##### title

```ts
title: string;
```

Display title.

## Type Aliases

### Abridgement

```ts
type Abridgement = "unknown" | "unabridged" | "abridged";
```

Whether an expression is abridged.

***

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
type AuthRole = "operator" | "owner" | "administrator" | "member";
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
authors?: string[];
```

##### exclude\_narrators?

```ts
exclude_narrators?: string[];
```

##### exclude\_sources?

```ts
exclude_sources?: string[];
```

##### genres?

```ts
genres?: string[];
```

##### languages?

```ts
languages?: string[];
```

##### max\_length\_minutes?

```ts
max_length_minutes?: number;
```

##### min\_length\_minutes?

```ts
min_length_minutes?: number;
```

##### min\_rating?

```ts
min_rating?: number;
```

##### narrators?

```ts
narrators?: string[];
```

##### series?

```ts
series?: string[];
```

##### sources?

```ts
sources?: string[];
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

### ControlOverrides

```ts
type ControlOverrides = Partial<Record<MemberControl, boolean>>;
```

One patron's overrides; a missing control follows the library's default.

***

### CoverSize

```ts
type CoverSize = 128 | 360 | "full";
```

A processed cover size the daemon serves: thumbnail, card, or full size.

***

### DiscoverFeedState

```ts
type DiscoverFeedState = "ready" | "refreshing" | "preparing";
```

How current a stored Discover feed is.

***

### Entitlement

```ts
type Entitlement = "purchase" | "plus" | "loan" | "free";
```

How a holder came to hold a listing.

***

### ExpressionKind

```ts
type ExpressionKind = "text" | "narration" | "dramatisation" | "graphic" | "original_audio";
```

How an expression renders its work.

***

### Holder

```ts
type Holder = AccountHolder | NodeHolder;
```

Who holds a listing: a store account, or a branch library's node.

***

### InstanceSettingValue

```ts
type InstanceSettingValue = SettingCellValue | SettingRow[];
```

One value in a plugin instance document.

***

### MediaKind

```ts
type MediaKind = "audio" | "ebook" | "print";
```

How an edition is consumed.

***

### MemberControl

```ts
type MemberControl = 
  | "link_accounts"
  | "download"
  | "edit_metadata"
  | "upload"
  | "request_books"
  | "see_prices";
```

One thing an administrator may allow or refuse a patron.

***

### MemberControls

```ts
type MemberControls = Record<MemberControl, boolean>;
```

A value for every member control (`true` = allowed).

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

### RuleEffect

```ts
type RuleEffect = "include" | "exclude";
```

Whether a patron scope rule lets matching books in or keeps them out.

***

### ScopeField

```ts
type ScopeField = "genre" | "series" | "author" | "narrator";
```

The book field a patron scope rule matches one value of.

***

### SettingCellValue

```ts
type SettingCellValue = string | number | boolean;
```

One stored cell of a `table` setting row.

***

### SettingRow

```ts
type SettingRow = Record<string, SettingCellValue>;
```

One stored row of a `table` setting, keyed by column.

***

### TitleMetaQuery

```ts
type TitleMetaQuery = object;
```

Query for public title metadata enrichment.

#### Properties

##### asin?

```ts
asin?: string | null;
```

##### authors?

```ts
authors?: string | null;
```

##### isbn?

```ts
isbn?: string | null;
```

##### length\_minutes?

```ts
length_minutes?: number | null;
```

##### narrators?

```ts
narrators?: string | null;
```

##### region?

```ts
region?: string;
```

##### store\_editions?

```ts
store_editions?: StoreEdition[] | null;
```

Store editions that listed the title. The server fills a sparse record
from the stores that answer their own product ids before asking ISBN
stores.

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
sort_by?: string;
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
page?: number;
```

##### page\_size?

```ts
page_size?: number;
```

##### region?

```ts
region?: string;
```

##### sort\_by?

```ts
sort_by?: TitleReviewsSort;
```

***

### TitleReviewsSort

```ts
type TitleReviewsSort = "MostHelpful" | "MostRecent";
```

Audible review sort modes for title detail.

***

### WorkFacet

```ts
type WorkFacet = 
  | "author"
  | "narrator"
  | "series"
  | "genre"
  | "publisher"
  | "source"
  | "tag"
  | "progress";
```

What `facet` and `value` filter works by: a value list of the page index,
the caller's own tags, or their progress (`finished` / `not_finished`).

***

### WorkKind

```ts
type WorkKind = "book" | "podcast";
```

What a work is: a book, or a podcast (its show).

***

### WorkSort

```ts
type WorkSort = "title" | "author" | "series" | "purchased" | "added" | "length";
```

How `/api/library/works` orders a page (ignored while searching).

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

### acquirableEditions()

```ts
function acquirableEditions(work): string[];
```

The editions of `work` an acquire should queue: those someone holds
that are not acquired or on their way.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `work` | [`WorkDetail`](#workdetail) | A work's detail. |

#### Returns

`string`[]

Edition ids for [triggerAcquire](#triggeracquire)'s `editions`.

***

### approveLibraryConsent()

```ts
function approveLibraryConsent(body): Promise<{
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
| `grant?` | \{ `addressCidrs?`: `string`[]; `allowUndeclaredPublicRedirects?`: `boolean`; `bindings`: `string`[]; `compatibilityFlags`: `string`[]; `cpuCores?`: `number`; `cpuMs?`: `number`; `diskMib?`: `number`; `domains`: `string`[]; `extraProcesses?`: `number`; `memoryMib?`: `number`; `networkMode`: `string`; `ownUserAgentHosts?`: `string`[]; `subrequests?`: `number`; `tcp?`: `object`[]; \} | Operator-approved grant overlay; omitted fields keep defaults. |
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
| `grant.ownUserAgentHosts?` | `string`[] | - |
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

A 503 with `Retry-After` (the session could not be checked yet) is asked
again a few times; only a 401 means signed out.

#### Returns

`Promise`\<[`AuthSession`](#authsession)\>

Anonymous session when the response is 401.

#### Throws

When the check fails or times out, which is not a
  sign-out.

***

### authMeUntilAnswered()

```ts
function authMeUntilAnswered(
   stopped, 
   check?, 
   sleep?
): Promise<AuthSession | null>;
```

Asks [authMe](#authme) until it answers, backing off between attempts.

A check that timed out, could not reach the server, or found it busy
(502, 503 or 504) is not a sign-out, so the app keeps its loading screen
and asks again rather than showing the sign-in page to someone whose
session is fine. Waits 1 s, 2 s, 4 s … up to 10 s. Any other failure is
thrown to the caller.

#### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `stopped` | () => `boolean` | `undefined` | True once the caller no longer wants the answer. |
| `check?` | () => `Promise`\<[`AuthSession`](#authsession)\> | `authMe` | Performs one session check (defaults to [authMe](#authme)). |
| `sleep?` | (`ms`) => `Promise`\<`void`\> | `...` | Waits the given milliseconds (defaults to a timer). |

#### Returns

`Promise`\<[`AuthSession`](#authsession) \| `null`\>

The session, or null when `stopped` turned true first.

#### Throws

When the check fails in a way asking again cannot fix.

***

### availableThroughLabel()

```ts
function availableThroughLabel(through): string | null;
```

One line naming the branches a title is available through.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `through` | [`BranchAvailability`](#branchavailability)[] \| `undefined` | The title's branch availability. |

#### Returns

`string` \| `null`

For example "Available through Riley's branch", or `null` when
  no branch lists the title.

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

### branchesToAsk()

```ts
function branchesToAsk(through): object[];
```

The distinct branches a title is available through, one entry per
branch, in the order given, with the label to ask each by.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `through` | [`BranchAvailability`](#branchavailability)[] \| `undefined` | The title's branch availability. |

#### Returns

`object`[]

For example `[{ node_id, work_id, label: "Riley's branch" }]`.

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

### coverSrc()

```ts
function coverSrc(path, size): string | null;
```

The same-origin URL of one processed size of a remote cover.

The daemon hands out `cover_path` (`/api/covers/<token>`) for every remote
`https` cover, so the web app never loads `cover_url` from another site.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `path` | `string` \| `null` \| `undefined` | A `cover_path` from an API response. |
| `size` | [`CoverSize`](#coversize) | The size to load. |

#### Returns

`string` \| `null`

The image URL, or `null` when there is no usable path.

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
| `body` | \{ `asin?`: `string`; `authors?`: `string`; `cover_url?`: `string` \| `null`; `description?`: `string` \| `null`; `genres?`: `string` \| `null`; `isbn?`: `string`; `language?`: `string` \| `null`; `length_minutes?`: `number` \| `null`; `narrators?`: `string` \| `null`; `notes?`: `string`; `published_at?`: `string` \| `null`; `publisher?`: `string` \| `null`; `purchase_hints?`: [`PurchaseHint`](#purchasehint)[]; `series?`: `string` \| `null`; `series_index?`: `string` \| `null`; `store_editions?`: [`StoreEdition`](#storeedition)[]; `subtitle?`: `string` \| `null`; `title`: `string`; \} | Title fields and optional catalog enrichment. |
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

### editionCoverUrl()

```ts
function editionCoverUrl(editionId): string;
```

Builds the authenticated URL of the cover an acquire wrote beside an
edition's file (404 when this node has none).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `editionId` | `string` | Edition id. |

#### Returns

`string`

Relative URL under `/api/library/editions/.../cover`.

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

### fetchCatalogListing()

```ts
function fetchCatalogListing(): Promise<CatalogListingView>;
```

Whether the signed-in user's books are listed to the libraries this
server is a branch of (`GET /api/auth/profile/catalog-listing`).

#### Returns

`Promise`\<[`CatalogListingView`](#cataloglistingview)\>

The choice, and how many libraries it applies to.

***

### fetchDiagnosticsReports()

```ts
function fetchDiagnosticsReports(): Promise<DiagnosticsReport[]>;
```

Lists the diagnostics reports this install sent (operator only).

#### Returns

`Promise`\<[`DiagnosticsReport`](#diagnosticsreport)[]\>

Reports, newest first (the install keeps the last 200).

***

### fetchDiscoverFeed()

```ts
function fetchDiscoverFeed(): Promise<DiscoverFeed>;
```

Loads the caller's stored Discover shelves.

#### Returns

`Promise`\<[`DiscoverFeed`](#discoverfeed)\>

Discover feed payload; `state` says whether shelves are still being prepared.

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

### fetchFacetValues()

```ts
function fetchFacetValues(facet, cursor?): Promise<FacetValuesPage>;
```

Fetches one page of a facet's values across the library's held works.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `facet` | \| `"source"` \| `"author"` \| `"series"` \| `"narrator"` \| `"genre"` \| `"publisher"` \| `"tag"` | The facet (not `progress`, whose values are fixed). |
| `cursor?` | `string` | The previous page's `next_cursor`; omitted for the first. |

#### Returns

`Promise`\<[`FacetValuesPage`](#facetvaluespage)\>

The page and the next page's cursor.

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

### fetchLibraryStatus()

```ts
function fetchLibraryStatus(): Promise<LibraryStatusResponse>;
```

Library health every node administrator may see, with detail for the
caller's visible nodes.

#### Returns

`Promise`\<[`LibraryStatusResponse`](#librarystatusresponse)\>

Library summary, visible nodes, and the caller's capabilities.

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

Resolves purchase hints through a batch shared with every other card.

Lookups asked close together go out as one batch request, one batch at a
time, after the covers on screen have loaded (see `lookupBatcher.ts`).

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

### fetchRetryingBusy()

```ts
function fetchRetryingBusy(
   input, 
   init, 
   fetcher?, 
   sleep?
): Promise<Response>;
```

Fetches a read, asking again while the daemon answers 503 with
`Retry-After`.

The daemon answers that way when it could not check the session yet (a
busy library, as on a small server that is still starting), which is not
the same as being signed out. Only for GET requests, which are safe to
repeat.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `input` | `string` | Request URL. |
| `init` | `RequestInit` \| `undefined` | Request options; the method must be GET. |
| `fetcher?` | (`input`, `init?`) => `Promise`\<`Response`\> | Performs one request (defaults to `fetch`). |
| `sleep?` | (`ms`) => `Promise`\<`void`\> | Waits the given milliseconds (defaults to a timer). |

#### Returns

`Promise`\<`Response`\>

The first response that is not such a 503, or the last 503.

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

Fetches title metadata through a batch shared with every other card (see
[fetchPurchaseHints](#fetchpurchasehints)).

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

### fetchWork()

```ts
function fetchWork(id): Promise<WorkDetail>;
```

Fetches one work with its editions, listings, holdings and own state.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | Work id. |

#### Returns

`Promise`\<[`WorkDetail`](#workdetail)\>

The work.

***

### fetchWorks()

```ts
function fetchWorks(query): Promise<WorksPage>;
```

Fetches one page of the library's works.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `query` | [`WorksQuery`](#worksquery) | Filters, order and the cursor of the page to read. |

#### Returns

`Promise`\<[`WorksPage`](#workspage)\>

The page and the next page's cursor.

***

### getDefaultControls()

```ts
function getDefaultControls(): Promise<{
  controls: MemberControls;
}>;
```

Reads the library's default member controls.

#### Returns

`Promise`\<\{
  `controls`: [`MemberControls`](#membercontrols);
\}\>

The default.

***

### getDefaultScope()

```ts
function getDefaultScope(): Promise<{
  scope: PatronScope;
}>;
```

Reads the library's default patron scope.

#### Returns

`Promise`\<\{
  `scope`: [`PatronScope`](#patronscope);
\}\>

The default.

***

### getUserControls()

```ts
function getUserControls(id): Promise<UserControls>;
```

Reads what a patron may do.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | The user. |

#### Returns

`Promise`\<[`UserControls`](#usercontrols)\>

The default, the patron's overrides and what applies.

***

### getUserScope()

```ts
function getUserScope(id): Promise<UserScope>;
```

Reads which part of the library a patron sees.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | The user. |

#### Returns

`Promise`\<[`UserScope`](#userscope)\>

The scope that applies and where it comes from.

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

### listLibraryConsents()

```ts
function listLibraryConsents(): Promise<LibraryConsentsResponse>;
```

Lists the libraries this host is enrolled in and what each asks to
run here. Operator only.

#### Returns

`Promise`\<[`LibraryConsentsResponse`](#libraryconsentsresponse)\>

Links and consent rows.

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

Resolves when the library answered that the sign-out is done.

#### Throws

When the library answers anything else.

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

### putDefaultControls()

```ts
function putDefaultControls(controls): Promise<{
  controls: MemberControls;
}>;
```

Replaces the library's default member controls.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `controls` | [`MemberControls`](#membercontrols) | The new default. |

#### Returns

`Promise`\<\{
  `controls`: [`MemberControls`](#membercontrols);
\}\>

The default after the change.

***

### putDefaultScope()

```ts
function putDefaultScope(scope): Promise<{
  scope: PatronScope;
}>;
```

Replaces the library's default patron scope.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `scope` | [`PatronScope`](#patronscope) | The new default. |

#### Returns

`Promise`\<\{
  `scope`: [`PatronScope`](#patronscope);
\}\>

The default after the change.

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

Hands an instance's deployments to the library scheduler, or changes its
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

### putUserControls()

```ts
function putUserControls(id, overrides): Promise<UserControls>;
```

Sets the controls that differ for one patron; the rest follow the
library's default.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | The user. |
| `overrides` | [`ControlOverrides`](#controloverrides) | The controls set for them alone (`{}` for none). |

#### Returns

`Promise`\<[`UserControls`](#usercontrols)\>

The controls after the change.

***

### putUserScope()

```ts
function putUserScope(id, scope): Promise<UserScope>;
```

Gives a patron their own scope, or with `null` returns them to the
library's default.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `number` | The user. |
| `scope` | [`PatronScope`](#patronscope) \| `null` | Their own scope, or `null`. |

#### Returns

`Promise`\<[`UserScope`](#userscope)\>

The scope after the change.

***

### putWorkState()

```ts
function putWorkState(id, edit): Promise<WorkState>;
```

Changes the caller's own finished flag, ratings or tags for a work.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `id` | `string` | The work. |
| `edit` | [`WorkStateEdit`](#workstateedit) | The fields to change. |

#### Returns

`Promise`\<[`WorkState`](#workstate)\>

The caller's state after the change.

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

### requestDiscoverRefresh()

```ts
function requestDiscoverRefresh(): Promise<DiscoverStatus>;
```

Queues a storefront fetch and ranking for the caller's shelves now, as
the Refresh button asks. A fetch already waiting absorbs the request.

#### Returns

`Promise`\<[`DiscoverStatus`](#discoverstatus)\>

The caller's Discover status once the fetch is queued.

***

### requestFromBranch()

```ts
function requestFromBranch(workId, nodeId): Promise<BranchRequestQueued>;
```

Asks a branch for a book it lists. The branch takes it automatically;
the book arrives with the branch's next catalog sync.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `workId` | `string` | The title's work (`BranchAvailability.work_id`). |
| `nodeId` | `string` | The branch to ask. |

#### Returns

`Promise`\<[`BranchRequestQueued`](#branchrequestqueued)\>

The queued request.

#### Throws

404 when the branch no longer lists the title.

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

### revokeLibraryConsent()

```ts
function revokeLibraryConsent(libraryId, pluginKey): Promise<{
  consent: RemoteConsentView;
}>;
```

Withdraws an approval: held jobs of the plugin are cancelled and given
back, and nothing of it is claimed until approved again.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `libraryId` | `string` | Library. |
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

### saveDiagnosticsSharing()

```ts
function saveDiagnosticsSharing(shareReports, shareUsage): Promise<SettingsResponse>;
```

Saves the install's diagnostics sharing choices (`diagnostics.share_reports`
and `diagnostics.share_usage`). The daemon applies them without a restart.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `shareReports` | `boolean` | Upload redacted reports after crashes and error bursts. |
| `shareUsage` | `boolean` | Send the weekly anonymous check-in and tag reports with the install ID. |

#### Returns

`Promise`\<[`SettingsResponse`](#settingsresponse)\>

The settings view after the write.

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
| `opts?` | \{ `allLanguages?`: `boolean`; `cursor?`: `string` \| `null`; `field?`: `"author"` \| `"series"` \| `"narrator"` \| `"genre"`; `filters?`: [`CatalogSearchFilters`](#catalogsearchfilters); `lang?`: `string`; `limit?`: `number`; `page_size?`: `number`; `sort?`: [`CatalogSearchSort`](#catalogsearchsort); `sort_dir?`: [`CatalogSortDir`](#catalogsortdir); \} | Pagination cursor, sort, language, and server-side facet filters. |
| `opts.allLanguages?` | `boolean` | When true, do not hard-filter by language. |
| `opts.cursor?` | `string` \| `null` | - |
| `opts.field?` | `"author"` \| `"series"` \| `"narrator"` \| `"genre"` | - |
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

### setCatalogListing()

```ts
function setCatalogListing(listed): Promise<CatalogListingView>;
```

Sets whether the signed-in user's books are listed to the libraries this
server is a branch of (`PUT /api/auth/profile/catalog-listing`).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `listed` | `boolean` | `false` keeps the user's books out of catalog answers. |

#### Returns

`Promise`\<[`CatalogListingView`](#cataloglistingview)\>

The stored choice.

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

Queues an acquire: everything pending when the body names nothing, or
the named editions or title.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | \{ `asin?`: `string`; `editions?`: `string`[]; `uuid?`: `string`; \} | `editions` (ids from a work's detail), or a title by `uuid` / `asin`. |
| `body.asin?` | `string` | - |
| `body.editions?` | `string`[] | - |
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
