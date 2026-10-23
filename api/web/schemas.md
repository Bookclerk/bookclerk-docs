# Schemas

The request and response bodies of the [web API](README.md), in alphabetical order.

## AcquireRequestBody

Optional body for `POST /api/library/acquire`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asin` | string or null | no | Audible ASIN filter; first of uuid/asin/isbn/product_id wins. |
| `uuid` | string or null | no | Library title UUID filter. |
| `isbn` | string or null | no | ISBN filter for non-Audible stores. |
| `product_id` | string or null | no | Store product id filter. |
| `account` | string or null | no | Restrict acquire to this store account. |
| `plugin_instance_id` | string or null | no | When set, fetch through this deployed plugin instance. |
| `titles` | array of string | no | More title ids; each (with the single filter above) becomes its own job. |
| `force` | boolean | no | Re-download titles already acquired. |
| `pdf_only` | boolean | no | Fetch only each title's companion PDF. |

## AcquireStatus

Acquire / download state for a title.

Type: one of `not_acquired`, `queued`, `downloading`, `acquired`, `error`

## ActionResponse

Generic mutation result with success flag, message, and optional job id.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ok` | boolean | yes | True when the action was accepted or completed. |
| `message` | string | yes | Operator-facing status text. |
| `job_id` | string | yes | Queued job id, or empty when the action is synchronous. |
| `error` | string or null | no | Machine slug when admission failed (`conflict`, `queue_full`). |

## ApiError

An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `error` | string | yes | Stable machine slug. |
| `message` | string | no | Explanation for people. |
| `status` | integer | no | The HTTP status, repeated. |

## ApproveBody

`POST /api/config/cluster-consents/approve` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | Owner cluster id. |
| `plugin_key` | string | yes | Canonical plugin key. |
| `request_digest` | string | yes | The `request_digest` of the request the operator reviewed, as the listing shows it (refused when the recorded request is another one). |
| `deny_domains` | array of string | no | Domains to take out of the request. No duplicates. |
| `deny_bindings` | array of string | no | Bindings to take out of the request. No duplicates. |
| `deny_network` | boolean | no | Deny all network access. |

## BinaryBody

A request body of raw bytes (a file upload), documented as
`type: string, format: binary` so generated clients send the bytes
rather than a JSON array.

Type: binary

## BookRecord

Book / library item (one ownership row per store product per account).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer (int64) | yes | Surrogate primary key assigned by the database. |
| `uuid` | string | yes | Stable UUID for this row (API / foreign-key identity). |
| `plugin_key` | string | yes | Provenance-qualified PluginKey that owns the account. |
| `source` | string | yes | Display alias of the owning plugin (`audible`, `libro`, …). |
| `account_id` | string | yes | Store or operator account id this row belongs to. |
| `product_id` | string | yes | Storefront product id (ASIN, ISBN, UUID, …). |
| `asin` | string or null | no | Audible ASIN when known (`None` for Libro-only rows without enrichment). |
| `isbn` | string or null | no | ISBN-13 when known (Libro always; Audible when the API provides it). |
| `marketplace` | string | yes | Store marketplace / locale code (for example `us`, `uk`). |
| `title` | string | yes | Display title of the work or edition. |
| `authors` | string or null | no | Comma-separated or JSON author list from the storefront. |
| `narrators` | string or null | no | Comma-separated or JSON narrator list when present. |
| `series` | string or null | no | Series name when the title belongs to a series. |
| `series_index` | string or null | no | Position within the series (storefront string form). |
| `series_asin` | string or null | no | Audible series / podcast-parent ASIN (`Series.AudibleSeriesId`). |
| `acquire_status` | [`AcquireStatus`](#acquirestatus) | yes | Download/acquire pipeline state (`not_acquired`, `queued`, …). |
| `storage_key` | string or null | no | Object-storage key for the primary audio artifact, if acquired. |
| `error_message` | string or null | no | Last acquire/convert failure message for operators. |
| `purchased_at` | string (date-time) or null (date-time) | no | RFC 3339 purchase time from the storefront, when known. |
| `tags` | string or null | no | Operator or storefront tags (serialized string). |
| `rating_overall` | number (float) or null (float) | no | Overall user rating from the storefront, if any. |
| `rating_performance` | number (float) or null (float) | no | Narration/performance rating from the storefront, if any. |
| `rating_story` | number (float) or null (float) | no | Story rating from the storefront, if any. |
| `is_finished` | boolean | yes | Whether the listener marked the title finished (0/1 or bool). |
| `pdf_status` | [`AcquireStatus`](#acquirestatus) | yes | Companion PDF acquire state (`not_acquired`, `acquired`, …). |
| `pdf_storage_key` | string or null | no | Object-storage key for the companion PDF, if present. |
| `publisher` | string or null | no | Publisher name from metadata enrichment or the storefront. |
| `length_minutes` | integer (int64) or null (int64) | no | Runtime in whole minutes when the storefront reports it. |
| `is_abridged` | boolean | yes | Whether the edition is abridged (0/1 or bool). |
| `content_kind` | string | yes | Title kind: `book`, `episode`, `podcast`, …. |
| `categories` | string or null | no | Storefront category / genre path list (serialized). |
| `subtitle` | string or null | no | Optional subtitle from bibliographic metadata. |
| `published_at` | string (date-time) or null (date-time) | no | Publication date string from the storefront or enrichment. |
| `description` | string or null | no | Blurb / synopsis text (may contain HTML). |
| `language` | string or null | no | BCP-47 or storefront language code when known. |
| `cover_url` | string or null | no | HTTPS URL for cover art when known. |
| `subjects` | string or null | no | Subject / topic tags (often from Open Library; `;`- or `,`-separated). |
| `enrich_source` | string or null | no | Plugin or catalog that last enriched bibliographic fields. |
| `enrich_confidence` | number (double) or null (double) | no | 0–1 confidence score for the last enrichment pass. |
| `enrich_updated_at` | string (date-time) or null (date-time) | no | RFC 3339 time of the last enrichment write. |
| `created_at` | string (date-time) | yes | RFC 3339 timestamp when the row was inserted. |
| `updated_at` | string (date-time) | yes | RFC 3339 timestamp when the row was last modified. |

## BooksResponse

Paginated library listing.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `books` | array of [`BookRecord`](#bookrecord) | yes | Current page of title records. |
| `total` | integer | yes | Match count before pagination. At least 0. |
| `limit` | integer | yes | Applied page size. At least 0. |
| `offset` | integer | yes | Applied skip count. At least 0. |

## BootstrapRequest

Operator-only body for creating the first Owner when none exist.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `display_name` | string or null | no | Optional display name; falls back to login or email. |
| `login_name` | string or null | no | Optional local login name. |
| `email` | string or null | no | Optional email. |
| `password` | string or null | no | Optional password; hashed before persist and never stored in plaintext. |

## BranchAuthorizeParams

The authorization request a consent is for (the query `/oidc/authorize`
forwarded to `/join/approve`).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `client_id` | string | yes | Must be [`BRANCH_CLIENT_ID`]. |
| `redirect_uri` | string | yes | Where the code goes: loopback or https. |
| `response_type` | string | yes | Must be `code`. |
| `scope` | string or null | no | Must be [`BRANCH_SCOPE`]. |
| `state` | string or null | no | Echoed on the redirect. |
| `code_challenge` | string | yes | PKCE S256 challenge. |
| `code_challenge_method` | string or null | no | Must be `S256`. |
| `branch_name` | string or null | no | The name the branch asks to join as. |
| `clerk_offer` | string or null | no | `1` when the joining install offers to join as a clerk ([`bookclerk_node_proto::CLERK_OFFER_PARAM`]). |

## BranchConsentBody

`POST /api/auth/branch-consent` body.

Includes every field of [`BranchAuthorizeParams`](#branchauthorizeparams).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `approve` | boolean | yes | True to let the branch join; false to deny. |
| `connection` | string or null | no | `branch` (default) or `clerk`. |
| `step_up` | [`StepUp`](#stepup) or null | no | The fresh second factor a clerk needs. |

## BrandInfo

CSS colors and logo href for a storefront or integration brand.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `bg` | string | yes | Background color (`#RRGGBB`). |
| `fg` | string | yes | Foreground / text color (`#RRGGBB`). |
| `accent` | string | yes | Accent color (`#RRGGBB`). |
| `logo` | string | yes | Logo image href (often a favicon URL). |

## CallbackBody

`POST /api/config/join/callback` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `code` | string | no | The code from the library. |
| `state` | string | no | The `state` it came back with. |
| `error` | string or null | no | The library's `error` (`access_denied`), when it sent one. |
| `connection` | string or null | no | The connection type the library's user chose (`clerk`; a branch when left out). The code is bound to it, so a changed value only fails the redeem. |

## CallbackParams

Query or form_post fields from the IdP callback.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `code` | string or null | no | Authorization code to exchange; required unless `error` is set. |
| `state` | string or null | no | CSRF `state` that must match the `bookclerk_oidc_tx` cookie. |
| `error` | string or null | no | IdP-reported deny/error; redirects the browser to `/?sso_error=denied`. |
| `user` | string or null | no | Apple first-login name/email JSON (form_post only). |

## CatalogSearchHit

One autocomplete / results-page hit (possibly spanning multiple storefronts).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `work_key` | string | yes | Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author). |
| `title` | string | yes | Display title as shown on the storefront or library card. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `narrators` | string or null | no | Comma-separated narrator names when known. |
| `series` | string or null | no | Series name when the title belongs to a named series. |
| `series_index` | string or null | no | Position within the series (e.g. `1`, `1.5`) when known. |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `cover_url` | string or null | no | HTTPS URL for cover art when the catalog exposes one. |
| `store_editions` | array of [`StoreEdition`](#storeedition) | yes | Per-storefront edition ids that collapsed into this work card. |
| `sources` | array of string | yes | Storefronts that matched (deduped source ids). |
| `subtitle` | string or null | no | Optional subtitle when the catalog distinguishes it from the title. |
| `description` | string or null | no | Publisher or storefront synopsis; may be truncated for embeddings. |
| `publisher` | string or null | no | Publisher imprint as reported by the catalog. |
| `length_minutes` | integer (int64) or null (int64) | no | Audiobook runtime in whole minutes when known. |
| `published_at` | string or null | no | Publication date string from the catalog (ISO or storefront format). |
| `genres` | string or null | no | Genre labels from the catalog (comma-separated when multiple). |
| `language` | string or null | no | BCP-47 / storefront language code (`en`, `de`, …) when known. |
| `is_abridged` | boolean or null | no | When `Some(true)`, the edition is marked abridged by the storefront. |
| `rating_overall` | number (double) or null (double) | no | Aggregate listener rating on the storefront's scale when known. |
| `rating_count` | integer (int64) or null (int64) | no | Number of ratings backing [`Self::rating_overall`] when known. |
| `price_cents` | integer (int64) or null (int64) | no | Catalog list/deal price in cents when the storefront provided it. |
| `purchase_hints` | array of [`PurchaseHint`](#purchasehint) | no | Buy / open-in-store links with optional member vs list pricing. |

## CatalogSearchPage

Paged catalog-search response.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `items` | array of [`CatalogSearchHit`](#catalogsearchhit) | yes | Ordered collection of child entries for this response. |
| `page_size` | integer | yes | Maximum number of items returned in this page. At least 0. |
| `has_more` | boolean | yes | When true, another page is available via [`Self::next_cursor`]. |
| `next_cursor` | string or null | no | Opaque cursor for the next page; omit or `None` when exhausted. |
| `sort` | string | yes | Sort field applied to the merged result set. |
| `sort_dir` | string | no | Ascending or descending order for [`Self::sort`]. |

## CeremonyFinish

Browser attestation/assertion plus the challenge id from `begin`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `challenge_id` | string | yes | Server-issued challenge id (consumed once; 5-minute TTL). |
| `credential` | any JSON | yes | Browser `PublicKeyCredential` JSON for `finish_*`. |
| `name` | string or null | no | Label stored with the credential (`Passkey` when empty). |

## ClusterJobsBody

`POST /api/config/clusters/{cluster_id}/jobs` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `job_ids` | array of string | yes | Job ids on the owner cluster. |

## ClusterScanBody

`POST /api/config/clusters/{cluster_id}/scan` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `account` | string or null | no | Scan only this store account of the owner cluster. |
| `plugin_instance_id` | string or null | no | Pin every planned job to this owner plugin instance. |

## ConfigOptionValueInfo

One allowed value for a storefront config option.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Stored option value (for example `m4b`). |
| `label` | string | yes | Operator-facing value label. |

## ConnectionInfo

One linked store account on the portal Accounts page.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `account_id` | string | yes | Store account id (unique within its source). |
| `source` | string | yes | Source plugin id that owns this account. |
| `plugin_key` | string | yes | Provenance-qualified PluginKey that owns the account. |
| `label` | string or null | no | Operator-facing account label when the store provided one. |
| `connection_status` | string | yes | Connection state (`active`, revoked, …); defaults to `active` if the account row is missing. |
| `source_enabled` | boolean | yes | Whether `[sources.<id>] enabled` is currently true (new connects blocked when false). |
| `brand` | [`BrandInfo`](#brandinfo) or null | no | Storefront or integration brand when a plugin or fallback palette is available. |

## ConnectionsResponse

JSON body for `GET /connections`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `connections` | array of [`ConnectionInfo`](#connectioninfo) | yes | Linked store accounts the SPA may show or revoke. |

## ConnectionType

How a joining install connects to a library.

Type: one of `branch`, `clerk`

## ConsentBody

Form body for `POST /oidc/authorize` after the user grants or denies consent.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `client_id` | string | yes | Registered OIDC client id (must exist in the library store). |
| `redirect_uri` | string | yes | Redirect URI; must match a URI registered for the client. |
| `response_type` | string | yes | OAuth `response_type`; only `code` is accepted. |
| `scope` | string or null | no | Space-separated scopes; defaults to `openid profile` when omitted. |
| `state` | string or null | no | Opaque CSRF value echoed back on the redirect. |
| `code_challenge` | string | yes | PKCE S256 challenge (base64url SHA-256 of the verifier). |
| `code_challenge_method` | string or null | no | PKCE method; omitted is treated as `plain` and then rejected. |
| `consent` | string or null | no | `deny` aborts with `access_denied`; any other value proceeds to issue a code. |

## CreatePluginInstance

`POST /api/config/plugin-instances` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `plugin_key` | string | yes | Canonical plugin key; the plugin must have a pinned schema. |
| `display_name` | string | no | Operator-facing name. |
| `node_id` | string or null | no | Make the instance local to this node (H-2): it deploys only there and is never placed by the scheduler. |

## CreateRequestBody

Wishlist / title-request create body from Discover or the portal.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | yes | Required title text; empty values are rejected with 400. |
| `authors` | string or null | no | Optional author list as a single display string. |
| `asin` | string or null | no | Optional Audible ASIN used for work-key identity. |
| `isbn` | string or null | no | Optional ISBN used for work-key identity. |
| `notes` | string or null | no | Optional operator or member note stored on the request. |
| `store_editions` | array of [`StoreEdition`](#storeedition) | no | Known storefront editions to persist as `title_request_sources`. |
| `purchase_hints` | array of [`PurchaseHint`](#purchasehint) | no | Optional priced storefront links snapshotted at wishlist time. |
| `work_key` | string or null | no | Optional stable work key; when omitted one is derived from bib fields. |
| `cover_url` | string or null | no | Optional cover image URL snapshotted onto the request. |
| `description` | string or null | no | Optional plot/summary text persisted with the request. |
| `subtitle` | string or null | no | Optional subtitle stored on the request and source rows. |
| `narrators` | string or null | no | Optional narrator list as a single display string. |
| `series` | string or null | no | Optional series name. |
| `series_index` | string or null | no | Optional series position as a display string. |
| `publisher` | string or null | no | Optional publisher name. |
| `length_minutes` | integer (int64) or null (int64) | no | Optional runtime in minutes. |
| `published_at` | string or null | no | Optional publication date as a display string. |
| `genres` | string or null | no | Optional genre list as a single display string. |
| `language` | string or null | no | Optional content language code. |

## CreateUserRequest

Provisioner body for creating a user and optional invite ticket.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `role` | string or null | no | Role to assign when permitted (`member` when omitted). |
| `display_name` | string or null | no | Optional display name for the new user. |
| `login_name` | string or null | no | Optional local login name for the new user. |
| `email` | string or null | no | Optional email for the new user. |
| `password` | string or null | no | Optional password; hashed before persist. Empty or omitted leaves the user passwordless. |
| `mint_invite` | boolean | no | When true (default), also mint an invite/claim ticket. |

## DatabaseMigrateRequest

Body for copying library rows between database plugins.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string or null | no | Source plugin id. Defaults to the active `[database].plugin` before reload. |
| `to` | string | yes | Destination database plugin id to copy into. |
| `dry_run` | boolean | no | When true, count rows without writing. |
| `force` | boolean | no | When true, overwrite a destination that is not empty. |
| `apply` | boolean | no | Update `[database].plugin` in config.toml after a successful copy. |

## DiscoverFeed

Full Discover page payload: ordered shelves of personalized candidates.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `shelves` | array of [`DiscoverShelf`](#discovershelf) | yes | Ordered horizontal shelves that make up the Discover page. |
| `shelf_kinds` | array of [`ShelfKindInfo`](#shelfkindinfo) | no | Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on). |

## DiscoverShelf

One horizontal Discover row (series to finish, more by author, …).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Stable identifier for this item. |
| `title` | string | yes | Display title as shown on the storefront or library card. |
| `subtitle` | string or null | no | Optional subtitle when the catalog distinguishes it from the title. |
| `items` | array of [`Recommendation`](#recommendation) | yes | Ordered collection of child entries for this response. |

## DiscoveryDocument

OIDC discovery document served at `/.well-known/openid-configuration`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `issuer` | string | yes | Issuer origin used as `iss` in minted JWTs (`public_origin` or loopback). |
| `authorization_endpoint` | string | yes | Absolute URL of the authorization endpoint (`/oidc/authorize`). |
| `token_endpoint` | string | yes | Absolute URL of the token endpoint (`/oidc/token`). |
| `userinfo_endpoint` | string | yes | Absolute URL of the userinfo endpoint (`/oidc/userinfo`). |
| `revocation_endpoint` | string | yes | Absolute URL of the refresh-token revocation endpoint (`/oidc/revoke`). |
| `response_types_supported` | array of string | yes | Supported `response_type` values; this server only issues `code`. |
| `grant_types_supported` | array of string | yes | Supported token grants: `authorization_code` and `refresh_token`. |
| `code_challenge_methods_supported` | array of string | yes | PKCE methods; only S256 is accepted (`plain` is rejected). |
| `scopes_supported` | array of string | yes | Scopes advertised to clients (`openid`, `profile`, and `email`). |
| `subject_types_supported` | array of string | yes | Subject identifier types; only public (stable user id) is used. |
| `id_token_signing_alg_values_supported` | array of string | yes | ID-token signing algorithms; tokens are HS256 with a derived key. |
| `token_endpoint_auth_methods_supported` | array of string | yes | Client auth methods: public PKCE (`none`) or `client_secret_post`. |

## DomainEventInfo

Operator JSON for one `domain_events` row.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Event id. |
| `eventType` | string | yes | Event type (`book_acquired`). |
| `schemaVersion` | integer (int64) | yes | Payload schema version. |
| `dispatchState` | string | yes | `pending` or `dispatched`. |
| `accountId` | string | yes | Tenant / account id. |
| `dedupKey` | string | yes | Producer idempotency key. |
| `createdAt` | string | yes | RFC 3339 insert time. |
| `queueName` | string | yes | Queue the row belongs to (`core.events` for domain events). |
| `contentType` | string | yes | Body encoding (`json`, `text`, `bytes`). |
| `source` | string | yes | Producer plugin id; empty when the host published it. |
| `deliverAfter` | string or null | no | RFC 3339 earliest delivery time for delayed queue messages. |

## ElevateRequest

JSON body for Owner password re-auth before issuing an elevated operator cookie.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `password` | string | yes | Owner account password (re-authentication). |

## EnrollBody

`POST /api/node/v1/enroll` body. Keys are base64url without padding.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `link_id` | string | yes | Link id from the token. |
| `secret` | string | yes | Link secret from the token. |
| `sign_pk` | string | yes | Ed25519 public signing key. |
| `box_pk` | string | yes | X25519 public box key. |
| `software_version` | string | no | Host's Bookclerk version. |

## EnrollFinish

First TOTP code that confirms the pending secret.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `code` | string | yes | Six-digit code from the authenticator app. |

## ErrorJson

JSON error object (`error`, `message`, numeric `status`) for API clients.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `error` | string | yes | Machine slug (same as [`ErrorBody::code`]). |
| `message` | string | yes | Operator-facing explanation. |
| `status` | integer (int32) | yes | Numeric HTTP status (duplicates the response status for clients). At least 0. |

## EventDeliveryInfo

Operator JSON for one `event_deliveries` row.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Delivery id (`{event_id}:{plugin_id}`). |
| `eventId` | string | yes | Parent event id. |
| `pluginId` | string | yes | Subscriber plugin id. |
| `state` | string | yes | Lifecycle state. |
| `attemptCount` | integer (int64) | yes | Claim count (not incremented on resume). |
| `leaseGeneration` | integer (int64) | yes | Fence generation. |
| `invocationSequence` | integer (int64) | yes | Resume ordinal. |
| `outcome` | string or null | no | Terminal outcome when set. |
| `errorMessage` | string or null | no | Operator-facing error. |
| `runAfter` | string | yes | RFC 3339 earliest claim time. |
| `updatedAt` | string | yes | RFC 3339 last update. |
| `cancelRequested` | boolean | yes | Cooperative cancel flag. |
| `resourceClass` | string | yes | Concurrency class (`network` today). |
| `resumePending` | boolean | yes | Next claim should resume rather than increment attempt. |
| `hasCheckpoint` | boolean | yes | True when a suspend checkpoint is stored. |
| `queueName` | string | yes | Queue of the parent event. |

## EventQueueStatus

Outbox delivery counts for `GET /api/status`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pending` | integer (int64) | yes | Pending deliveries without a suspend checkpoint. |
| `running` | integer (int64) | yes | Currently claimed deliveries. |
| `suspended` | integer (int64) | yes | Pending deliveries parked with a checkpoint. |
| `dead_letter` | integer (int64) | yes | Dead-lettered deliveries. |
| `acked` | integer (int64) | yes | Successfully acked deliveries. |
| `oldest_pending_age_secs` | integer (int64) or null (int64) | no | Age in seconds of the oldest pending delivery, when any exist. |
| `retries_total` | integer (int64) | yes | Durable retry outcomes (including reclaim-as-retry). |
| `suspensions_total` | integer (int64) | yes | Durable suspend outcomes. |
| `dead_letters_total` | integer (int64) | yes | Durable transitions into `dead_letter`. |
| `dispatch_latency_ms_avg` | integer (int64) or null (int64) | no | Average first-dispatch latency in milliseconds, when sampled. |
| `handler_latency_ms_avg` | integer (int64) or null (int64) | no | Average `onEvent` handler duration in milliseconds, when sampled. |

## HealthResponse

Liveness payload for unauthenticated `GET /health`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string | yes | Always `ok` when the process can serve HTTP. |

## ImpersonateRequest

JSON body naming the first-party user to impersonate.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `user_id` | integer (int64) | yes | Target `users.id` to impersonate. |

## InspectBody

`POST /api/config/join/inspect` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `address` | string | yes | The library's address (`https://owner.example`, `owner.lan:8443`). |

## IntegrationLoginBody

JSON body for `POST /login/integration` (provider credential login).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `provider` | string | yes | Integration plugin id (must be enabled in config). |
| `username` | string | yes | Provider login name forwarded to `authenticate_user`. |
| `password` | string | yes | Provider password; never logged. |

## IntegrationScanRequest

Optional body for `POST /integrations/{id}/scan`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `force` | boolean or null | no | When true, the integration rescans even if it considers itself current. |

## InviteBody

`POST /api/config/join/invite` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `link` | string | yes | The whole branch invite (`https://<library>/enroll#bcn1...`). |

## JobInfo

Snapshot of one queued or running control-plane job.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Opaque job id returned to the client when the job is accepted. |
| `kind` | string | yes | Job type (`scan`, `acquire`, `listen_sync`). |
| `status` | string | yes | Lifecycle string (`pending`, `running`, `succeeded`, `failed`, `cancelled`). |
| `detail` | string or null | no | Optional operator-facing progress or error text. |
| `progress` | string or null | no | Human-readable progress string while the job is running. |
| `attempt_count` | integer (int64) | no | Number of times a worker has claimed this row. |
| `max_attempts` | integer (int64) | no | Maximum claims before a failure is terminal. |
| `error_kind` | string or null | no | Structured error kind when failed or cancelled. |
| `created_at` | string or null | no | RFC 3339 timestamp when the row was inserted. |
| `finished_at` | string or null | no | RFC 3339 timestamp when the row reached a terminal state. |

## LeaveBody

`POST /api/config/join/leave` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | The library (owner cluster id) to leave. |

## LoginBegin

Login-name or email used to start a discoverable-less passkey login.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `login` | string | yes | Login name or email looked up before issuing an authentication challenge. |

## LoginRequest

JSON body for `POST /api/auth/login` (operator token).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `token` | string | yes | Operator token from the client; compared in constant time and never persisted. |

## LoginVerify

TOTP code plus the challenge id from password login.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `challenge_id` | string | yes | Server-issued challenge id (5-minute TTL, single use). |
| `code` | string | yes | Six-digit authenticator code. |

## MeResponse

JSON body for `GET /me`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `provider` | string | yes | Identity provider id (`local` or an integration plugin id). |
| `external_user_id` | string | yes | Provider-scoped user id stored on the portal identity row. |
| `label` | string or null | no | Optional display label from the identity row. |

## MfaPolicyPut

PUT body for the host-wide second-factor policy.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `require_second_factor` | boolean | yes | When true, password login requires TOTP or the user must use a passkey. |
| `current_password` | string or null | no | Current password for Owner step-up (operators omit this). |

## OAuthEnrollRequest

`POST /api/node/v1/enroll/oauth` body. Keys are base64url without
padding.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `code` | string | yes | The authorization code from the callback. |
| `code_verifier` | string | yes | The PKCE verifier. |
| `redirect_uri` | string | yes | The redirect URI the authorization was requested with. |
| `sign_pk` | string | yes | Ed25519 public signing key. |
| `box_pk` | string | yes | X25519 public box key. |
| `branch_name` | string | no | The name this branch joins as (the node's label). |
| `software_version` | string | no | The joining install's Bookclerk version. |
| `connection` | [`ConnectionType`](#connectiontype) | no | The connection type the code was approved for. It must match the one bound to the code at consent; left out for a branch. |

## OidcClientWrite

Body for creating or updating a Bookclerk-as-IdP client.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `client_id` | string or null | no | Public client_id (create only; path wins on update). |
| `name` | string or null | no | Operator-facing display name. |
| `redirect_uris` | array of string | no | Allowed OAuth redirect URIs. |
| `confidential` | boolean | no | When true, mint a client secret (confidential). When false, public PKCE. |
| `issue_refresh_token` | boolean | no | When true, token responses include a refresh token. |
| `allowed_scopes` | array of string | no | Scopes this client may request (`openid`, `profile`, `email`). |
| `enabled` | boolean | no | When true, authorize and token endpoints accept this client. |
| `current_password` | string or null | no | Required for a non-elevated Owner whose portal session is older than 15 minutes. |

## OidcConfigPut

Operator PUT body that replaces the broker table (secrets omitted or replaced).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `enabled` | boolean | yes | Whether the broker should accept authorize starts after this write. |
| `allowed_email_domains` | array of string | no | Replacement broker-wide email domain allowlist. |
| `providers` | array of [`OidcProviderPut`](#oidcproviderput) | no | Full replacement list of IdPs (omitted secrets keep the previous generation). |
| `public_origin` | string or null | no | `integrations.public_origin`. Omit to leave unchanged; empty string clears. |
| `current_password` | string or null | no | Required for a non-elevated Owner whose portal session is older than 15 minutes. |

## OidcConfigResponse

Operator-facing OIDC broker snapshot; secret material is never included.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `enabled` | boolean | yes | Whether the broker accepts new authorize starts. |
| `allowed_email_domains` | array of string | yes | Broker-wide email domain allowlist applied before per-provider rules. |
| `public_origin` | string or null | no | Configured `integrations.public_origin` (issuer / callback / invite base). |
| `issuer_url` | string | yes | Effective issuer origin (`public_origin`, else this request, else localhost). |
| `detected_origin` | string | yes | Origin detected from this request (loopback IPs rewritten to `localhost`). |
| `discovery_url` | string | yes | Absolute OpenID discovery URL (`/.well-known/openid-configuration`). |
| `authorization_endpoint` | string | yes | Absolute authorization endpoint (`/oidc/authorize`) for Bookclerk-as-IdP. |
| `token_endpoint` | string | yes | Absolute token endpoint (`/oidc/token`) for Bookclerk-as-IdP. |
| `userinfo_endpoint` | string | yes | Absolute userinfo endpoint (`/oidc/userinfo`) for Bookclerk-as-IdP. |
| `revocation_endpoint` | string | yes | Absolute revocation endpoint (`/oidc/revoke`) for Bookclerk-as-IdP. |
| `callback_url` | string | yes | Absolute `/api/auth/oidc/callback` derived from the effective issuer origin. |
| `providers` | array of [`OidcProviderView`](#oidcproviderview) | yes | Configured IdPs with presence flags instead of secret material. |

## OidcProviderPut

One IdP in a PUT: omit secret fields to keep, empty string to clear.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Stable provider id; also the logical name of the sealed client-secret row. |
| `name` | string | no | Operator-facing display name stored in config.toml. |
| `preset` | string or null | no | Built-in social preset; empty or omitted means a generic OpenID issuer. |
| `issuer` | string or null | no | OpenID issuer URL required when `preset` is unset. |
| `client_id` | string | yes | OAuth client id registered at the IdP. |
| `client_secret` | string or null | no | Omit to keep; empty string to clear; non-empty to store. |
| `scopes` | array of string | no | OAuth scopes; empty is replaced with the provider default set. |
| `provision` | string | no | JIT, mapped-role, allowlist, or invite-only policy for first login. |
| `default_role` | string | no | Fallback portal role when no mapped claim matches. |
| `role_claim` | string | no | Claim name used for role mapping (defaults to `groups`). |
| `role_map` | map of string to string | no | Claim value to portal role mapping applied at provision time. |
| `link_by_email` | boolean | no | When true, verified email may attach this IdP to an existing user. |
| `allowed_email_domains` | array of string | no | Per-provider email domain allowlist. |
| `allowed_emails` | array of string | no | Exact emails permitted under allowlist provision. |
| `allowed_subjects` | array of string | no | Exact IdP subject ids permitted under allowlist provision. |
| `apple_team_id` | string or null | no | Apple Developer team id for client-secret JWT minting. |
| `apple_key_id` | string or null | no | Apple Sign In key id paired with the sealed private key. |
| `apple_private_key` | string or null | no | Omit to keep; empty string to clear; non-empty to store. |

## OidcProviderView

One IdP as shown to an operator (no client secret or Apple PEM).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Stable provider id used in authorize URLs and sealed-row names. |
| `name` | string | yes | Operator-facing display name. |
| `preset` | string or null | no | Built-in social preset (`google`, `github`, `apple`, `discord`) when not a generic issuer. |
| `issuer` | string or null | no | OpenID issuer URL for discovery when no social preset applies. |
| `client_id` | string | yes | OAuth client id registered at the IdP. |
| `scopes` | array of string | yes | OAuth scopes requested at authorize. |
| `provision` | string | yes | How a first login becomes a portal user (JIT, mapped role, allowlist, invite-only). |
| `default_role` | string | yes | Portal role used when no mapped claim matches (`member` by default). |
| `role_claim` | string | yes | Claim name read from id_token/userinfo for role mapping (`groups` by default). |
| `role_map` | map of string to string | yes | IdP group/role claim values mapped onto Bookclerk portal roles. |
| `link_by_email` | boolean | yes | When true, a verified email may attach this IdP to an existing user. |
| `allowed_email_domains` | array of string | yes | Per-provider email domain allowlist (in addition to the broker-wide list). |
| `allowed_emails` | array of string | yes | Exact emails permitted under allowlist provision. |
| `allowed_subjects` | array of string | yes | Exact IdP subject ids permitted under allowlist provision. |
| `apple_team_id` | string or null | no | Apple Developer team id used to mint the client-secret JWT. |
| `apple_key_id` | string or null | no | Apple Sign In key id paired with the sealed private key. |
| `has_client_secret` | boolean | yes | Whether a client secret exists in env, TOML, or the sealed store (value never returned). |
| `has_apple_private_key` | boolean | yes | Whether an Apple PEM exists in env, TOML, or the sealed store (PEM never returned). |
| `secret_source` | string | yes | Where the client secret was found: `env`, `config`, `store`, or `none`. |

## PasskeyReauthBody

Optional current password for step-up before register or delete.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `current_password` | string or null | no | Password used for recent-reauth when the user already has a credential. |

## PasswordLoginBody

JSON body for password-store connect (`email` + `password`).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | string | yes | Store account email / username. |
| `password` | string | yes | Store account password; never logged. |

## PasswordLoginRequest

JSON body for local password login (login name or email plus password).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `login` | string | yes | Login name or email. |
| `password` | string | yes | Password verified against the stored hash; mismatch fails closed and counts toward lockout. |

## PatchPreferencesBody

Partial preferences update; omitted fields keep their stored values.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `default_view` | string or null | no | Replacement landing view when set. |
| `disabled_shelves` | array of string or null | no | Replacement hidden-shelf list when set. |
| `discover_sort` | string or null | no | Replacement Discover sort key when set. |
| `discover_sort_dir` | string or null | no | Replacement Discover sort direction when set. |
| `discover_language` | string or null | no | Present + value sets language; present + `null` clears to browser default; omitted leaves unchanged. |
| `discover_excluded_sources` | array of string or null | no | Replacement excluded-store list when set. |
| `theme` | string or null | no | Replacement appearance preference when set (`system`, `light`, `dark`). |

## PatchProfileRequest

JSON body for `PATCH /api/auth/profile`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `display_name` | string or null | no | Replacement display name; empty or whitespace clears the field. |
| `email` | string or null | no | Replacement email; empty or whitespace clears the field. |
| `avatar_source` | string or null | no | Picture choice (`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`). |

## PatchSettingsRequest

Body for `PATCH /api/settings`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `settings` | array of [`SettingsUpdate`](#settingsupdate) | yes | Key/value pairs to validate, persist, and reload. |
| `expected_output_revision` | integer (int64) or null (int64) | no | `core.output` document revision the client read (`output_revision` in `GET /api/settings`). When set and stale, an `output.*` update is rejected with 409. |

## PatchUserRequest

Partial profile, role, or status patch for a first-party user.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `role` | string or null | no | Replacement role (`owner` / `administrator` / `member`) when present. |
| `status` | string or null | no | Replacement status (`active` / `disabled`) when present. |
| `display_name` | string or null | no | Replacement display name when present. |
| `login_name` | string or null | no | Replacement local login name when present. |
| `email` | string or null | no | Replacement email when present. |

## PlacementView

Scheduler placement of one instance: the operator's intent and the last
pass's result.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `replicas` | integer or `all` | yes | A replica count (0 stops every replica) or `"all"`. |
| `constraints` | array of string | yes | Label constraints every chosen node must satisfy. |
| `revision` | integer (int64) | yes | Compare-and-swap revision of the intent. |
| `placed` | integer (int32) | yes | Replicas the last scheduler pass placed. At least 0. |
| `detail` | string | yes | Why fewer than requested are placed; empty when satisfied. |
| `scheduled_at` | string | yes | RFC 3339 time the scheduler last changed the status. |
| `updated_at` | string | yes | RFC 3339 time of the last intent write. |

## PluginConsentApproveRequest

Body for `POST /api/plugins/{id}/consent`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `approve` | boolean | yes | Must be true; a false value is rejected with 400. |
| `grant` | [`PluginGrantOverride`](#plugingrantoverride) or null | no | Optional operator overrides applied on top of the manifest request. |

## PluginConsentBrand

Optional branding shown on the plugin consent dialog.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes | Display name from the portal brand or plugin manifest. |
| `bg` | string or null | no | Background CSS color from the portal brand, when known. |
| `fg` | string or null | no | Foreground CSS color from the portal brand, when known. |
| `accent` | string or null | no | Accent CSS color from the portal brand, when known. |
| `logo` | string or null | no | Logo URL or host-served embed path. |

## PluginConsentLimits

Host-capped resource defaults shown on the consent dialog.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cpu_ms` | integer (int32) | yes | Requested workerd CPU budget in milliseconds. At least 0. |
| `subrequests` | integer (int32) | yes | Requested workerd subrequest budget. At least 0. |
| `max_cpu_ms` | integer (int32) | yes | Host hard cap for workerd CPU milliseconds. At least 0. |
| `max_subrequests` | integer (int32) | yes | Host hard cap for workerd subrequests. At least 0. |
| `disk_mib` | integer (int32) | yes | Default plugin state disk budget in mebibytes. At least 0. |
| `max_disk_mib` | integer (int32) | yes | Host hard cap for plugin state disk in mebibytes. At least 0. |
| `memory_mib` | integer (int32) | yes | Default jail memory in mebibytes. At least 0. |
| `max_memory_mib` | integer (int32) | yes | Host hard cap for jail memory in mebibytes. At least 0. |
| `cpu_cores` | number (double) | yes | Manifest / default jail CPU in cores (2 d.p.). |
| `max_cpu_cores` | number (double) | yes | Host max jail CPU in cores (2 d.p.). |
| `jail_cpu_cores` | number (double) or null (double) | no | Optional `[plugins.jail]` ceiling in cores. |
| `extra_processes` | integer (int32) | yes | Default extra process/thread budget (native consent). At least 0. |
| `max_extra_processes` | integer (int32) | yes | Host hard cap for extra process budget. At least 0. |
| `known_bindings` | array of string | yes | Suggested / default host bindings operators may grant. |

## PluginConsentResponse

Consent dialog payload for one plugin.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `plugin_id` | string | yes | Plugin id whose grant is being reviewed. |
| `plugin_key` | string | yes | Canonical PluginKey for this install (empty when unknown). |
| `provenance` | string | yes | Host-evaluated provenance label. |
| `runtime` | string | yes | Guest runtime (`native` or `workerd`) — drives which controls are enforceable. |
| `request` | [`PluginGrantView`](#plugingrantview) | yes | Manifest-derived grant the plugin is asking for. |
| `covered` | boolean | yes | True when an existing on-disk grant already satisfies the request. |
| `summary` | array of string | yes | Operator-facing bullet list of requested capabilities. |
| `existing` | [`PluginGrantView`](#plugingrantview) or null | no | Previously approved grant, when one is stored. |
| `brand` | [`PluginConsentBrand`](#pluginconsentbrand) or null | no | Optional portal or manifest branding for the dialog. |
| `limits` | [`PluginConsentLimits`](#pluginconsentlimits) | yes | Host-capped resource defaults (workerd budgets + shared disk). Always set. |

## PluginDeploymentView

This node's view of one deployment row, with its latest observation.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `deployment_id` | string | yes | Deployment row id. |
| `node_id` | string | yes | Node the row targets. |
| `desired` | string | yes | `present` or `absent`. |
| `revision` | integer (int64) | yes | Compare-and-swap revision of the desired row. |
| `status` | string or null | no | Latest observation status from that node, when it reported one. |
| `detail` | string or null | no | Observation detail (error text, redacted). |
| `applied_config_revision` | integer (int64) or null (int64) | no | Document revision the running guest applied. |
| `observed_at` | string or null | no | RFC 3339 time of the observation. |

## PluginGrantOverride

Operator-supplied grant fields that replace matching request values.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `networkMode` | string or null | no | Override egress mode when set. |
| `domains` | array of string or null | no | Override approved hostnames when set. |
| `tcp` | array of [`TcpGrantView`](#tcpgrantview) or null | no | Override raw TCP grants when set. |
| `addressCidrs` | array of string or null | no | Override address-space CIDRs when set. |
| `allowUndeclaredPublicRedirects` | boolean or null | no | Override undeclared-public-redirect permission when set. |
| `bindings` | array of string or null | no | Override approved host bindings when set. |
| `compatibilityFlags` | array of string or null | no | Override compatibility flags when set. |
| `cpuMs` | integer (int32) or null (int32) | no | Override workerd CPU milliseconds when set. At least 0. |
| `subrequests` | integer (int32) or null (int32) | no | Override workerd subrequest budget when set. At least 0. |
| `diskMib` | integer (int32) or null (int32) | no | Override disk budget in mebibytes when set. At least 0. |
| `memoryMib` | integer (int32) or null (int32) | no | Override jail memory in mebibytes when set. At least 0. |
| `cpuCores` | number (double) or null (double) | no | Jail CPU in cores (2 d.p.); converted to Spec percent server-side. |
| `extraProcesses` | integer (int32) or null (int32) | no | Extra process/thread budget (native); ignored for workerd. At least 0. |

## PluginGrantView

Operator-facing grant JSON: CPU as cores (not Spec percent).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pluginKey` | string | yes | Canonical provenance-qualified PluginKey. |
| `pluginId` | string | yes | Plugin id this grant applies to. |
| `entrypoints` | array of string | yes | Exported entrypoints the operator approved (`storefront`, `cli`, …). |
| `producers` | array of string | yes | Event types the operator approved the plugin to publish. |
| `consumers` | array of object | yes | Approved event-consumer subscriptions. |
| `jobs` | array of string | yes | Approved job trigger types. |
| `networkMode` | string | yes | Approved egress mode (`deny`, `allowlist`, and similar). |
| `domains` | array of string | yes | Hostnames the operator approved for egress. |
| `manifestDomains` | array of string | yes | Destinations requested by the current manifest. |
| `operatorAddedDomains` | array of string | yes | Operator-added destinations beyond the manifest. |
| `operatorDeniedDomains` | array of string | yes | Operator-denied destinations still listed by the manifest. |
| `tcp` | array of [`TcpGrantView`](#tcpgrantview) | yes | Effective raw TCP grants. |
| `operatorAddedTcp` | array of [`TcpGrantView`](#tcpgrantview) | yes | Operator-added TCP grants. |
| `operatorDeniedTcp` | array of [`TcpGrantView`](#tcpgrantview) | yes | Operator-denied TCP grants still listed by the manifest. |
| `addressCidrs` | array of string | yes | Effective address-space CIDRs. |
| `operatorAddedCidrs` | array of string | yes | Operator-added CIDRs. |
| `operatorDeniedCidrs` | array of string | yes | Operator-denied CIDRs still listed by the manifest. |
| `allowUndeclaredPublicRedirects` | boolean | yes | Undeclared public redirect permission. |
| `grantRevision` | string | yes | SHA-256 of persisted operator consent (no host overlays). |
| `authorityRevision` | string | yes | SHA-256 of the canonical effective authority. |
| `bindings` | array of string | yes | Host bindings the operator approved. |
| `compatibilityFlags` | array of string | yes | Extra compatibility flags stored on the grant. |
| `approvedAt` | string | yes | RFC 3339 timestamp when the grant was last approved. |
| `cpuMs` | integer (int32) or null (int32) | no | Optional workerd CPU budget in milliseconds. At least 0. |
| `subrequests` | integer (int32) or null (int32) | no | Optional workerd subrequest budget. At least 0. |
| `diskMib` | integer (int32) or null (int32) | no | Optional plugin state disk budget in mebibytes. At least 0. |
| `memoryMib` | integer (int32) or null (int32) | no | Optional jail memory in mebibytes. At least 0. |
| `cpuCores` | number (double) or null (double) | no | Jail CPU in cores (2 d.p.); absent for workerd. |
| `extraProcesses` | integer (int32) or null (int32) | no | Extra process/thread budget beyond launcher overhead; native only. At least 0. |

## PluginSettingChoice

One selectable value for a Settings dropdown.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `value` | string | yes | Stored config value; empty string means use the plugin default. |
| `label` | string | yes | Operator-facing choice caption. |

## PluginSettingOption

One editable setting shown in the Settings UI.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `key` | string | yes | Dotted `config.toml` key for this option. |
| `label` | string | yes | Operator-facing field caption. |
| `value` | string | yes | Current string form of the setting. |
| `value_type` | string | yes | UI widget hint (`boolean`, `string`, or `number`). |
| `choices` | array of [`PluginSettingChoice`](#pluginsettingchoice) or null | no | Optional dropdown values; omitted for free-form fields. |
| `secret` | boolean | no | True for a `secret` setting: `value` is never the stored value. |
| `secret_set` | boolean or null | no | For a secret setting, whether the instance document names a sealed value. |

## PluginSettingsGroup

Settings UI group for one discovered or loaded plugin.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Plugin id (`audible`, `sqlite`, and similar). |
| `family` | string | yes | Handler family wire label (`source`, `integration`, `output`, `database`). |
| `plugin_key` | string or null | no | Provenance-qualified PluginKey when this row came from a discovered install. |
| `provenance` | string or null | no | Host-evaluated provenance (`platform_bundled`, `verified_staged`, `installed`, …). |
| `logo` | string or null | no | Google favicon (or portal brand) URL for Settings list rows. |
| `settings` | array of [`PluginSettingOption`](#pluginsettingoption) | yes | Editable options for this plugin, including `.enabled`. |
| `plugin_instance_id` | string or null | no | Plugin instance this group edits; absent for a bootstrap database adapter. |
| `display_name` | string or null | no | Operator-facing instance name, when the operator set one. |
| `is_default` | boolean | no | True when a bare plugin key or alias routes to this instance. |
| `config_revision` | integer (int64) or null (int64) | no | Instance document revision for compare-and-swap writes. |
| `deployment` | [`PluginDeploymentView`](#plugindeploymentview) or null | no | This node's deployment of the instance, when one exists. |
| `placement` | [`PlacementView`](#placementview) or null | no | Scheduler placement; absent when the instance is hand-managed (the Enabled switch applies only then). |
| `present_on` | array of string | no | Nodes whose deployment of the instance is `present`. |

## PortalErrorBody

JSON body of every portal error response.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `error` | string | yes | Operator-facing error text (no structured code). |

## PostNodeInvite

`POST /api/config/node-invites` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `user_id` | integer (int64) or null (int64) | no | User who will own the node. Defaults to the caller; the operator must name one. |
| `ttl_secs` | integer (int64) or null (int64) | no | Link lifetime in seconds (default 24 h, 5 min to 7 days). At least 0. |
| `label` | string | no | Label shown in listings. |

## PostPluginInstall

`POST /api/config/plugins` body: exactly one of `coordinate` or
`package` (a directory name under `plugin-packages/`).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `coordinate` | string or null | no | Registry coordinate (`cargo:…@v`, `registry:…#name@v`). |
| `package` | string or null | no | Operator-placed package directory under `plugin-packages/`. |
| `activate` | boolean | no | Make the new version active even when another is. |
| `approve` | boolean | no | Record the requested consent as approved. |
| `allow_unverified_publisher` | boolean | no | Accept a publisher without independent authenticity. |

## PreferencesResponse

Normalized Discover/library preferences for the authenticated subject.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `default_view` | string | yes | Landing view id (`library`, `discover`, …) after normalization. |
| `disabled_shelves` | array of string | yes | Lowercased Discover shelf ids the user has hidden. |
| `discover_sort` | string | yes | Discover sort key (`relevance`, `popularity`, …). |
| `discover_sort_dir` | string | yes | Discover sort direction (`asc` or `desc`). |
| `discover_language` | string or null | no | `null` = use browser language on the client. |
| `discover_excluded_sources` | array of string | yes | Lowercased store ids hidden from Discover results. |
| `theme` | string | yes | Appearance preference (`system`, `light`, or `dark`). |

## PurchaseHint

A purchase / catalog availability hint (optionally priced at view time).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `source` | string | yes | Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …). |
| `product_id` | string | yes | Storefront-native product id (ASIN, ISBN, UUID, …). |
| `title` | string or null | no | Display title as shown on the storefront or library card. |
| `url` | string or null | no | Absolute HTTPS URL. |
| `price_cents` | integer (int64) or null (int64) | no | Primary / best known sell price in minor units (prefer member when dual). `0` = free. |
| `currency` | string or null | no | ISO 4217 currency code for price fields (`USD`, `EUR`, …). |
| `price_label` | string or null | no | Display string from the store (`$2.99`, `FREE`, …). |
| `list_price_cents` | integer (int64) or null (int64) | no | Non-member / list price in integer cents when known. |
| `list_price_label` | string or null | no | Human-readable list price from the storefront. |
| `member_price_cents` | integer (int64) or null (int64) | no | Member / credit price in integer cents when known. |
| `member_price_label` | string or null | no | Human-readable member price from the storefront. |

## PurchaseHintsBatchBody

Body for batched purchase-hint lookups (max 24 queries).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `queries` | array of [`PurchaseHintsQuery`](#purchasehintsquery) | no | Per-title purchase-hint queries in this batch. |

## PurchaseHintsBatchResponse

Parallel purchase-hint results aligned with the request queries.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results` | array of [`PurchaseHintsResponse`](#purchasehintsresponse) | yes | One purchase-hint response per query, in request order. |

## PurchaseHintsQuery

Inputs for view-time catalog + pricing lookup.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | yes | Display title as shown on the storefront or library card. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `candidate_source` | string or null | no | Storefront id of the edition that produced this card. |
| `candidate_product_id` | string or null | no | Storefront product id of the edition that produced this card. |
| `store_editions` | array of [`StoreEdition`](#storeedition) | no | Known storefront editions already on the recommendation card. |
| `region` | string or null | no | Marketplace / region code (`us`, `uk`, …) for catalog lookups. |
| `preferred_sources` | array of string | no | Storefronts the caller has linked accounts for. Treated as **member** pricing when picking `best`; other stores are compared at list / non-member price. Every catalog match is still returned in `hints`. |

## PurchaseHintsResponse

Priced catalog matches for one title, sorted best-first.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `hints` | array of [`PurchaseHint`](#purchasehint) | yes | Resolved purchase hints for this query. |
| `best` | [`PurchaseHint`](#purchasehint) or null | no | Lowest-priced hint (or first catalog hit when no prices resolved). |

## PutActiveVersion

`PUT /api/config/plugins/{key}/active` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `version` | string | yes | Installed version to activate. |

## PutBranches

`PUT /api/config/branches` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `enrollment` | boolean | yes | "Let new branches join" (`core.host_enrollment.enabled`). |

## PutEventsDomainRequest

JSON body for `PUT /api/config/domains/core.events`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `expected_revision` | integer (int64) | yes | Revision the caller read. A mismatch is a conflict. |
| `retention_days` | integer (int64) | yes | Replacement `[events]` document. At least 0. |
| `dead_letter_retention_days` | integer (int64) | yes | Replacement dead-letter retention. At least 0. |
| `concurrency` | integer (int32) | yes | Replacement concurrency cap. At least 0. |

## PutHostEnrollment

`PUT /api/config/host-enrollment` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `enabled` | boolean | yes | Whether host nodes may enroll into this cluster. |

## PutNodeLabels

`PUT /api/config/nodes/{node_id}/labels` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `labels` | array of string | yes | The node's complete operator label set. |

## PutNodeOwner

`PUT /api/config/nodes/{node_id}/owner` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `user_id` | integer (int64) or null (int64) | no | User who owns the node, or `null` to clear. |

## PutNodeRuntime

`PUT /api/config/nodes/{node_id}/runtime` body: any subset of the
node's runtime document.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `label` | string or null | no | Operator label. |
| `cpu_budget_percent` | integer (int32) or null (int32) | no | Share of detected CPU the scheduler may commit (1..=100). At least 0. |
| `memory_budget_percent` | integer (int32) or null (int32) | no | Share of detected memory the scheduler may commit (1..=100). At least 0. |
| `schedulable` | boolean or null | no | False drains the node. |

## PutOutputDomainRequest

JSON body for `PUT /api/config/domains/core.output`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `expected_revision` | integer (int64) | yes | Revision the caller read. A mismatch is a conflict. |
| `document` | object | yes | Complete replacement document (schema version 1). |

## PutPluginDeployment

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `desired` | string | yes | `present` or `absent`. |
| `expected_revision` | integer (int64) or null (int64) | no | Revision the caller last observed; required when the row exists and the caller wants compare-and-swap. Omitted means "replace whatever is current". |

## PutPluginInstanceConfig

`PUT /api/config/plugin-instances/{id}/config` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `expected_revision` | integer (int64) | yes | Revision the caller last observed. |
| `settings` | object | no | Scalar settings. Secret values are not accepted here. |
| `secret_refs` | array of object | no | Secret refs to keep. Omitted refs are dropped from the document; the ciphertext stays sealed until the secret is overwritten. |
| `secrets` | map of string to string | no | Secret values to seal, keyed by the declared `secret` setting. Each is sealed into `encrypted_secrets` and referenced from the document; the plaintext is never stored in the document or echoed back. |

## PutPluginPlacement

`PUT /api/config/plugin-instances/{id}/placement` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `replicas` | integer or `all` | yes | A replica count (0 stops every replica) or `"all"`. |
| `constraints` | array of string | no | Label constraints (`name`, `name=value`, `!name`). |
| `expected_revision` | integer (int64) or null (int64) | no | Current revision to replace; omit (or 0) to create. |

## QueueDetail

Operator JSON for `GET /api/queues/{name}`.

Includes every field of [`QueueInfo`](#queueinfo).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `messages` | array of [`DomainEventInfo`](#domaineventinfo) | yes | Recent messages on the queue, newest first. |
| `deliveries` | array of [`EventDeliveryInfo`](#eventdeliveryinfo) | yes | Recent deliveries on the queue, newest first. |

## QueueInfo

Operator JSON for one queue on `GET /api/queues`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes | Queue name. |
| `core` | boolean | yes | True for the domain-event bus (`core.events`). |
| `pending` | integer (int64) | yes | Deliveries waiting to be claimed (including suspended rows). |
| `running` | integer (int64) | yes | Deliveries currently claimed by a worker. |
| `acked` | integer (int64) | yes | Deliveries acked. |
| `deadLetter` | integer (int64) | yes | Deliveries parked for operator review. |
| `consumers` | array of string | yes | Live catalog consumers (plugin keys or instance ids) of this queue. |

## QueueWisher

Compact wisher shown on the global queue (avatar + label).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `user_id` | integer (int64) or null (int64) | no | First-party `users.id` when the portal identity is linked. |
| `identity_id` | integer (int64) or null (int64) | no | Portal identity that created the wish (dedupe for unlinked identities). |
| `display_name` | string or null | no | Display name, identity label, or `"Operator"`. |
| `login_name` | string or null | no | Local login name when the first-party user has one. |
| `operator` | boolean | no | True when the wish came from the operator token (no portal user). |
| `has_avatar` | boolean | no | True when an uploaded avatar file exists (filled by the daemon). |
| `avatar_source` | string or null | no | Explicit picture choice when the user has set one. |
| `gravatar_hash` | string or null | no | SHA-256 hex of the contact email for Gravatar. |
| `picture_url` | string or null | no | HTTPS picture from the wishing identity, when any. |

## RankedQueueEntry

Global wishlist queue entry ranked by overall/operator recommend taste +
heavy wish-count weight (shared order; not per-viewer personalized).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `work_key` | string | yes | Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author). |
| `title` | string | yes | Display title as shown on the storefront or library card. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `cover_url` | string or null | no | HTTPS URL for cover art when the catalog exposes one. |
| `wish_count` | integer (int64) | yes | Number of distinct users who wishlisted this title. |
| `wishers` | array of [`QueueWisher`](#queuewisher) | no | Distinct people who have this title on an open wishlist. |
| `sample_uuids` | array of string | yes | Sample Bookclerk user UUIDs who requested this title (privacy-capped). |
| `first_requested_at` | string (date-time) | yes | UTC timestamp of the earliest wishlist request for this work. |
| `last_requested_at` | string (date-time) | yes | UTC timestamp of the latest wishlist request for this work. |
| `score` | number (double) | yes | Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`]). |
| `taste_score` | number (double) | yes | Local taste / embedding component before the wish-count boost. |
| `reasons` | array of string | yes | Short human-readable explanations for why this item was ranked here. |

## Recommendation

One ranked recommendation (typically an unowned storefront title).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `work_id` | string or null | no | Library `works.id` UUID when the recommendation is tied to an owned work. |
| `title` | string | yes | Display title as shown on the storefront or library card. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `narrators` | string or null | no | Comma-separated narrator names when known. |
| `series` | string or null | no | Series name when the title belongs to a named series. |
| `series_index` | string or null | no | Position within the series (e.g. `1`, `1.5`) when known. |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `score` | number (double) | yes | Ranking score (higher is better); units are algorithm-specific. |
| `reasons` | array of string | yes | Short human-readable explanations for why this item was ranked here. |
| `purchase_hints` | array of [`PurchaseHint`](#purchasehint) | yes | Buy / open-in-store links with optional member vs list pricing. |
| `from_request` | boolean | yes | True when sourced from an open title request rather than storefront discovery. |
| `request_uuid` | string or null | no | Wishlist / title-request UUID when this card came from a request. |
| `candidate_source` | string or null | no | Storefront that proposed this title (`audible`, `libro`, …). |
| `candidate_product_id` | string or null | no | Storefront product id of the edition that produced this card. |
| `store_editions` | array of [`StoreEdition`](#storeedition) | no | All known storefront editions (for multi-store purchase links). |
| `seed_categories` | string or null | no | Categories/subjects copied from the taste seed that produced this hit. |
| `categories` | array of string | no | Stable shelf-kind tags for prefs (`finish_series`, `author`, `chirp_deals`, …). Dynamic shelf titles stay in `build_discover_feed`; membership uses these. |
| `work_key` | string | no | Stable bibliographic key for wishlist / merge (`isbn:` / `asin:` / `soft:`…). |
| `cover_url` | string or null | no | Cover art URL when known from a storefront catalog hit or wishlist row. |
| `subtitle` | string or null | no | Optional bibliographic extras from storefront catalog / Audnexus. |
| `description` | string or null | no | Publisher or storefront synopsis; may be truncated for embeddings. |
| `publisher` | string or null | no | Publisher imprint as reported by the catalog. |
| `length_minutes` | integer (int64) or null (int64) | no | Audiobook runtime in whole minutes when known. |
| `published_at` | string or null | no | Publication date string from the catalog (ISO or storefront format). |
| `genres` | string or null | no | Candidate genres (not taste-seed `seed_categories`). |
| `language` | string or null | no | BCP-47 / storefront language code (`en`, `de`, …) when known. |
| `wishers` | array of [`QueueWisher`](#queuewisher) | no | Distinct people who have this title on an open wishlist. |
| `wish_count` | integer (int64) | no | Number of distinct users who wishlisted this title (may exceed `wishers.len()`). |

## RedeemBody

JSON body for `POST /redeem` (claim ticket plus retry nonce).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ticket` | string | yes | Raw claim-ticket token to redeem into a portal session. |
| `nonce` | string | yes | Browser-generated one-time nonce persisted across HTTP retries. |
| `password` | string or null | no | Required when the linked local user has no password (invite / reset). |

## RemoveBranchTitles

`POST /api/config/branches/titles/remove` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `node_id` | string or null | no | Only this branch's entries; every branch's when absent. |

## RequestStatus

Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).

Type: one of `open`, `cancelled`

## RevokeBody

`POST /api/config/cluster-consents/revoke` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | Owner cluster id. |
| `plugin_key` | string | yes | Canonical plugin key. |

## RevokeForm

Form body for `POST /oidc/revoke` (RFC 7009-style refresh-token revoke).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `token` | string | yes | Token to revoke; treated as a refresh token (hashed before lookup). |
| `token_type_hint` | string or null | no | Optional hint ignored; this server only stores refresh tokens. |

## RotateSecretBody

Optional password for rotating a confidential client's secret.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `current_password` | string or null | no | Required for a non-elevated Owner whose portal session is older than 15 minutes. |

## ScanPlanResponse

Result of a planned scan or acquire: [`ActionResponse`] plus every job id.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ok` | boolean | yes | True when at least one job was accepted. |
| `message` | string | yes | Operator-facing status text. |
| `job_id` | string | yes | First accepted (or, when none was, already queued) job id. |
| `job_ids` | array of string | yes | Every accepted or already-queued job id, in plan order. |
| `error` | string or null | no | Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`). |

## ScanRequest

Optional body for `POST /api/library/scan`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `account` | string or null | no | When set, scan this account even if `scan_enabled` is false. |
| `plugin_instance_id` | string or null | no | When set, scan only this deployed plugin instance. |

## SetPasswordRequest

JSON body for setting a password (self or provisioner target).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `password` | string | yes | New password; hashed before persist. Empty is rejected. |
| `current_password` | string or null | no | Required when the caller already has a password (not first-time invite setup). |
| `user_id` | integer (int64) or null (int64) | no | Target `users.id` when a provisioner sets another user's password. |

## SettingsResponse

Settings page payload: file values, effective runtime, and plugin groups.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `settings` | map of string to string | yes | Values as written in config.toml (and env overlays), plus `output.*` from the `core.output` database document. |
| `effective` | map of string to string | yes | Runtime-effective values after the last successful reload (auth, plugins). |
| `plugins` | array of [`PluginSettingsGroup`](#pluginsettingsgroup) | yes | Per-plugin setting groups rendered by the Settings UI. |
| `host_cpu_cores_max` | number (double) | yes | Host max jail CPU in cores (2 d.p.; equals logical CPU count). |
| `jail_cpu_cores` | number (double) or null (double) | no | Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`). |
| `output_revision` | integer (int64) or null (int64) | no | Revision of the `core.output` document the `output.*` values in `settings` were read from; send it back as `expected_output_revision`. `effective["output.revision"]` is the revision this host has applied, which lags the document until the host's next wakeup. |

## SettingsUpdate

One `config.toml` key/value pair from a settings PATCH.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `key` | string | yes | Dotted config key; rejected unless it passes the settings allowlist. |
| `value` | string | yes | Raw string value, normalized before it is written. |

## SharingBody

`POST /api/config/join/sharing` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | The library (owner cluster id). |
| `share_books` | boolean | yes | "Share this branch's books with *library*". |
| `include_branches` | boolean | yes | "Include books from my own branches". |

## ShelfKindInfo

A shelf kind the operator can ignore (all offered by default).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Stable kind id (`finish_series`, `author`, `genre`, `from_store`, …). |
| `label` | string | yes | Operator-visible label for this account or item. |

## SourceConfigOptionInfo

One storefront config option (key, label, allowed values).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `key` | string | yes | Config table key (for example `container`). |
| `label` | string | yes | Operator-facing option name. |
| `values` | array of [`ConfigOptionValueInfo`](#configoptionvalueinfo) | yes | Allowed values the SPA may offer for this option. |

## SourceInfo

One enabled storefront as shown on the portal Accounts page.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Source plugin id (`audible`, `libro`, …). |
| `name` | string | yes | Operator-facing storefront name. |
| `auth` | string | yes | Portal auth mode (`password` or `oauth`). |
| `config_options` | array of [`SourceConfigOptionInfo`](#sourceconfigoptioninfo) | no | Store-specific knobs the SPA can present (omitted when empty). |
| `brand` | [`BrandInfo`](#brandinfo) | yes | Storefront colors and logo href for the Accounts UI. |

## SourcesResponse

JSON body for `GET /sources`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `sources` | array of [`SourceInfo`](#sourceinfo) | yes | Enabled storefronts the SPA may offer for connect / login. |

## StartBody

`POST /api/config/join/start` body.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `address` | string | yes | The library's address. |
| `fingerprint` | string or null | no | The fingerprint the operator confirmed (self-signed libraries). |
| `branch_name` | string or null | no | The name to join as (default: this machine's name). |
| `clerk` | boolean | no | Offer to join as a clerk (an owner node that shares the library's database): only an empty install may, and its data is replaced. |

## StatusResponse

Library and daemon snapshot for `GET /api/status`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `accounts` | integer | yes | Count of configured store accounts. At least 0. |
| `books` | integer | yes | Total title rows in the library. At least 0. |
| `acquired` | integer (int64) | yes | Titles whose `acquire_status` is `acquired`. |
| `pending` | integer (int64) | yes | Titles still needing acquire (`not_acquired`). |
| `error` | integer (int64) | yes | Titles stuck in `error` after a failed acquire. |
| `in_progress` | integer (int64) | yes | Titles currently `queued` or `downloading`. |
| `listen` | string | yes | Comma-joined `daemon.listen` addresses from live config. |
| `storage_backend` | string | yes | Comma-joined enabled destination plugin ids, or `none`. |
| `auth_enabled` | boolean | yes | Whether the live auth middleware requires operator credentials. |
| `events` | [`EventQueueStatus`](#eventqueuestatus) | yes | Durable domain-event delivery-queue counters. |

## StepUp

A fresh second factor for a clerk consent. `Debug` prints no code.

Type: object

## StoreEdition

One storefront edition of a work (for multi-store purchase links).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `source` | string | yes | Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …). |
| `product_id` | string | yes | Storefront-native product id (ASIN, ISBN, UUID, …). |

## TcpGrantView

Operator-facing TCP grant (`host` + `ports`).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `host` | string | yes | Hostname or `*.` wildcard. |
| `ports` | array of integer (int32) | yes | Destination ports. |

## TitleMeta

Bibliographic fields suitable for an Audible-style detail panel.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `title` | string or null | no | Display title as shown on the storefront or library card. |
| `subtitle` | string or null | no | Optional subtitle when the catalog distinguishes it from the title. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `narrators` | string or null | no | Comma-separated narrator names when known. |
| `series` | string or null | no | Series name when the title belongs to a named series. |
| `series_index` | string or null | no | Position within the series (e.g. `1`, `1.5`) when known. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `cover_url` | string or null | no | HTTPS URL for cover art when the catalog exposes one. |
| `description` | string or null | no | Publisher or storefront synopsis; may be truncated for embeddings. |
| `publisher` | string or null | no | Publisher imprint as reported by the catalog. |
| `length_minutes` | integer (int64) or null (int64) | no | Audiobook runtime in whole minutes when known. |
| `published_at` | string or null | no | Publication date string from the catalog (ISO or storefront format). |
| `categories` | string or null | no | Genre / subject tags as a single delimited string when known. |
| `language` | string or null | no | BCP-47 / storefront language code (`en`, `de`, …) when known. |
| `is_abridged` | boolean or null | no | `Some(true)` abridged / `Some(false)` unabridged when the storefront said so. |
| `rating_overall` | number (double) or null (double) | no | Audible community overall rating (0–5). |
| `rating_performance` | number (double) or null (double) | no | Narration / performance sub-score when the storefront splits ratings. |
| `rating_story` | number (double) or null (double) | no | Story / content sub-score when the storefront splits ratings. |
| `rating_count` | integer (int64) or null (int64) | no | Count of star ratings (Audible `num_ratings`). |
| `review_count` | integer (int64) or null (int64) | no | Count of written reviews (Audible `num_reviews`). |
| `reviews` | array of [`TitleReview`](#titlereview) | no | Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty for older clients; prefer the paginated reviews endpoint. |

## TitleMetaBatchBody

Body for batched title-metadata lookups (max 24 queries).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `queries` | array of [`TitleMetaQuery`](#titlemetaquery) | no | Per-title metadata queries in this batch. |

## TitleMetaBatchResponse

Title-metadata results aligned with the request; failed items are `null`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results` | array of [`TitleMeta`](#titlemeta) or null | yes | One optional metadata payload per query; enrichment errors become `None`. |

## TitleMetaQuery

Query for public title metadata (Audnexus / Audible catalog).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | no | Display title as shown on the storefront or library card. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `narrators` | string or null | no | Comma-separated narrator names when known. |
| `length_minutes` | integer (int64) or null (int64) | no | Audiobook runtime in whole minutes when known. |
| `region` | string | no | Marketplace / region code (`us`, `uk`, …) for catalog lookups. |

## TitleRequestRecord

Personal wishlist row (also contributes to the shared global queue while open).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer (int64) | yes | Surrogate primary key assigned by the database. |
| `uuid` | string | yes | Stable UUID for this row (API / foreign-key identity). |
| `identity_id` | integer (int64) or null (int64) | no | Foreign key to `portal_identities.id`. |
| `title` | string | yes | Display title of the work or edition. |
| `authors` | string or null | no | Comma-separated or JSON author list from the storefront. |
| `asin` | string or null | no | Amazon ASIN when known; otherwise null. |
| `isbn` | string or null | no | ISBN-10/13 when known; otherwise null. |
| `notes` | string or null | no | Free-form operator or requester notes. |
| `status` | [`RequestStatus`](#requeststatus) | yes | Lifecycle status for the row (user, request, …). |
| `work_key` | string | no | Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation. |
| `work_id` | string or null | no | Canonical work id this edition or request resolves to. |
| `resolved_book_uuid` | string or null | no | Library `books.uuid` once the wishlist item is fulfilled. |
| `cover_url` | string or null | no | HTTPS URL for cover art when known. |
| `sources` | array of [`TitleRequestSourceRecord`](#titlerequestsourcerecord) | no | Per-storefront snapshots (empty for legacy rows). |
| `description` | string or null | no | Blurb / synopsis text (may contain HTML). |
| `subtitle` | string or null | no | Optional subtitle from bibliographic metadata. |
| `narrators` | string or null | no | Comma-separated or JSON narrator list when present. |
| `series` | string or null | no | Series name when the title belongs to a series. |
| `series_index` | string or null | no | Position within the series (storefront string form). |
| `publisher` | string or null | no | Publisher name from metadata enrichment or the storefront. |
| `length_minutes` | integer (int64) or null (int64) | no | Runtime in whole minutes when the storefront reports it. |
| `published_at` | string or null | no | Publication date string from the storefront or enrichment. |
| `genres` | string or null | no | Merged genre / category list for display. |
| `language` | string or null | no | BCP-47 or storefront language code when known. |
| `store_editions` | array of [`WishlistStoreEdition`](#wishliststoreedition) | no | Distinct storefront editions contributing to this wishlist item. |
| `purchase_hints` | array of [`WishlistPurchaseHint`](#wishlistpurchasehint) | no | Purchase URLs and prices observed per storefront. |
| `created_at` | string (date-time) | yes | RFC 3339 timestamp when the row was inserted. |
| `updated_at` | string (date-time) | yes | RFC 3339 timestamp when the row was last modified. |

## TitleRequestSourceRecord

Per-storefront catalog/pricing snapshot attached to a wishlist row.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer (int64) | yes | Surrogate primary key assigned by the database. |
| `title_request_id` | integer (int64) | yes | Foreign key to `title_requests.id`. |
| `source` | string | yes | Content-source plugin id (`audible`, `libro`, …). |
| `product_id` | string | yes | Storefront product id (ASIN, ISBN, UUID, …). |
| `title` | string or null | no | Display title of the work or edition. |
| `subtitle` | string or null | no | Optional subtitle from bibliographic metadata. |
| `authors` | string or null | no | Comma-separated or JSON author list from the storefront. |
| `narrators` | string or null | no | Comma-separated or JSON narrator list when present. |
| `series` | string or null | no | Series name when the title belongs to a series. |
| `series_index` | string or null | no | Position within the series (storefront string form). |
| `asin` | string or null | no | Amazon ASIN when known; otherwise null. |
| `isbn` | string or null | no | ISBN-10/13 when known; otherwise null. |
| `description` | string or null | no | Blurb / synopsis text (may contain HTML). |
| `publisher` | string or null | no | Publisher name from metadata enrichment or the storefront. |
| `length_minutes` | integer (int64) or null (int64) | no | Runtime in whole minutes when the storefront reports it. |
| `published_at` | string or null | no | Publication date string from the storefront or enrichment. |
| `categories` | string or null | no | Storefront category / genre path list (serialized). |
| `language` | string or null | no | BCP-47 or storefront language code when known. |
| `cover_url` | string or null | no | HTTPS URL for cover art when known. |
| `url` | string or null | no | Storefront product or purchase URL when known. |
| `price_cents` | integer (int64) or null (int64) | no | Observed price in minor currency units, when known. |
| `currency` | string or null | no | ISO 4217 currency code for price fields. |
| `price_label` | string or null | no | Storefront-formatted price string for display. |
| `list_price_cents` | integer (int64) or null (int64) | no | List/MSRP price in minor units, when known. |
| `list_price_label` | string or null | no | Storefront-formatted list price for display. |
| `member_price_cents` | integer (int64) or null (int64) | no | Member/subscriber price in minor units, when known. |
| `member_price_label` | string or null | no | Storefront-formatted member price for display. |
| `observed_at` | string (date-time) or null (date-time) | no | RFC 3339 time when this storefront snapshot was observed. |
| `created_at` | string (date-time) | yes | RFC 3339 timestamp when the row was inserted. |
| `updated_at` | string (date-time) | yes | RFC 3339 timestamp when the row was last modified. |

## TitleReview

One Audible customer review for the title detail panel.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string or null | no | Stable identifier for this item. |
| `title` | string or null | no | Display title as shown on the storefront or library card. |
| `body` | string | yes | Full review text from the storefront. |
| `author_name` | string or null | no | Display name of the review author on the storefront. |
| `overall_rating` | integer (int64) or null (int64) | no | Overall star rating for this review when present. |
| `performance_rating` | integer (int64) or null (int64) | no | Narration sub-rating for this review when present. |
| `story_rating` | integer (int64) or null (int64) | no | Story sub-rating for this review when present. |
| `submitted_at` | string or null | no | When the review was submitted (storefront timestamp string). |

## TitleReviewsPage

One page of title reviews for infinite scroll.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asin` | string | yes | Audible / Amazon ASIN when this edition is sold on Audible. |
| `page` | integer (int32) | yes | 1-based page index for this reviews / list request. At least 0. |
| `page_size` | integer (int32) | yes | Maximum number of items returned in this page. At least 0. |
| `has_more` | boolean | yes | When true, another page is available by incrementing [`Self::page`]. |
| `sort_by` | string | yes | Remote sort key for reviews (`most_helpful`, `most_recent`, …). |
| `reviews` | array of [`TitleReview`](#titlereview) | yes | Review rows on this page. |

## TitleReviewsQuery

Query for a page of Audible customer reviews.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asin` | string | yes | Audible / Amazon ASIN when this edition is sold on Audible. |
| `region` | string | no | Marketplace / region code (`us`, `uk`, …) for catalog lookups. |
| `page` | integer (int32) | no | 1-based page index. At least 0. |
| `page_size` | integer (int32) | no | Page size (clamped server-side to 1..=20). At least 0. |
| `sort_by` | string | no | `MostHelpful` (default) or `MostRecent`. |

## TokenForm

`application/x-www-form-urlencoded` body for `POST /oidc/token`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `grant_type` | string | yes | Grant: `authorization_code` or `refresh_token`; anything else is 400. |
| `code` | string or null | no | Authorization code from the redirect (`authorization_code` grant). |
| `redirect_uri` | string or null | no | Must match the URI bound to the consumed authorization code. |
| `client_id` | string or null | no | Client id; required for code exchange, optional (but checked) on refresh. |
| `client_secret` | string or null | no | Client secret for confidential clients (`client_secret_post`). |
| `code_verifier` | string or null | no | PKCE verifier; SHA-256 must match the stored challenge. |
| `refresh_token` | string or null | no | Refresh token to rotate (`refresh_token` grant). |

## TokenResponse

Successful token-endpoint JSON (access, optional refresh, and ID tokens).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `access_token` | string | yes | HS256 JWT access token bound to the User (`token_use=access`). |
| `token_type` | string | yes | Always `Bearer` for this server. |
| `expires_in` | integer (int64) | yes | Access-token lifetime in seconds (same as [`ACCESS_TTL_SECS`]). |
| `refresh_token` | string or null | no | Opaque refresh token when the client is allowed to receive one. |
| `id_token` | string | yes | HS256 ID token; profile/email claims follow the granted scope. |
| `scope` | string | yes | Space-separated scopes granted to this token pair. |

## TotpReauthBody

Optional current password for enroll/disable step-up.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `current_password` | string or null | no | Password used for recent-reauth when the user already has a credential. |

## UploadAnswer

Answer of the upload endpoint (every status carries one).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string | yes | Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`. |
| `reason` | string or null | no | Why, for `refused`. |
| `job_completed` | boolean | no | For `committed`: the job is finished. |

## UploadCommitRequest

Body of `POST <UPLOADS_PATH><upload_id>/commit`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `finish` | boolean | no | Complete the job with this commit (the job's closing upload). |
| `detail` | string or null | no | Outcome text recorded as the job's progress when `finish` is set. |

## WishlistPurchaseHint

Snapshotted purchase link/price for a wishlist title.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `source` | string | yes | Content-source plugin id (`audible`, `libro`, …). |
| `product_id` | string | yes | Storefront product id (ASIN, ISBN, UUID, …). |
| `title` | string or null | no | Display title of the work or edition. |
| `url` | string or null | no | Storefront product or purchase URL when known. |
| `price_cents` | integer (int64) or null (int64) | no | Observed price in minor currency units, when known. |
| `currency` | string or null | no | ISO 4217 currency code for price fields. |
| `price_label` | string or null | no | Storefront-formatted price string for display. |
| `list_price_cents` | integer (int64) or null (int64) | no | List/MSRP price in minor units, when known. |
| `list_price_label` | string or null | no | Storefront-formatted list price for display. |
| `member_price_cents` | integer (int64) or null (int64) | no | Member/subscriber price in minor units, when known. |
| `member_price_label` | string or null | no | Storefront-formatted member price for display. |

## WishlistStoreEdition

Storefront edition key for wishlist / queue payloads.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `source` | string | yes | Content-source plugin id (`audible`, `libro`, …). |
| `product_id` | string | yes | Storefront product id (ASIN, ISBN, UUID, …). |
