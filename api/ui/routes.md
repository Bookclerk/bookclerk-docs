# routes

## Type Aliases

### DiscoverSearchHandoff

```ts
type DiscoverSearchHandoff = object;
```

Optional post-search filter/sort applied after a Discover catalog query.

#### Properties

##### authors?

```ts
optional authors?: string[];
```

Seed include filters (server-applied).

##### excludeSources?

```ts
optional excludeSources?: string[];
```

##### field?

```ts
optional field?: "author" | "narrator" | "series" | "genre";
```

Storefront-scoped search (`author` / `narrator` / `series` / `genre`).

##### ~~filterKind?~~

```ts
optional filterKind?: "all" | "authors" | "series" | "narrators" | "genres" | "sources";
```

###### Deprecated

legacy single-dimension filter

##### ~~filterValue?~~

```ts
optional filterValue?: string;
```

###### Deprecated

##### genres?

```ts
optional genres?: string[];
```

##### hideVirtualVoice?

```ts
optional hideVirtualVoice?: boolean;
```

When false, do not exclude Virtual Voice (default hide is on).

##### language?

```ts
optional language?: string;
```

Hard language include (`en`, `zh`, …) or `__all__` for no hard filter.
Omit to use the browser language default.

##### minRating?

```ts
optional minRating?: number;
```

##### narrators?

```ts
optional narrators?: string[];
```

##### q

```ts
q: string;
```

##### runtimeBucket?

```ts
optional runtimeBucket?: "any" | "under6" | "6to12" | "12to20" | "over20";
```

##### series?

```ts
optional series?: string[];
```

##### sort?

```ts
optional sort?: 
  | "relevance"
  | "popularity"
  | "rating"
  | "title"
  | "author"
  | "price"
  | "length";
```

##### sortDir?

```ts
optional sortDir?: "asc" | "desc";
```

##### ~~sortKey?~~

```ts
optional sortKey?: 
  | "title"
  | "author"
  | "series"
  | "relevance"
  | "popularity"
  | "rating"
  | "price"
  | "length";
```

###### Deprecated

##### sources?

```ts
optional sources?: string[];
```

## Variables

### JOIN\_APPROVE\_PATH

```ts
const JOIN_APPROVE_PATH: "/join/approve" = "/join/approve";
```

The library's consent page for a branch's sign-in.

***

### JOIN\_CALLBACK\_PATH

```ts
const JOIN_CALLBACK_PATH: "/join/callback" = "/join/callback";
```

Where a library sends the joining install's browser back.

## Functions

### dropTicketUnlessInvitePath()

```ts
function dropTicketUnlessInvitePath(): void;
```

Invite magic links are `/invite?ticket=` only. Drop `ticket` on any other
path so `/discover?ticket=` cannot open the accept-invite form.

#### Returns

`void`

***

### isAppPath()

```ts
function isAppPath(pathname): boolean;
```

Returns whether `pathname` is a known GUI route (not an API/static/unknown URL).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `pathname` | `string` | Document pathname (may include a trailing slash). |

#### Returns

`boolean`

True when the path is owned by the React SPA.

***

### isInvitePath()

```ts
function isInvitePath(pathname?): boolean;
```

True when this document is the magic-link invite route.

#### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `pathname` | `string` | `window.location.pathname` | Document pathname. |

#### Returns

`boolean`

True for `/invite` (trailing slash ignored).

***

### joinPage()

```ts
function joinPage(pathname?): "approve" | "callback" | null;
```

Which Join a library page this document is, if any.

#### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `pathname` | `string` | `window.location.pathname` | Document pathname. |

#### Returns

`"approve"` \| `"callback"` \| `null`

`approve`, `callback`, or `null`.

***

### normalizeAppView()

```ts
function normalizeAppView(v): AppView;
```

Coerces an unknown string into a valid [AppView](api.md#appview).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `v` | `string` \| `undefined` | Candidate view id from the session or URL. |

#### Returns

[`AppView`](api.md#appview)

Valid view, defaulting to `discover`.

***

### normalizePathname()

```ts
function normalizePathname(pathname): string;
```

Strips a trailing slash (except for `/`).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `pathname` | `string` | Raw document pathname. |

#### Returns

`string`

Normalized pathname.

***

### pathForView()

```ts
function pathForView(view): string;
```

Returns the canonical document path for an app view.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `view` | [`AppView`](api.md#appview) | SPA view id. |

#### Returns

`string`

Path such as `/library`.

***

### queueDiscoverSearch()

```ts
function queueDiscoverSearch(handoff): void;
```

Queues a Discover catalog search for the next Discover page mount (cross-view).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `handoff` | `string` \| [`DiscoverSearchHandoff`](#discoversearchhandoff) | Query string or full handoff object (ignored when `q` is under 2 chars). |

#### Returns

`void`

***

### resolveView()

```ts
function resolveView(pathname, defaultView): AppView;
```

Resolves the view to show for the current URL plus signed-in default.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `pathname` | `string` | Document pathname. |
| `defaultView` | `string` \| `undefined` | Session default view when the path is `/`. |

#### Returns

[`AppView`](api.md#appview)

Effective [AppView](api.md#appview).

***

### syncUrlToView()

```ts
function syncUrlToView(view, mode?): void;
```

Updates the URL bar without a full reload.

Drops `ticket` and `sso_error` so a claimed invite URL is not kept on
`/discover?ticket=…` (a refresh of that path would skip the 410 invite page).

#### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `view` | [`AppView`](api.md#appview) | `undefined` | View to reflect in the path. |
| `mode` | `"replace"` \| `"push"` | `"push"` | History `push` (default) or `replace`. |

#### Returns

`void`

***

### takeQueuedDiscoverSearch()

```ts
function takeQueuedDiscoverSearch(): DiscoverSearchHandoff | null;
```

Consumes a queued Discover search (once).

#### Returns

[`DiscoverSearchHandoff`](#discoversearchhandoff) \| `null`

Handoff payload, or `null` when none / invalid.

***

### viewFromPath()

```ts
function viewFromPath(pathname): AppView | null;
```

Maps a document path to an app view, or `null` for `/` (use default_view).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `pathname` | `string` | Document pathname. |

#### Returns

[`AppView`](api.md#appview) \| `null`

Matching [AppView](api.md#appview), or `null` when the path is `/` or unknown.
