# Schemas

The request and response bodies of the [web API](README.md), in alphabetical order.

## AcquireRequestBody

Optional body for `POST /api/library/acquire`.

- `asin` *string or null* — Audible ASIN filter; first of uuid/asin/isbn/product_id wins.
- `uuid` *string or null* — Library title UUID filter.
- `isbn` *string or null* — ISBN filter for non-Audible stores.
- `product_id` *string or null* — Store product id filter.
- `account` *string or null* — Restrict acquire to this store account.
- `plugin_instance_id` *string or null* — When set, fetch through this deployed plugin instance.
- `titles` *array of string* — More title ids; each (with the single filter above) becomes its own job.
- `force` *boolean* — Re-download titles already acquired.
- `pdf_only` *boolean* — Fetch only each title's companion PDF.

## AcquireStatus

Acquire / download state for a title.

Type: one of `not_acquired`, `queued`, `downloading`, `acquired`, `error`

## ActionResponse

Generic mutation result with success flag, message, and optional job id.

- `ok` *boolean*, **required** — True when the action was accepted or completed.
- `message` *string*, **required** — Operator-facing status text.
- `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
- `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).

## ApiError

An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.

- `error` *string*, **required** — Stable machine slug.
- `message` *string* — Explanation for people.
- `status` *integer* — The HTTP status, repeated.

## ApproveBody

`POST /api/config/cluster-consents/approve` body.

- `cluster_id` *string*, **required** — Owner cluster id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `request_digest` *string*, **required** — The `request_digest` of the request the operator reviewed, as the listing shows it (refused when the recorded request is another one).
- `deny_domains` *array of string* — Domains to take out of the request. No duplicates.
- `deny_bindings` *array of string* — Bindings to take out of the request. No duplicates.
- `deny_network` *boolean* — Deny all network access.

## BinaryBody

A request body of raw bytes (a file upload), documented as
`type: string, format: binary` so generated clients send the bytes
rather than a JSON array.

Type: binary

## BookRecord

Book / library item (one ownership row per store product per account).

- `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
- `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
- `plugin_key` *string*, **required** — Provenance-qualified PluginKey that owns the account.
- `source` *string*, **required** — Display alias of the owning plugin (`audible`, `libro`, …).
- `account_id` *string*, **required** — Store or operator account id this row belongs to.
- `product_id` *string*, **required** — Storefront product id (ASIN, ISBN, UUID, …).
- `asin` *string or null* — Audible ASIN when known (`None` for Libro-only rows without enrichment).
- `isbn` *string or null* — ISBN-13 when known (Libro always; Audible when the API provides it).
- `marketplace` *string*, **required** — Store marketplace / locale code (for example `us`, `uk`).
- `title` *string*, **required** — Display title of the work or edition.
- `authors` *string or null* — Comma-separated or JSON author list from the storefront.
- `narrators` *string or null* — Comma-separated or JSON narrator list when present.
- `series` *string or null* — Series name when the title belongs to a series.
- `series_index` *string or null* — Position within the series (storefront string form).
- `series_asin` *string or null* — Audible series / podcast-parent ASIN (`Series.AudibleSeriesId`).
- `acquire_status` *[`AcquireStatus`](#acquirestatus)*, **required** — Download/acquire pipeline state (`not_acquired`, `queued`, …).
- `storage_key` *string or null* — Object-storage key for the primary audio artifact, if acquired.
- `error_message` *string or null* — Last acquire/convert failure message for operators.
- `purchased_at` *string (date-time) or null* — RFC 3339 purchase time from the storefront, when known.
- `tags` *string or null* — Operator or storefront tags (serialized string).
- `rating_overall` *number (float) or null* — Overall user rating from the storefront, if any.
- `rating_performance` *number (float) or null* — Narration/performance rating from the storefront, if any.
- `rating_story` *number (float) or null* — Story rating from the storefront, if any.
- `is_finished` *boolean*, **required** — Whether the listener marked the title finished (0/1 or bool).
- `pdf_status` *[`AcquireStatus`](#acquirestatus)*, **required** — Companion PDF acquire state (`not_acquired`, `acquired`, …).
- `pdf_storage_key` *string or null* — Object-storage key for the companion PDF, if present.
- `publisher` *string or null* — Publisher name from metadata enrichment or the storefront.
- `length_minutes` *integer (int64) or null* — Runtime in whole minutes when the storefront reports it.
- `is_abridged` *boolean*, **required** — Whether the edition is abridged (0/1 or bool).
- `content_kind` *string*, **required** — Title kind: `book`, `episode`, `podcast`, ….
- `categories` *string or null* — Storefront category / genre path list (serialized).
- `subtitle` *string or null* — Optional subtitle from bibliographic metadata.
- `published_at` *string (date-time) or null* — Publication date string from the storefront or enrichment.
- `description` *string or null* — Blurb / synopsis text (may contain HTML).
- `language` *string or null* — BCP-47 or storefront language code when known.
- `cover_url` *string or null* — HTTPS URL for cover art when known.
- `subjects` *string or null* — Subject / topic tags (often from Open Library; `;`- or `,`-separated).
- `enrich_source` *string or null* — Plugin or catalog that last enriched bibliographic fields.
- `enrich_confidence` *number (double) or null* — 0–1 confidence score for the last enrichment pass.
- `enrich_updated_at` *string (date-time) or null* — RFC 3339 time of the last enrichment write.
- `created_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was inserted.
- `updated_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was last modified.

## BooksResponse

Paginated library listing.

- `books` *array of [`BookRecord`](#bookrecord)*, **required** — Current page of title records.
- `total` *integer*, **required** — Match count before pagination. At least 0.
- `limit` *integer*, **required** — Applied page size. At least 0.
- `offset` *integer*, **required** — Applied skip count. At least 0.

## BootstrapRequest

Operator-only body for creating the first Owner when none exist.

- `display_name` *string or null* — Optional display name; falls back to login or email.
- `login_name` *string or null* — Optional local login name.
- `email` *string or null* — Optional email.
- `password` *string or null* — Optional password; hashed before persist and never stored in plaintext.

## BranchAuthorizeParams

The authorization request a consent is for (the query `/oidc/authorize`
forwarded to `/join/approve`).

- `client_id` *string*, **required** — Must be [`BRANCH_CLIENT_ID`].
- `redirect_uri` *string*, **required** — Where the code goes: loopback or https.
- `response_type` *string*, **required** — Must be `code`.
- `scope` *string or null* — Must be [`BRANCH_SCOPE`].
- `state` *string or null* — Echoed on the redirect.
- `code_challenge` *string*, **required** — PKCE S256 challenge.
- `code_challenge_method` *string or null* — Must be `S256`.
- `branch_name` *string or null* — The name the branch asks to join as.
- `clerk_offer` *string or null* — `1` when the joining install offers to join as a clerk ([`bookclerk_node_proto::CLERK_OFFER_PARAM`]).

## BranchConsentBody

`POST /api/auth/branch-consent` body.

Includes every field of [`BranchAuthorizeParams`](#branchauthorizeparams).

- `approve` *boolean*, **required** — True to let the branch join; false to deny.
- `connection` *string or null* — `branch` (default) or `clerk`.
- `step_up` *[`StepUp`](#stepup) or null* — The fresh second factor a clerk needs.

## BrandInfo

CSS colors and logo href for a storefront or integration brand.

- `bg` *string*, **required** — Background color (`#RRGGBB`).
- `fg` *string*, **required** — Foreground / text color (`#RRGGBB`).
- `accent` *string*, **required** — Accent color (`#RRGGBB`).
- `logo` *string*, **required** — Logo image href (often a favicon URL).

## CallbackBody

`POST /api/config/join/callback` body.

- `code` *string* — The code from the library.
- `state` *string* — The `state` it came back with.
- `error` *string or null* — The library's `error` (`access_denied`), when it sent one.
- `connection` *string or null* — The connection type the library's user chose (`clerk`; a branch when left out). The code is bound to it, so a changed value only fails the redeem.

## CallbackParams

Query or form_post fields from the IdP callback.

- `code` *string or null* — Authorization code to exchange; required unless `error` is set.
- `state` *string or null* — CSRF `state` that must match the `bookclerk_oidc_tx` cookie.
- `error` *string or null* — IdP-reported deny/error; redirects the browser to `/?sso_error=denied`.
- `user` *string or null* — Apple first-login name/email JSON (form_post only).

## CatalogSearchHit

One autocomplete / results-page hit (possibly spanning multiple storefronts).

- `work_key` *string*, **required** — Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `narrators` *string or null* — Comma-separated narrator names when known.
- `series` *string or null* — Series name when the title belongs to a named series.
- `series_index` *string or null* — Position within the series (e.g. `1`, `1.5`) when known.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
- `store_editions` *array of [`StoreEdition`](#storeedition)*, **required** — Per-storefront edition ids that collapsed into this work card.
- `sources` *array of string*, **required** — Storefronts that matched (deduped source ids).
- `subtitle` *string or null* — Optional subtitle when the catalog distinguishes it from the title.
- `description` *string or null* — Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher` *string or null* — Publisher imprint as reported by the catalog.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `published_at` *string or null* — Publication date string from the catalog (ISO or storefront format).
- `genres` *string or null* — Genre labels from the catalog (comma-separated when multiple).
- `language` *string or null* — BCP-47 / storefront language code (`en`, `de`, …) when known.
- `is_abridged` *boolean or null* — When `Some(true)`, the edition is marked abridged by the storefront.
- `rating_overall` *number (double) or null* — Aggregate listener rating on the storefront's scale when known.
- `rating_count` *integer (int64) or null* — Number of ratings backing [`Self::rating_overall`] when known.
- `price_cents` *integer (int64) or null* — Catalog list/deal price in cents when the storefront provided it.
- `purchase_hints` *array of [`PurchaseHint`](#purchasehint)* — Buy / open-in-store links with optional member vs list pricing.

## CatalogSearchPage

Paged catalog-search response.

- `items` *array of [`CatalogSearchHit`](#catalogsearchhit)*, **required** — Ordered collection of child entries for this response.
- `page_size` *integer*, **required** — Maximum number of items returned in this page. At least 0.
- `has_more` *boolean*, **required** — When true, another page is available via [`Self::next_cursor`].
- `next_cursor` *string or null* — Opaque cursor for the next page; omit or `None` when exhausted.
- `sort` *string*, **required** — Sort field applied to the merged result set.
- `sort_dir` *string* — Ascending or descending order for [`Self::sort`].

## CeremonyFinish

Browser attestation/assertion plus the challenge id from `begin`.

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — Browser `PublicKeyCredential` JSON for `finish_*`.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

## ClusterJobsBody

`POST /api/config/clusters/{cluster_id}/jobs` body.

- `job_ids` *array of string*, **required** — Job ids on the owner cluster.

## ClusterScanBody

`POST /api/config/clusters/{cluster_id}/scan` body.

- `account` *string or null* — Scan only this store account of the owner cluster.
- `plugin_instance_id` *string or null* — Pin every planned job to this owner plugin instance.

## ConfigOptionValueInfo

One allowed value for a storefront config option.

- `id` *string*, **required** — Stored option value (for example `m4b`).
- `label` *string*, **required** — Operator-facing value label.

## ConnectionInfo

One linked store account on the portal Accounts page.

- `account_id` *string*, **required** — Store account id (unique within its source).
- `source` *string*, **required** — Source plugin id that owns this account.
- `plugin_key` *string*, **required** — Provenance-qualified PluginKey that owns the account.
- `label` *string or null* — Operator-facing account label when the store provided one.
- `connection_status` *string*, **required** — Connection state (`active`, revoked, …); defaults to `active` if the account row is missing.
- `source_enabled` *boolean*, **required** — Whether `[sources.<id>] enabled` is currently true (new connects blocked when false).
- `brand` *[`BrandInfo`](#brandinfo) or null* — Storefront or integration brand when a plugin or fallback palette is available.

## ConnectionsResponse

JSON body for `GET /connections`.

- `connections` *array of [`ConnectionInfo`](#connectioninfo)*, **required** — Linked store accounts the SPA may show or revoke.

## ConnectionType

How a joining install connects to a library.

Type: one of `branch`, `clerk`

## ConsentBody

Form body for `POST /oidc/authorize` after the user grants or denies consent.

- `client_id` *string*, **required** — Registered OIDC client id (must exist in the library store).
- `redirect_uri` *string*, **required** — Redirect URI; must match a URI registered for the client.
- `response_type` *string*, **required** — OAuth `response_type`; only `code` is accepted.
- `scope` *string or null* — Space-separated scopes; defaults to `openid profile` when omitted.
- `state` *string or null* — Opaque CSRF value echoed back on the redirect.
- `code_challenge` *string*, **required** — PKCE S256 challenge (base64url SHA-256 of the verifier).
- `code_challenge_method` *string or null* — PKCE method; omitted is treated as `plain` and then rejected.
- `consent` *string or null* — `deny` aborts with `access_denied`; any other value proceeds to issue a code.

## CreatePluginInstance

`POST /api/config/plugin-instances` body.

- `plugin_key` *string*, **required** — Canonical plugin key; the plugin must have a pinned schema.
- `display_name` *string* — Operator-facing name.
- `node_id` *string or null* — Make the instance local to this node (H-2): it deploys only there and is never placed by the scheduler.

## CreateRequestBody

Wishlist / title-request create body from Discover or the portal.

- `title` *string*, **required** — Required title text; empty values are rejected with 400.
- `authors` *string or null* — Optional author list as a single display string.
- `asin` *string or null* — Optional Audible ASIN used for work-key identity.
- `isbn` *string or null* — Optional ISBN used for work-key identity.
- `notes` *string or null* — Optional operator or member note stored on the request.
- `store_editions` *array of [`StoreEdition`](#storeedition)* — Known storefront editions to persist as `title_request_sources`.
- `purchase_hints` *array of [`PurchaseHint`](#purchasehint)* — Optional priced storefront links snapshotted at wishlist time.
- `work_key` *string or null* — Optional stable work key; when omitted one is derived from bib fields.
- `cover_url` *string or null* — Optional cover image URL snapshotted onto the request.
- `description` *string or null* — Optional plot/summary text persisted with the request.
- `subtitle` *string or null* — Optional subtitle stored on the request and source rows.
- `narrators` *string or null* — Optional narrator list as a single display string.
- `series` *string or null* — Optional series name.
- `series_index` *string or null* — Optional series position as a display string.
- `publisher` *string or null* — Optional publisher name.
- `length_minutes` *integer (int64) or null* — Optional runtime in minutes.
- `published_at` *string or null* — Optional publication date as a display string.
- `genres` *string or null* — Optional genre list as a single display string.
- `language` *string or null* — Optional content language code.

## CreateUserRequest

Provisioner body for creating a user and optional invite ticket.

- `role` *string or null* — Role to assign when permitted (`member` when omitted).
- `display_name` *string or null* — Optional display name for the new user.
- `login_name` *string or null* — Optional local login name for the new user.
- `email` *string or null* — Optional email for the new user.
- `password` *string or null* — Optional password; hashed before persist. Empty or omitted leaves the user passwordless.
- `mint_invite` *boolean* — When true (default), also mint an invite/claim ticket.

## DatabaseMigrateRequest

Body for copying library rows between database plugins.

- `from` *string or null* — Source plugin id. Defaults to the active `[database].plugin` before reload.
- `to` *string*, **required** — Destination database plugin id to copy into.
- `dry_run` *boolean* — When true, count rows without writing.
- `force` *boolean* — When true, overwrite a destination that is not empty.
- `apply` *boolean* — Update `[database].plugin` in config.toml after a successful copy.

## DesiredState

What a deployment row asks its node to do, and who decided it.

Operators write [`DesiredState::Present`] and [`DesiredState::Absent`]
(`plugins deploy|undeploy`, the Settings switch). The placement scheduler
writes [`DesiredState::Placed`] and [`DesiredState::Unplaced`] for an
instance it owns. A node runs the instance for `present` and `placed`
and retires it for `absent` and `unplaced`; the split only records whose
decision the row carries, so "the operator turned this off" never reads
like "the scheduler put the replica elsewhere".

Type: one of `present`, `absent`, `placed`, `unplaced`

## DiscoverFeed

Full Discover page payload: ordered shelves of personalized candidates.

- `shelves` *array of [`DiscoverShelf`](#discovershelf)*, **required** — Ordered horizontal shelves that make up the Discover page.
- `shelf_kinds` *array of [`ShelfKindInfo`](#shelfkindinfo)* — Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on).

## DiscoverShelf

One horizontal Discover row (series to finish, more by author, …).

- `id` *string*, **required** — Stable identifier for this item.
- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `subtitle` *string or null* — Optional subtitle when the catalog distinguishes it from the title.
- `items` *array of [`Recommendation`](#recommendation)*, **required** — Ordered collection of child entries for this response.

## DiscoveryDocument

OIDC discovery document served at `/.well-known/openid-configuration`.

- `issuer` *string*, **required** — Issuer origin used as `iss` in minted JWTs (`public_origin` or loopback).
- `authorization_endpoint` *string*, **required** — Absolute URL of the authorization endpoint (`/oidc/authorize`).
- `token_endpoint` *string*, **required** — Absolute URL of the token endpoint (`/oidc/token`).
- `userinfo_endpoint` *string*, **required** — Absolute URL of the userinfo endpoint (`/oidc/userinfo`).
- `revocation_endpoint` *string*, **required** — Absolute URL of the refresh-token revocation endpoint (`/oidc/revoke`).
- `response_types_supported` *array of string*, **required** — Supported `response_type` values; this server only issues `code`.
- `grant_types_supported` *array of string*, **required** — Supported token grants: `authorization_code` and `refresh_token`.
- `code_challenge_methods_supported` *array of string*, **required** — PKCE methods; only S256 is accepted (`plain` is rejected).
- `scopes_supported` *array of string*, **required** — Scopes advertised to clients (`openid`, `profile`, and `email`).
- `subject_types_supported` *array of string*, **required** — Subject identifier types; only public (stable user id) is used.
- `id_token_signing_alg_values_supported` *array of string*, **required** — ID-token signing algorithms; tokens are HS256 with a derived key.
- `token_endpoint_auth_methods_supported` *array of string*, **required** — Client auth methods: public PKCE (`none`) or `client_secret_post`.

## DomainEventInfo

Operator JSON for one `domain_events` row.

- `id` *string*, **required** — Event id.
- `eventType` *string*, **required** — Event type (`book_acquired`).
- `schemaVersion` *integer (int64)*, **required** — Payload schema version.
- `dispatchState` *string*, **required** — `pending` or `dispatched`.
- `accountId` *string*, **required** — Tenant / account id.
- `dedupKey` *string*, **required** — Producer idempotency key.
- `createdAt` *string*, **required** — RFC 3339 insert time.
- `queueName` *string*, **required** — Queue the row belongs to (`core.events` for domain events).
- `contentType` *string*, **required** — Body encoding (`json`, `text`, `bytes`).
- `source` *string*, **required** — Producer plugin id; empty when the host published it.
- `deliverAfter` *string or null* — RFC 3339 earliest delivery time for delayed queue messages.

## ElevateRequest

JSON body for Owner password re-auth before issuing an elevated operator cookie.

- `password` *string*, **required** — Owner account password (re-authentication).

## EnrollBody

`POST /api/node/v1/enroll` body. Keys are base64url without padding.

- `link_id` *string*, **required** — Link id from the token.
- `secret` *string*, **required** — Link secret from the token.
- `sign_pk` *string*, **required** — Ed25519 public signing key.
- `box_pk` *string*, **required** — X25519 public box key.
- `software_version` *string* — Host's Bookclerk version.

## EnrollFinish

First TOTP code that confirms the pending secret.

- `code` *string*, **required** — Six-digit code from the authenticator app.

## ErrorJson

JSON error object (`error`, `message`, numeric `status`) for API clients.

- `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
- `message` *string*, **required** — Operator-facing explanation.
- `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.

## EventDeliveryInfo

Operator JSON for one `event_deliveries` row.

- `id` *string*, **required** — Delivery id (`{event_id}:{plugin_id}`).
- `eventId` *string*, **required** — Parent event id.
- `pluginId` *string*, **required** — Subscriber plugin id.
- `state` *string*, **required** — Lifecycle state.
- `attemptCount` *integer (int64)*, **required** — Claim count (not incremented on resume).
- `leaseGeneration` *integer (int64)*, **required** — Fence generation.
- `invocationSequence` *integer (int64)*, **required** — Resume ordinal.
- `outcome` *string or null* — Terminal outcome when set.
- `errorMessage` *string or null* — Operator-facing error.
- `runAfter` *string*, **required** — RFC 3339 earliest claim time.
- `updatedAt` *string*, **required** — RFC 3339 last update.
- `cancelRequested` *boolean*, **required** — Cooperative cancel flag.
- `resourceClass` *string*, **required** — Concurrency class (`network` today).
- `resumePending` *boolean*, **required** — Next claim should resume rather than increment attempt.
- `hasCheckpoint` *boolean*, **required** — True when a suspend checkpoint is stored.
- `queueName` *string*, **required** — Queue of the parent event.

## EventQueueStatus

Outbox delivery counts for `GET /api/status`.

- `pending` *integer (int64)*, **required** — Pending deliveries without a suspend checkpoint.
- `running` *integer (int64)*, **required** — Currently claimed deliveries.
- `suspended` *integer (int64)*, **required** — Pending deliveries parked with a checkpoint.
- `dead_letter` *integer (int64)*, **required** — Dead-lettered deliveries.
- `acked` *integer (int64)*, **required** — Successfully acked deliveries.
- `oldest_pending_age_secs` *integer (int64) or null* — Age in seconds of the oldest pending delivery, when any exist.
- `retries_total` *integer (int64)*, **required** — Durable retry outcomes (including reclaim-as-retry).
- `suspensions_total` *integer (int64)*, **required** — Durable suspend outcomes.
- `dead_letters_total` *integer (int64)*, **required** — Durable transitions into `dead_letter`.
- `dispatch_latency_ms_avg` *integer (int64) or null* — Average first-dispatch latency in milliseconds, when sampled.
- `handler_latency_ms_avg` *integer (int64) or null* — Average `onEvent` handler duration in milliseconds, when sampled.

## HealthResponse

Liveness payload for unauthenticated `GET /health`.

- `status` *string*, **required** — Always `ok` when the process can serve HTTP.

## ImpersonateRequest

JSON body naming the first-party user to impersonate.

- `user_id` *integer (int64)*, **required** — Target `users.id` to impersonate.

## InspectBody

`POST /api/config/join/inspect` body.

- `address` *string*, **required** — The library's address (`https://owner.example`, `owner.lan:8443`).

## IntegrationLoginBody

JSON body for `POST /login/integration` (provider credential login).

- `provider` *string*, **required** — Integration plugin id (must be enabled in config).
- `username` *string*, **required** — Provider login name forwarded to `authenticate_user`.
- `password` *string*, **required** — Provider password; never logged.

## IntegrationScanRequest

Optional body for `POST /integrations/{id}/scan`.

- `force` *boolean or null* — When true, the integration rescans even if it considers itself current.

## InviteBody

`POST /api/config/join/invite` body.

- `link` *string*, **required** — The whole branch invite (`https://<library>/enroll#bcn1...`).

## JobInfo

Snapshot of one queued or running control-plane job.

- `id` *string*, **required** — Opaque job id returned to the client when the job is accepted.
- `kind` *[`JobKind`](#jobkind)*, **required** — What the job does.
- `status` *[`JobState`](#jobstate)*, **required** — Where the job is in its lifecycle.
- `detail` *string or null* — Optional operator-facing progress or error text.
- `progress` *string or null* — Human-readable progress string while the job is running.
- `attempt_count` *integer (int64)* — Number of times a worker has claimed this row.
- `max_attempts` *integer (int64)* — Maximum claims before a failure is terminal.
- `error_kind` *string or null* — Structured error kind when failed or cancelled.
- `created_at` *string or null* — RFC 3339 timestamp when the row was inserted.
- `finished_at` *string or null* — RFC 3339 timestamp when the row reached a terminal state.

## JobKind

Kind of durable daemon work stored in the `jobs` table.

Type: one of `scan`, `acquire`, `listen_sync`, `integration_scan`, `plugin_copy`, `invalid`

## JobState

Lifecycle state of a durable job row.

Type: one of `pending`, `running`, `succeeded`, `failed`, `cancelled`

## LeaveBody

`POST /api/config/join/leave` body.

- `cluster_id` *string*, **required** — The library (owner cluster id) to leave.

## LoginBegin

Login-name or email used to start a discoverable-less passkey login.

- `login` *string*, **required** — Login name or email looked up before issuing an authentication challenge.

## LoginRequest

JSON body for `POST /api/auth/login` (operator token).

- `token` *string*, **required** — Operator token from the client; compared in constant time and never persisted.

## LoginVerify

TOTP code plus the challenge id from password login.

- `challenge_id` *string*, **required** — Server-issued challenge id (5-minute TTL, single use).
- `code` *string*, **required** — Six-digit authenticator code.

## MeResponse

JSON body for `GET /me`.

- `provider` *string*, **required** — Identity provider id (`local` or an integration plugin id).
- `external_user_id` *string*, **required** — Provider-scoped user id stored on the portal identity row.
- `label` *string or null* — Optional display label from the identity row.

## MfaPolicyPut

PUT body for the host-wide second-factor policy.

- `require_second_factor` *boolean*, **required** — When true, password login requires TOTP or the user must use a passkey.
- `current_password` *string or null* — Current password for Owner step-up (operators omit this).

## OAuthEnrollRequest

`POST /api/node/v1/enroll/oauth` body. Keys are base64url without
padding.

- `code` *string*, **required** — The authorization code from the callback.
- `code_verifier` *string*, **required** — The PKCE verifier.
- `redirect_uri` *string*, **required** — The redirect URI the authorization was requested with.
- `sign_pk` *string*, **required** — Ed25519 public signing key.
- `box_pk` *string*, **required** — X25519 public box key.
- `branch_name` *string* — The name this branch joins as (the node's label).
- `software_version` *string* — The joining install's Bookclerk version.
- `connection` *[`ConnectionType`](#connectiontype)* — The connection type the code was approved for. It must match the one bound to the code at consent; left out for a branch.

## OidcClientWrite

Body for creating or updating a Bookclerk-as-IdP client.

- `client_id` *string or null* — Public client_id (create only; path wins on update).
- `name` *string or null* — Operator-facing display name.
- `redirect_uris` *array of string* — Allowed OAuth redirect URIs.
- `confidential` *boolean* — When true, mint a client secret (confidential). When false, public PKCE.
- `issue_refresh_token` *boolean* — When true, token responses include a refresh token.
- `allowed_scopes` *array of string* — Scopes this client may request (`openid`, `profile`, `email`).
- `enabled` *boolean* — When true, authorize and token endpoints accept this client.
- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

## OidcConfigPut

Operator PUT body that replaces the broker table (secrets omitted or replaced).

- `enabled` *boolean*, **required** — Whether the broker should accept authorize starts after this write.
- `allowed_email_domains` *array of string* — Replacement broker-wide email domain allowlist.
- `providers` *array of [`OidcProviderPut`](#oidcproviderput)* — Full replacement list of IdPs (omitted secrets keep the previous generation).
- `public_origin` *string or null* — `integrations.public_origin`. Omit to leave unchanged; empty string clears.
- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

## OidcConfigResponse

Operator-facing OIDC broker snapshot; secret material is never included.

- `enabled` *boolean*, **required** — Whether the broker accepts new authorize starts.
- `allowed_email_domains` *array of string*, **required** — Broker-wide email domain allowlist applied before per-provider rules.
- `public_origin` *string or null* — Configured `integrations.public_origin` (issuer / callback / invite base).
- `issuer_url` *string*, **required** — Effective issuer origin (`public_origin`, else this request, else localhost).
- `detected_origin` *string*, **required** — Origin detected from this request (loopback IPs rewritten to `localhost`).
- `discovery_url` *string*, **required** — Absolute OpenID discovery URL (`/.well-known/openid-configuration`).
- `authorization_endpoint` *string*, **required** — Absolute authorization endpoint (`/oidc/authorize`) for Bookclerk-as-IdP.
- `token_endpoint` *string*, **required** — Absolute token endpoint (`/oidc/token`) for Bookclerk-as-IdP.
- `userinfo_endpoint` *string*, **required** — Absolute userinfo endpoint (`/oidc/userinfo`) for Bookclerk-as-IdP.
- `revocation_endpoint` *string*, **required** — Absolute revocation endpoint (`/oidc/revoke`) for Bookclerk-as-IdP.
- `callback_url` *string*, **required** — Absolute `/api/auth/oidc/callback` derived from the effective issuer origin.
- `providers` *array of [`OidcProviderView`](#oidcproviderview)*, **required** — Configured IdPs with presence flags instead of secret material.

## OidcProviderPut

One IdP in a PUT: omit secret fields to keep, empty string to clear.

- `id` *string*, **required** — Stable provider id; also the logical name of the sealed client-secret row.
- `name` *string* — Operator-facing display name stored in config.toml.
- `preset` *string or null* — Built-in social preset; empty or omitted means a generic OpenID issuer.
- `issuer` *string or null* — OpenID issuer URL required when `preset` is unset.
- `client_id` *string*, **required** — OAuth client id registered at the IdP.
- `client_secret` *string or null* — Omit to keep; empty string to clear; non-empty to store.
- `scopes` *array of string* — OAuth scopes; empty is replaced with the provider default set.
- `provision` *string* — JIT, mapped-role, allowlist, or invite-only policy for first login.
- `default_role` *string* — Fallback portal role when no mapped claim matches.
- `role_claim` *string* — Claim name used for role mapping (defaults to `groups`).
- `role_map` *map of string to string* — Claim value to portal role mapping applied at provision time.
- `link_by_email` *boolean* — When true, verified email may attach this IdP to an existing user.
- `allowed_email_domains` *array of string* — Per-provider email domain allowlist.
- `allowed_emails` *array of string* — Exact emails permitted under allowlist provision.
- `allowed_subjects` *array of string* — Exact IdP subject ids permitted under allowlist provision.
- `apple_team_id` *string or null* — Apple Developer team id for client-secret JWT minting.
- `apple_key_id` *string or null* — Apple Sign In key id paired with the sealed private key.
- `apple_private_key` *string or null* — Omit to keep; empty string to clear; non-empty to store.

## OidcProviderView

One IdP as shown to an operator (no client secret or Apple PEM).

- `id` *string*, **required** — Stable provider id used in authorize URLs and sealed-row names.
- `name` *string*, **required** — Operator-facing display name.
- `preset` *string or null* — Built-in social preset (`google`, `github`, `apple`, `discord`) when not a generic issuer.
- `issuer` *string or null* — OpenID issuer URL for discovery when no social preset applies.
- `client_id` *string*, **required** — OAuth client id registered at the IdP.
- `scopes` *array of string*, **required** — OAuth scopes requested at authorize.
- `provision` *string*, **required** — How a first login becomes a portal user (JIT, mapped role, allowlist, invite-only).
- `default_role` *string*, **required** — Portal role used when no mapped claim matches (`member` by default).
- `role_claim` *string*, **required** — Claim name read from id_token/userinfo for role mapping (`groups` by default).
- `role_map` *map of string to string*, **required** — IdP group/role claim values mapped onto Bookclerk portal roles.
- `link_by_email` *boolean*, **required** — When true, a verified email may attach this IdP to an existing user.
- `allowed_email_domains` *array of string*, **required** — Per-provider email domain allowlist (in addition to the broker-wide list).
- `allowed_emails` *array of string*, **required** — Exact emails permitted under allowlist provision.
- `allowed_subjects` *array of string*, **required** — Exact IdP subject ids permitted under allowlist provision.
- `apple_team_id` *string or null* — Apple Developer team id used to mint the client-secret JWT.
- `apple_key_id` *string or null* — Apple Sign In key id paired with the sealed private key.
- `has_client_secret` *boolean*, **required** — Whether a client secret exists in env, TOML, or the sealed store (value never returned).
- `has_apple_private_key` *boolean*, **required** — Whether an Apple PEM exists in env, TOML, or the sealed store (PEM never returned).
- `secret_source` *string*, **required** — Where the client secret was found: `env`, `config`, `store`, or `none`.

## PasskeyReauthBody

Optional current password for step-up before register or delete.

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

## PasswordLoginBody

JSON body for password-store connect (`email` + `password`).

- `email` *string*, **required** — Store account email / username.
- `password` *string*, **required** — Store account password; never logged.

## PasswordLoginRequest

JSON body for local password login (login name or email plus password).

- `login` *string*, **required** — Login name or email.
- `password` *string*, **required** — Password verified against the stored hash; mismatch fails closed and counts toward lockout.

## PatchPreferencesBody

Partial preferences update; omitted fields keep their stored values.

- `default_view` *string or null* — Replacement landing view when set.
- `disabled_shelves` *array of string or null* — Replacement hidden-shelf list when set.
- `discover_sort` *string or null* — Replacement Discover sort key when set.
- `discover_sort_dir` *string or null* — Replacement Discover sort direction when set.
- `discover_language` *string or null* — Present + value sets language; present + `null` clears to browser default; omitted leaves unchanged.
- `discover_excluded_sources` *array of string or null* — Replacement excluded-store list when set.
- `theme` *string or null* — Replacement appearance preference when set (`system`, `light`, `dark`).

## PatchProfileRequest

JSON body for `PATCH /api/auth/profile`.

- `display_name` *string or null* — Replacement display name; empty or whitespace clears the field.
- `email` *string or null* — Replacement email; empty or whitespace clears the field.
- `avatar_source` *string or null* — Picture choice (`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`).

## PatchSettingsRequest

Body for `PATCH /api/settings`.

- `settings` *array of [`SettingsUpdate`](#settingsupdate)*, **required** — Key/value pairs to validate, persist, and reload.
- `expected_output_revision` *integer (int64) or null* — `core.output` document revision the client read (`output_revision` in `GET /api/settings`). When set and stale, an `output.*` update is rejected with 409.

## PatchUserRequest

Partial profile, role, or status patch for a first-party user.

- `role` *string or null* — Replacement role (`owner` / `administrator` / `member`) when present.
- `status` *string or null* — Replacement status (`active` / `disabled`) when present.
- `display_name` *string or null* — Replacement display name when present.
- `login_name` *string or null* — Replacement local login name when present.
- `email` *string or null* — Replacement email when present.

## PlacementView

Scheduler placement of one instance: the operator's intent and the last
pass's result.

- `replicas` *integer or `all`*, **required** — A replica count (0 stops every replica) or `"all"`.
- `constraints` *array of string*, **required** — Label constraints every chosen node must satisfy.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision of the intent.
- `placed` *integer (int32)*, **required** — Replicas the last scheduler pass placed. At least 0.
- `detail` *string*, **required** — Why fewer than requested are placed; empty when satisfied.
- `scheduled_at` *string*, **required** — RFC 3339 time the scheduler last changed the status.
- `updated_at` *string*, **required** — RFC 3339 time of the last intent write.

## PluginConsentApproveRequest

Body for `POST /api/plugins/{id}/consent`.

- `approve` *boolean*, **required** — Must be true; a false value is rejected with 400.
- `grant` *[`PluginGrantOverride`](#plugingrantoverride) or null* — Optional operator overrides applied on top of the manifest request.

## PluginConsentBrand

Optional branding shown on the plugin consent dialog.

- `name` *string*, **required** — Display name from the portal brand or plugin manifest.
- `bg` *string or null* — Background CSS color from the portal brand, when known.
- `fg` *string or null* — Foreground CSS color from the portal brand, when known.
- `accent` *string or null* — Accent CSS color from the portal brand, when known.
- `logo` *string or null* — Logo URL or host-served embed path.

## PluginConsentLimits

Host-capped resource defaults shown on the consent dialog.

- `cpu_ms` *integer (int32)*, **required** — Requested workerd CPU budget in milliseconds. At least 0.
- `subrequests` *integer (int32)*, **required** — Requested workerd subrequest budget. At least 0.
- `max_cpu_ms` *integer (int32)*, **required** — Host hard cap for workerd CPU milliseconds. At least 0.
- `max_subrequests` *integer (int32)*, **required** — Host hard cap for workerd subrequests. At least 0.
- `disk_mib` *integer (int32)*, **required** — Default plugin state disk budget in mebibytes. At least 0.
- `max_disk_mib` *integer (int32)*, **required** — Host hard cap for plugin state disk in mebibytes. At least 0.
- `memory_mib` *integer (int32)*, **required** — Default jail memory in mebibytes. At least 0.
- `max_memory_mib` *integer (int32)*, **required** — Host hard cap for jail memory in mebibytes. At least 0.
- `cpu_cores` *number (double)*, **required** — Manifest / default jail CPU in cores (2 d.p.).
- `max_cpu_cores` *number (double)*, **required** — Host max jail CPU in cores (2 d.p.).
- `jail_cpu_cores` *number (double) or null* — Optional `[plugins.jail]` ceiling in cores.
- `extra_processes` *integer (int32)*, **required** — Default extra process/thread budget (native consent). At least 0.
- `max_extra_processes` *integer (int32)*, **required** — Host hard cap for extra process budget. At least 0.
- `known_bindings` *array of string*, **required** — Suggested / default host bindings operators may grant.

## PluginConsentResponse

Consent dialog payload for one plugin.

- `plugin_id` *string*, **required** — Plugin id whose grant is being reviewed.
- `plugin_key` *string*, **required** — Canonical PluginKey for this install (empty when unknown).
- `provenance` *string*, **required** — Host-evaluated provenance label.
- `runtime` *string*, **required** — Guest runtime (`native` or `workerd`) — drives which controls are enforceable.
- `request` *[`PluginGrantView`](#plugingrantview)*, **required** — Manifest-derived grant the plugin is asking for.
- `covered` *boolean*, **required** — True when an existing on-disk grant already satisfies the request.
- `summary` *array of string*, **required** — Operator-facing bullet list of requested capabilities.
- `existing` *[`PluginGrantView`](#plugingrantview) or null* — Previously approved grant, when one is stored.
- `brand` *[`PluginConsentBrand`](#pluginconsentbrand) or null* — Optional portal or manifest branding for the dialog.
- `limits` *[`PluginConsentLimits`](#pluginconsentlimits)*, **required** — Host-capped resource defaults (workerd budgets + shared disk). Always set.

## PluginDeploymentView

This node's view of one deployment row, with its latest observation.

- `deployment_id` *string*, **required** — Deployment row id.
- `node_id` *string*, **required** — Node the row targets.
- `desired` *[`DesiredState`](#desiredstate)*, **required** — What the row asks the node to do: `present` / `absent` from an operator, `placed` / `unplaced` from the scheduler.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision of the desired row.
- `status` *string or null* — Latest observation status from that node, when it reported one.
- `detail` *string or null* — Observation detail (error text, redacted).
- `applied_config_revision` *integer (int64) or null* — Document revision the running guest applied.
- `observed_at` *string or null* — RFC 3339 time of the observation.

## PluginGrantOverride

Operator-supplied grant fields that replace matching request values.

- `networkMode` *string or null* — Override egress mode when set.
- `domains` *array of string or null* — Override approved hostnames when set.
- `tcp` *array of [`TcpGrantView`](#tcpgrantview) or null* — Override raw TCP grants when set.
- `addressCidrs` *array of string or null* — Override address-space CIDRs when set.
- `allowUndeclaredPublicRedirects` *boolean or null* — Override undeclared-public-redirect permission when set.
- `bindings` *array of string or null* — Override approved host bindings when set.
- `compatibilityFlags` *array of string or null* — Override compatibility flags when set.
- `cpuMs` *integer (int32) or null* — Override workerd CPU milliseconds when set. At least 0.
- `subrequests` *integer (int32) or null* — Override workerd subrequest budget when set. At least 0.
- `diskMib` *integer (int32) or null* — Override disk budget in mebibytes when set. At least 0.
- `memoryMib` *integer (int32) or null* — Override jail memory in mebibytes when set. At least 0.
- `cpuCores` *number (double) or null* — Jail CPU in cores (2 d.p.); converted to Spec percent server-side.
- `extraProcesses` *integer (int32) or null* — Extra process/thread budget (native); ignored for workerd. At least 0.

## PluginGrantView

Operator-facing grant JSON: CPU as cores (not Spec percent).

- `pluginKey` *string*, **required** — Canonical provenance-qualified PluginKey.
- `pluginId` *string*, **required** — Plugin id this grant applies to.
- `entrypoints` *array of string*, **required** — Exported entrypoints the operator approved (`storefront`, `cli`, …).
- `producers` *array of string*, **required** — Event types the operator approved the plugin to publish.
- `consumers` *array of object*, **required** — Approved event-consumer subscriptions.
- `jobs` *array of string*, **required** — Approved job trigger types.
- `networkMode` *string*, **required** — Approved egress mode (`deny`, `allowlist`, and similar).
- `domains` *array of string*, **required** — Hostnames the operator approved for egress.
- `manifestDomains` *array of string*, **required** — Destinations requested by the current manifest.
- `operatorAddedDomains` *array of string*, **required** — Operator-added destinations beyond the manifest.
- `operatorDeniedDomains` *array of string*, **required** — Operator-denied destinations still listed by the manifest.
- `tcp` *array of [`TcpGrantView`](#tcpgrantview)*, **required** — Effective raw TCP grants.
- `operatorAddedTcp` *array of [`TcpGrantView`](#tcpgrantview)*, **required** — Operator-added TCP grants.
- `operatorDeniedTcp` *array of [`TcpGrantView`](#tcpgrantview)*, **required** — Operator-denied TCP grants still listed by the manifest.
- `addressCidrs` *array of string*, **required** — Effective address-space CIDRs.
- `operatorAddedCidrs` *array of string*, **required** — Operator-added CIDRs.
- `operatorDeniedCidrs` *array of string*, **required** — Operator-denied CIDRs still listed by the manifest.
- `allowUndeclaredPublicRedirects` *boolean*, **required** — Undeclared public redirect permission.
- `grantRevision` *string*, **required** — SHA-256 of persisted operator consent (no host overlays).
- `authorityRevision` *string*, **required** — SHA-256 of the canonical effective authority.
- `bindings` *array of string*, **required** — Host bindings the operator approved.
- `compatibilityFlags` *array of string*, **required** — Extra compatibility flags stored on the grant.
- `approvedAt` *string*, **required** — RFC 3339 timestamp when the grant was last approved.
- `cpuMs` *integer (int32) or null* — Optional workerd CPU budget in milliseconds. At least 0.
- `subrequests` *integer (int32) or null* — Optional workerd subrequest budget. At least 0.
- `diskMib` *integer (int32) or null* — Optional plugin state disk budget in mebibytes. At least 0.
- `memoryMib` *integer (int32) or null* — Optional jail memory in mebibytes. At least 0.
- `cpuCores` *number (double) or null* — Jail CPU in cores (2 d.p.); absent for workerd.
- `extraProcesses` *integer (int32) or null* — Extra process/thread budget beyond launcher overhead; native only. At least 0.

## PluginSettingChoice

One selectable value for a Settings dropdown.

- `value` *string*, **required** — Stored config value; empty string means use the plugin default.
- `label` *string*, **required** — Operator-facing choice caption.

## PluginSettingOption

One editable setting shown in the Settings UI.

- `key` *string*, **required** — Dotted `config.toml` key for this option.
- `label` *string*, **required** — Operator-facing field caption.
- `value` *string*, **required** — Current string form of the setting.
- `value_type` *string*, **required** — UI widget hint (`boolean`, `string`, or `number`).
- `choices` *array of [`PluginSettingChoice`](#pluginsettingchoice) or null* — Optional dropdown values; omitted for free-form fields.
- `secret` *boolean* — True for a `secret` setting: `value` is never the stored value.
- `secret_set` *boolean or null* — For a secret setting, whether the instance document names a sealed value.

## PluginSettingsGroup

Settings UI group for one discovered or loaded plugin.

- `id` *string*, **required** — Plugin id (`audible`, `sqlite`, and similar).
- `family` *string*, **required** — Handler family wire label (`source`, `integration`, `output`, `database`).
- `plugin_key` *string or null* — Provenance-qualified PluginKey when this row came from a discovered install.
- `provenance` *string or null* — Host-evaluated provenance (`platform_bundled`, `verified_staged`, `installed`, …).
- `logo` *string or null* — Google favicon (or portal brand) URL for Settings list rows.
- `settings` *array of [`PluginSettingOption`](#pluginsettingoption)*, **required** — Editable options for this plugin, including `.enabled`.
- `plugin_instance_id` *string or null* — Plugin instance this group edits; absent for a bootstrap database adapter.
- `display_name` *string or null* — Operator-facing instance name, when the operator set one.
- `is_default` *boolean* — True when a bare plugin key or alias routes to this instance.
- `config_revision` *integer (int64) or null* — Instance document revision for compare-and-swap writes.
- `deployment` *[`PluginDeploymentView`](#plugindeploymentview) or null* — This node's deployment of the instance, when one exists.
- `placement` *[`PlacementView`](#placementview) or null* — Scheduler placement; absent when the instance is hand-managed (the Enabled switch applies only then).
- `present_on` *array of string* — Nodes whose deployment of the instance is `present`.

## PortalErrorBody

JSON body of every portal error response.

- `error` *string*, **required** — Operator-facing error text (no structured code).

## PostNodeInvite

`POST /api/config/node-invites` body.

- `user_id` *integer (int64) or null* — User who will own the node. Defaults to the caller; the operator must name one.
- `ttl_secs` *integer (int64) or null* — Link lifetime in seconds (default 24 h, 5 min to 7 days). At least 0.
- `label` *string* — Label shown in listings.

## PostPluginInstall

`POST /api/config/plugins` body: exactly one of `coordinate` or
`package` (a directory name under `plugin-packages/`).

- `coordinate` *string or null* — Registry coordinate (`cargo:…@v`, `registry:…#name@v`).
- `package` *string or null* — Operator-placed package directory under `plugin-packages/`.
- `activate` *boolean* — Make the new version active even when another is.
- `approve` *boolean* — Record the requested consent as approved.
- `allow_unverified_publisher` *boolean* — Accept a publisher without independent authenticity.

## PreferencesResponse

Normalized Discover/library preferences for the authenticated subject.

- `default_view` *string*, **required** — Landing view id (`library`, `discover`, …) after normalization.
- `disabled_shelves` *array of string*, **required** — Lowercased Discover shelf ids the user has hidden.
- `discover_sort` *string*, **required** — Discover sort key (`relevance`, `popularity`, …).
- `discover_sort_dir` *string*, **required** — Discover sort direction (`asc` or `desc`).
- `discover_language` *string or null* — `null` = use browser language on the client.
- `discover_excluded_sources` *array of string*, **required** — Lowercased store ids hidden from Discover results.
- `theme` *string*, **required** — Appearance preference (`system`, `light`, or `dark`).

## PurchaseHint

A purchase / catalog availability hint (optionally priced at view time).

- `source` *string*, **required** — Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id` *string*, **required** — Storefront-native product id (ASIN, ISBN, UUID, …).
- `title` *string or null* — Display title as shown on the storefront or library card.
- `url` *string or null* — Absolute HTTPS URL.
- `price_cents` *integer (int64) or null* — Primary / best known sell price in minor units (prefer member when dual). `0` = free.
- `currency` *string or null* — ISO 4217 currency code for price fields (`USD`, `EUR`, …).
- `price_label` *string or null* — Display string from the store (`$2.99`, `FREE`, …).
- `list_price_cents` *integer (int64) or null* — Non-member / list price in integer cents when known.
- `list_price_label` *string or null* — Human-readable list price from the storefront.
- `member_price_cents` *integer (int64) or null* — Member / credit price in integer cents when known.
- `member_price_label` *string or null* — Human-readable member price from the storefront.

## PurchaseHintsBatchBody

Body for batched purchase-hint lookups (max 24 queries).

- `queries` *array of [`PurchaseHintsQuery`](#purchasehintsquery)* — Per-title purchase-hint queries in this batch.

## PurchaseHintsBatchResponse

Parallel purchase-hint results aligned with the request queries.

- `results` *array of [`PurchaseHintsResponse`](#purchasehintsresponse)*, **required** — One purchase-hint response per query, in request order.

## PurchaseHintsQuery

Inputs for view-time catalog + pricing lookup.

- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `candidate_source` *string or null* — Storefront id of the edition that produced this card.
- `candidate_product_id` *string or null* — Storefront product id of the edition that produced this card.
- `store_editions` *array of [`StoreEdition`](#storeedition)* — Known storefront editions already on the recommendation card.
- `region` *string or null* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `preferred_sources` *array of string* — Storefronts the caller has linked accounts for. Treated as **member** pricing when picking `best`; other stores are compared at list / non-member price. Every catalog match is still returned in `hints`.

## PurchaseHintsResponse

Priced catalog matches for one title, sorted best-first.

- `hints` *array of [`PurchaseHint`](#purchasehint)*, **required** — Resolved purchase hints for this query.
- `best` *[`PurchaseHint`](#purchasehint) or null* — Lowest-priced hint (or first catalog hit when no prices resolved).

## PutActiveVersion

`PUT /api/config/plugins/{key}/active` body.

- `version` *string*, **required** — Installed version to activate.

## PutBranches

`PUT /api/config/branches` body.

- `enrollment` *boolean*, **required** — "Let new branches join" (`core.host_enrollment.enabled`).

## PutEventsDomainRequest

JSON body for `PUT /api/config/domains/core.events`.

- `expected_revision` *integer (int64)*, **required** — Revision the caller read. A mismatch is a conflict.
- `retention_days` *integer (int64)*, **required** — Replacement `[events]` document. At least 0.
- `dead_letter_retention_days` *integer (int64)*, **required** — Replacement dead-letter retention. At least 0.
- `concurrency` *integer (int32)*, **required** — Replacement concurrency cap. At least 0.

## PutHostEnrollment

`PUT /api/config/host-enrollment` body.

- `enabled` *boolean*, **required** — Whether host nodes may enroll into this cluster.

## PutNodeLabels

`PUT /api/config/nodes/{node_id}/labels` body.

- `labels` *array of string*, **required** — The node's complete operator label set.

## PutNodeOwner

`PUT /api/config/nodes/{node_id}/owner` body.

- `user_id` *integer (int64) or null* — User who owns the node, or `null` to clear.

## PutNodeRuntime

`PUT /api/config/nodes/{node_id}/runtime` body: any subset of the
node's runtime document.

- `label` *string or null* — Operator label.
- `cpu_budget_percent` *integer (int32) or null* — Share of detected CPU the scheduler may commit (1..=100). At least 0.
- `memory_budget_percent` *integer (int32) or null* — Share of detected memory the scheduler may commit (1..=100). At least 0.
- `schedulable` *boolean or null* — False drains the node.

## PutOutputDomainRequest

JSON body for `PUT /api/config/domains/core.output`.

- `expected_revision` *integer (int64)*, **required** — Revision the caller read. A mismatch is a conflict.
- `document` *object*, **required** — Complete replacement document (schema version 1).

## PutPluginDeployment

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}` body.

- `desired` *[`DesiredState`](#desiredstate)*, **required** — What the node should do: `present` or `absent` (the scheduler's `placed` / `unplaced` are accepted too).
- `expected_revision` *integer (int64) or null* — Revision the caller last observed; required when the row exists and the caller wants compare-and-swap. Omitted means "replace whatever is current".

## PutPluginInstanceConfig

`PUT /api/config/plugin-instances/{id}/config` body.

- `expected_revision` *integer (int64)*, **required** — Revision the caller last observed.
- `settings` *object* — Scalar settings. Secret values are not accepted here.
- `secret_refs` *array of object* — Secret refs to keep. Omitted refs are dropped from the document; the ciphertext stays sealed until the secret is overwritten.
- `secrets` *map of string to string* — Secret values to seal, keyed by the declared `secret` setting. Each is sealed into `encrypted_secrets` and referenced from the document; the plaintext is never stored in the document or echoed back.

## PutPluginPlacement

`PUT /api/config/plugin-instances/{id}/placement` body.

- `replicas` *integer or `all`*, **required** — A replica count (0 stops every replica) or `"all"`.
- `constraints` *array of string* — Label constraints (`name`, `name=value`, `!name`).
- `expected_revision` *integer (int64) or null* — Current revision to replace; omit (or 0) to create.

## QueueDetail

Operator JSON for `GET /api/queues/{name}`.

Includes every field of [`QueueInfo`](#queueinfo).

- `messages` *array of [`DomainEventInfo`](#domaineventinfo)*, **required** — Recent messages on the queue, newest first.
- `deliveries` *array of [`EventDeliveryInfo`](#eventdeliveryinfo)*, **required** — Recent deliveries on the queue, newest first.

## QueueInfo

Operator JSON for one queue on `GET /api/queues`.

- `name` *string*, **required** — Queue name.
- `core` *boolean*, **required** — True for the domain-event bus (`core.events`).
- `pending` *integer (int64)*, **required** — Deliveries waiting to be claimed (including suspended rows).
- `running` *integer (int64)*, **required** — Deliveries currently claimed by a worker.
- `acked` *integer (int64)*, **required** — Deliveries acked.
- `deadLetter` *integer (int64)*, **required** — Deliveries parked for operator review.
- `consumers` *array of string*, **required** — Live catalog consumers (plugin keys or instance ids) of this queue.

## QueueWisher

Compact wisher shown on the global queue (avatar + label).

- `user_id` *integer (int64) or null* — First-party `users.id` when the portal identity is linked.
- `identity_id` *integer (int64) or null* — Portal identity that created the wish (dedupe for unlinked identities).
- `display_name` *string or null* — Display name, identity label, or `"Operator"`.
- `login_name` *string or null* — Local login name when the first-party user has one.
- `operator` *boolean* — True when the wish came from the operator token (no portal user).
- `has_avatar` *boolean* — True when an uploaded avatar file exists (filled by the daemon).
- `avatar_source` *string or null* — Explicit picture choice when the user has set one.
- `gravatar_hash` *string or null* — SHA-256 hex of the contact email for Gravatar.
- `picture_url` *string or null* — HTTPS picture from the wishing identity, when any.

## RankedQueueEntry

Global wishlist queue entry ranked by overall/operator recommend taste +
heavy wish-count weight (shared order; not per-viewer personalized).

- `work_key` *string*, **required** — Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
- `wish_count` *integer (int64)*, **required** — Number of distinct users who wishlisted this title.
- `wishers` *array of [`QueueWisher`](#queuewisher)* — Distinct people who have this title on an open wishlist.
- `sample_uuids` *array of string*, **required** — Sample Bookclerk user UUIDs who requested this title (privacy-capped).
- `first_requested_at` *string (date-time)*, **required** — UTC timestamp of the earliest wishlist request for this work.
- `last_requested_at` *string (date-time)*, **required** — UTC timestamp of the latest wishlist request for this work.
- `score` *number (double)*, **required** — Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`]).
- `taste_score` *number (double)*, **required** — Local taste / embedding component before the wish-count boost.
- `reasons` *array of string*, **required** — Short human-readable explanations for why this item was ranked here.

## Recommendation

One ranked recommendation (typically an unowned storefront title).

- `work_id` *string or null* — Library `works.id` UUID when the recommendation is tied to an owned work.
- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `narrators` *string or null* — Comma-separated narrator names when known.
- `series` *string or null* — Series name when the title belongs to a named series.
- `series_index` *string or null* — Position within the series (e.g. `1`, `1.5`) when known.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `score` *number (double)*, **required** — Ranking score (higher is better); units are algorithm-specific.
- `reasons` *array of string*, **required** — Short human-readable explanations for why this item was ranked here.
- `purchase_hints` *array of [`PurchaseHint`](#purchasehint)*, **required** — Buy / open-in-store links with optional member vs list pricing.
- `from_request` *boolean*, **required** — True when sourced from an open title request rather than storefront discovery.
- `request_uuid` *string or null* — Wishlist / title-request UUID when this card came from a request.
- `candidate_source` *string or null* — Storefront that proposed this title (`audible`, `libro`, …).
- `candidate_product_id` *string or null* — Storefront product id of the edition that produced this card.
- `store_editions` *array of [`StoreEdition`](#storeedition)* — All known storefront editions (for multi-store purchase links).
- `seed_categories` *string or null* — Categories/subjects copied from the taste seed that produced this hit.
- `categories` *array of string* — Stable shelf-kind tags for prefs (`finish_series`, `author`, `chirp_deals`, …). Dynamic shelf titles stay in `build_discover_feed`; membership uses these.
- `work_key` *string* — Stable bibliographic key for wishlist / merge (`isbn:` / `asin:` / `soft:`…).
- `cover_url` *string or null* — Cover art URL when known from a storefront catalog hit or wishlist row.
- `subtitle` *string or null* — Optional bibliographic extras from storefront catalog / Audnexus.
- `description` *string or null* — Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher` *string or null* — Publisher imprint as reported by the catalog.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `published_at` *string or null* — Publication date string from the catalog (ISO or storefront format).
- `genres` *string or null* — Candidate genres (not taste-seed `seed_categories`).
- `language` *string or null* — BCP-47 / storefront language code (`en`, `de`, …) when known.
- `wishers` *array of [`QueueWisher`](#queuewisher)* — Distinct people who have this title on an open wishlist.
- `wish_count` *integer (int64)* — Number of distinct users who wishlisted this title (may exceed `wishers.len()`).

## RedeemBody

JSON body for `POST /redeem` (claim ticket plus retry nonce).

- `ticket` *string*, **required** — Raw claim-ticket token to redeem into a portal session.
- `nonce` *string*, **required** — Browser-generated one-time nonce persisted across HTTP retries.
- `password` *string or null* — Required when the linked local user has no password (invite / reset).

## RemoveBranchTitles

`POST /api/config/branches/titles/remove` body.

- `node_id` *string or null* — Only this branch's entries; every branch's when absent.

## RequestStatus

Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).

Type: one of `open`, `cancelled`

## RevokeBody

`POST /api/config/cluster-consents/revoke` body.

- `cluster_id` *string*, **required** — Owner cluster id.
- `plugin_key` *string*, **required** — Canonical plugin key.

## RevokeForm

Form body for `POST /oidc/revoke` (RFC 7009-style refresh-token revoke).

- `token` *string*, **required** — Token to revoke; treated as a refresh token (hashed before lookup).
- `token_type_hint` *string or null* — Optional hint ignored; this server only stores refresh tokens.

## RotateSecretBody

Optional password for rotating a confidential client's secret.

- `current_password` *string or null* — Required for a non-elevated Owner whose portal session is older than 15 minutes.

## ScanPlanResponse

Result of a planned scan or acquire: [`ActionResponse`] plus every job id.

- `ok` *boolean*, **required** — True when at least one job was accepted.
- `message` *string*, **required** — Operator-facing status text.
- `job_id` *string*, **required** — First accepted (or, when none was, already queued) job id.
- `job_ids` *array of string*, **required** — Every accepted or already-queued job id, in plan order.
- `error` *string or null* — Machine slug when nothing was accepted (`conflict`, `queue_full`, `nothing_to_scan`, `nothing_to_acquire`).

## ScanRequest

Optional body for `POST /api/library/scan`.

- `account` *string or null* — When set, scan this account even if `scan_enabled` is false.
- `plugin_instance_id` *string or null* — When set, scan only this deployed plugin instance.

## SetPasswordRequest

JSON body for setting a password (self or provisioner target).

- `password` *string*, **required** — New password; hashed before persist. Empty is rejected.
- `current_password` *string or null* — Required when the caller already has a password (not first-time invite setup).
- `user_id` *integer (int64) or null* — Target `users.id` when a provisioner sets another user's password.

## SettingsResponse

Settings page payload: file values, effective runtime, and plugin groups.

- `settings` *map of string to string*, **required** — Values as written in config.toml (and env overlays), plus `output.*` from the `core.output` database document.
- `effective` *map of string to string*, **required** — Runtime-effective values after the last successful reload (auth, plugins).
- `plugins` *array of [`PluginSettingsGroup`](#pluginsettingsgroup)*, **required** — Per-plugin setting groups rendered by the Settings UI.
- `host_cpu_cores_max` *number (double)*, **required** — Host max jail CPU in cores (2 d.p.; equals logical CPU count).
- `jail_cpu_cores` *number (double) or null* — Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`).
- `output_revision` *integer (int64) or null* — Revision of the `core.output` document the `output.*` values in `settings` were read from; send it back as `expected_output_revision`. `effective["output.revision"]` is the revision this host has applied, which lags the document until the host's next wakeup.

## SettingsUpdate

One `config.toml` key/value pair from a settings PATCH.

- `key` *string*, **required** — Dotted config key; rejected unless it passes the settings allowlist.
- `value` *string*, **required** — Raw string value, normalized before it is written.

## SharingBody

`POST /api/config/join/sharing` body.

- `cluster_id` *string*, **required** — The library (owner cluster id).
- `share_books` *boolean*, **required** — "Share this branch's books with *library*".
- `include_branches` *boolean*, **required** — "Include books from my own branches".

## ShelfKindInfo

A shelf kind the operator can ignore (all offered by default).

- `id` *string*, **required** — Stable kind id (`finish_series`, `author`, `genre`, `from_store`, …).
- `label` *string*, **required** — Operator-visible label for this account or item.

## SourceConfigOptionInfo

One storefront config option (key, label, allowed values).

- `key` *string*, **required** — Config table key (for example `container`).
- `label` *string*, **required** — Operator-facing option name.
- `values` *array of [`ConfigOptionValueInfo`](#configoptionvalueinfo)*, **required** — Allowed values the SPA may offer for this option.

## SourceInfo

One enabled storefront as shown on the portal Accounts page.

- `id` *string*, **required** — Source plugin id (`audible`, `libro`, …).
- `name` *string*, **required** — Operator-facing storefront name.
- `auth` *string*, **required** — Portal auth mode (`password` or `oauth`).
- `config_options` *array of [`SourceConfigOptionInfo`](#sourceconfigoptioninfo)* — Store-specific knobs the SPA can present (omitted when empty).
- `brand` *[`BrandInfo`](#brandinfo)*, **required** — Storefront colors and logo href for the Accounts UI.

## SourcesResponse

JSON body for `GET /sources`.

- `sources` *array of [`SourceInfo`](#sourceinfo)*, **required** — Enabled storefronts the SPA may offer for connect / login.

## StartBody

`POST /api/config/join/start` body.

- `address` *string*, **required** — The library's address.
- `fingerprint` *string or null* — The fingerprint the operator confirmed (self-signed libraries).
- `branch_name` *string or null* — The name to join as (default: this machine's name).
- `clerk` *boolean* — Offer to join as a clerk (an owner node that shares the library's database): only an empty install may, and its data is replaced.

## StatusResponse

Library and daemon snapshot for `GET /api/status`.

- `accounts` *integer*, **required** — Count of configured store accounts. At least 0.
- `books` *integer*, **required** — Total title rows in the library. At least 0.
- `acquired` *integer (int64)*, **required** — Titles whose `acquire_status` is `acquired`.
- `pending` *integer (int64)*, **required** — Titles still needing acquire (`not_acquired`).
- `error` *integer (int64)*, **required** — Titles stuck in `error` after a failed acquire.
- `in_progress` *integer (int64)*, **required** — Titles currently `queued` or `downloading`.
- `listen` *string*, **required** — Comma-joined `daemon.listen` addresses from live config.
- `storage_backend` *string*, **required** — Comma-joined enabled destination plugin ids, or `none`.
- `auth_enabled` *boolean*, **required** — Whether the live auth middleware requires operator credentials.
- `events` *[`EventQueueStatus`](#eventqueuestatus)*, **required** — Durable domain-event delivery-queue counters.

## StepUp

A fresh second factor for a clerk consent. `Debug` prints no code.

Type: object

## StoreEdition

One storefront edition of a work (for multi-store purchase links).

- `source` *string*, **required** — Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id` *string*, **required** — Storefront-native product id (ASIN, ISBN, UUID, …).

## TcpGrantView

Operator-facing TCP grant (`host` + `ports`).

- `host` *string*, **required** — Hostname or `*.` wildcard.
- `ports` *array of integer (int32)*, **required** — Destination ports.

## TitleMeta

Bibliographic fields suitable for an Audible-style detail panel.

- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `title` *string or null* — Display title as shown on the storefront or library card.
- `subtitle` *string or null* — Optional subtitle when the catalog distinguishes it from the title.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `narrators` *string or null* — Comma-separated narrator names when known.
- `series` *string or null* — Series name when the title belongs to a named series.
- `series_index` *string or null* — Position within the series (e.g. `1`, `1.5`) when known.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
- `description` *string or null* — Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher` *string or null* — Publisher imprint as reported by the catalog.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `published_at` *string or null* — Publication date string from the catalog (ISO or storefront format).
- `categories` *string or null* — Genre / subject tags as a single delimited string when known.
- `language` *string or null* — BCP-47 / storefront language code (`en`, `de`, …) when known.
- `is_abridged` *boolean or null* — `Some(true)` abridged / `Some(false)` unabridged when the storefront said so.
- `rating_overall` *number (double) or null* — Audible community overall rating (0–5).
- `rating_performance` *number (double) or null* — Narration / performance sub-score when the storefront splits ratings.
- `rating_story` *number (double) or null* — Story / content sub-score when the storefront splits ratings.
- `rating_count` *integer (int64) or null* — Count of star ratings (Audible `num_ratings`).
- `review_count` *integer (int64) or null* — Count of written reviews (Audible `num_reviews`).
- `reviews` *array of [`TitleReview`](#titlereview)* — Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty for older clients; prefer the paginated reviews endpoint.

## TitleMetaBatchBody

Body for batched title-metadata lookups (max 24 queries).

- `queries` *array of [`TitleMetaQuery`](#titlemetaquery)* — Per-title metadata queries in this batch.

## TitleMetaBatchResponse

Title-metadata results aligned with the request; failed items are `null`.

- `results` *array of [`TitleMeta`](#titlemeta) or null*, **required** — One optional metadata payload per query; enrichment errors become `None`.

## TitleMetaQuery

Query for public title metadata (Audnexus / Audible catalog).

- `title` *string* — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `narrators` *string or null* — Comma-separated narrator names when known.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `region` *string* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.

## TitleRequestRecord

Personal wishlist row (also contributes to the shared global queue while open).

- `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
- `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
- `identity_id` *integer (int64) or null* — Foreign key to `portal_identities.id`.
- `title` *string*, **required** — Display title of the work or edition.
- `authors` *string or null* — Comma-separated or JSON author list from the storefront.
- `asin` *string or null* — Amazon ASIN when known; otherwise null.
- `isbn` *string or null* — ISBN-10/13 when known; otherwise null.
- `notes` *string or null* — Free-form operator or requester notes.
- `status` *[`RequestStatus`](#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
- `work_key` *string* — Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation.
- `work_id` *string or null* — Canonical work id this edition or request resolves to.
- `resolved_book_uuid` *string or null* — Library `books.uuid` once the wishlist item is fulfilled.
- `cover_url` *string or null* — HTTPS URL for cover art when known.
- `sources` *array of [`TitleRequestSourceRecord`](#titlerequestsourcerecord)* — Per-storefront snapshots (empty for legacy rows).
- `description` *string or null* — Blurb / synopsis text (may contain HTML).
- `subtitle` *string or null* — Optional subtitle from bibliographic metadata.
- `narrators` *string or null* — Comma-separated or JSON narrator list when present.
- `series` *string or null* — Series name when the title belongs to a series.
- `series_index` *string or null* — Position within the series (storefront string form).
- `publisher` *string or null* — Publisher name from metadata enrichment or the storefront.
- `length_minutes` *integer (int64) or null* — Runtime in whole minutes when the storefront reports it.
- `published_at` *string or null* — Publication date string from the storefront or enrichment.
- `genres` *string or null* — Merged genre / category list for display.
- `language` *string or null* — BCP-47 or storefront language code when known.
- `store_editions` *array of [`WishlistStoreEdition`](#wishliststoreedition)* — Distinct storefront editions contributing to this wishlist item.
- `purchase_hints` *array of [`WishlistPurchaseHint`](#wishlistpurchasehint)* — Purchase URLs and prices observed per storefront.
- `created_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was inserted.
- `updated_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was last modified.

## TitleRequestSourceRecord

Per-storefront catalog/pricing snapshot attached to a wishlist row.

- `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
- `title_request_id` *integer (int64)*, **required** — Foreign key to `title_requests.id`.
- `source` *string*, **required** — Content-source plugin id (`audible`, `libro`, …).
- `product_id` *string*, **required** — Storefront product id (ASIN, ISBN, UUID, …).
- `title` *string or null* — Display title of the work or edition.
- `subtitle` *string or null* — Optional subtitle from bibliographic metadata.
- `authors` *string or null* — Comma-separated or JSON author list from the storefront.
- `narrators` *string or null* — Comma-separated or JSON narrator list when present.
- `series` *string or null* — Series name when the title belongs to a series.
- `series_index` *string or null* — Position within the series (storefront string form).
- `asin` *string or null* — Amazon ASIN when known; otherwise null.
- `isbn` *string or null* — ISBN-10/13 when known; otherwise null.
- `description` *string or null* — Blurb / synopsis text (may contain HTML).
- `publisher` *string or null* — Publisher name from metadata enrichment or the storefront.
- `length_minutes` *integer (int64) or null* — Runtime in whole minutes when the storefront reports it.
- `published_at` *string or null* — Publication date string from the storefront or enrichment.
- `categories` *string or null* — Storefront category / genre path list (serialized).
- `language` *string or null* — BCP-47 or storefront language code when known.
- `cover_url` *string or null* — HTTPS URL for cover art when known.
- `url` *string or null* — Storefront product or purchase URL when known.
- `price_cents` *integer (int64) or null* — Observed price in minor currency units, when known.
- `currency` *string or null* — ISO 4217 currency code for price fields.
- `price_label` *string or null* — Storefront-formatted price string for display.
- `list_price_cents` *integer (int64) or null* — List/MSRP price in minor units, when known.
- `list_price_label` *string or null* — Storefront-formatted list price for display.
- `member_price_cents` *integer (int64) or null* — Member/subscriber price in minor units, when known.
- `member_price_label` *string or null* — Storefront-formatted member price for display.
- `observed_at` *string (date-time) or null* — RFC 3339 time when this storefront snapshot was observed.
- `created_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was inserted.
- `updated_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was last modified.

## TitleReview

One Audible customer review for the title detail panel.

- `id` *string or null* — Stable identifier for this item.
- `title` *string or null* — Display title as shown on the storefront or library card.
- `body` *string*, **required** — Full review text from the storefront.
- `author_name` *string or null* — Display name of the review author on the storefront.
- `overall_rating` *integer (int64) or null* — Overall star rating for this review when present.
- `performance_rating` *integer (int64) or null* — Narration sub-rating for this review when present.
- `story_rating` *integer (int64) or null* — Story sub-rating for this review when present.
- `submitted_at` *string or null* — When the review was submitted (storefront timestamp string).

## TitleReviewsPage

One page of title reviews for infinite scroll.

- `asin` *string*, **required** — Audible / Amazon ASIN when this edition is sold on Audible.
- `page` *integer (int32)*, **required** — 1-based page index for this reviews / list request. At least 0.
- `page_size` *integer (int32)*, **required** — Maximum number of items returned in this page. At least 0.
- `has_more` *boolean*, **required** — When true, another page is available by incrementing [`Self::page`].
- `sort_by` *string*, **required** — Remote sort key for reviews (`most_helpful`, `most_recent`, …).
- `reviews` *array of [`TitleReview`](#titlereview)*, **required** — Review rows on this page.

## TitleReviewsQuery

Query for a page of Audible customer reviews.

- `asin` *string*, **required** — Audible / Amazon ASIN when this edition is sold on Audible.
- `region` *string* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `page` *integer (int32)* — 1-based page index. At least 0.
- `page_size` *integer (int32)* — Page size (clamped server-side to 1..=20). At least 0.
- `sort_by` *string* — `MostHelpful` (default) or `MostRecent`.

## TokenForm

`application/x-www-form-urlencoded` body for `POST /oidc/token`.

- `grant_type` *string*, **required** — Grant: `authorization_code` or `refresh_token`; anything else is 400.
- `code` *string or null* — Authorization code from the redirect (`authorization_code` grant).
- `redirect_uri` *string or null* — Must match the URI bound to the consumed authorization code.
- `client_id` *string or null* — Client id; required for code exchange, optional (but checked) on refresh.
- `client_secret` *string or null* — Client secret for confidential clients (`client_secret_post`).
- `code_verifier` *string or null* — PKCE verifier; SHA-256 must match the stored challenge.
- `refresh_token` *string or null* — Refresh token to rotate (`refresh_token` grant).

## TokenResponse

Successful token-endpoint JSON (access, optional refresh, and ID tokens).

- `access_token` *string*, **required** — HS256 JWT access token bound to the User (`token_use=access`).
- `token_type` *string*, **required** — Always `Bearer` for this server.
- `expires_in` *integer (int64)*, **required** — Access-token lifetime in seconds (same as [`ACCESS_TTL_SECS`]).
- `refresh_token` *string or null* — Opaque refresh token when the client is allowed to receive one.
- `id_token` *string*, **required** — HS256 ID token; profile/email claims follow the granted scope.
- `scope` *string*, **required** — Space-separated scopes granted to this token pair.

## TotpReauthBody

Optional current password for enroll/disable step-up.

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

## UploadAnswer

Answer of the upload endpoint (every status carries one).

- `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
- `reason` *string or null* — Why, for `refused`.
- `job_completed` *boolean* — For `committed`: the job is finished.

## UploadCommitRequest

Body of `POST <UPLOADS_PATH><upload_id>/commit`.

- `finish` *boolean* — Complete the job with this commit (the job's closing upload).
- `detail` *string or null* — Outcome text recorded as the job's progress when `finish` is set.

## UserChangeConflict

Body of a `409 Conflict` from a user change that would leave the
library without an owner or administrator.

- `error` *string*, **required** — Why the change was refused: `last_owner` or `last_administrator`.

## UserChangeInvalid

Body of a `400 Bad Request` from a user change with an invalid email.

- `error` *string*, **required** — Machine slug for the rejected value.
- `message` *string*, **required** — A sentence to show the person who made the request.

## UserChangeResponse

Body of a user change that succeeded: the user as stored afterwards.

- `ok` *boolean*, **required** — Always true; the status code says the change was applied.
- `user` *[`UserSummary`](#usersummary)*, **required** — The user after the change.

## UserClaimTicketResponse

Body of `POST /api/users/{id}/claim-ticket`.

- `ok` *boolean*, **required** — Always true; the status code says the ticket was minted.
- `claim_ticket` *string*, **required** — Opaque single-use claim ticket for the user.
- `invite_url` *string*, **required** — Invite link that redeems the claim ticket.

## UserCreatedResponse

Body of `POST /api/users`: the new user and, when one was asked for,
the invite that lets them claim it.

- `ok` *boolean*, **required** — Always true; the status code says the user was created.
- `user` *[`UserSummary`](#usersummary)*, **required** — The user as created.
- `claim_ticket` *string or null* — Opaque single-use claim ticket, when `mint_invite` was true.
- `invite_url` *string or null* — Invite link that redeems the claim ticket, when one was minted.

## UserDeletedResponse

Body of `DELETE /api/users/{id}`.

- `ok` *boolean*, **required** — Always true; the status code says the user was deleted.

## UserIdentitySummary

An external sign-in identity linked to a user.

- `provider` *string*, **required** — Identity-broker id (`oidc:google`, `oidc:github`, …).
- `external_user_id` *string*, **required** — The user's id at the provider.
- `label` *string or null* — Operator-facing label for the identity, when one was set.

## UserIntegrationHint

Storefront / integration connection summary for admin user lists.

- `source` *string*, **required** — Content-source or integration plugin id (`audible`, `libro`, …).
- `account_id` *string*, **required** — Store or operator account id this row belongs to.
- `label` *string or null* — Optional operator-facing account label.

## UserListeningHint

Recent unfinished listening hint for admin user lists.

- `title` *string or null* — Display title when the provider reported one.
- `provider` *string*, **required** — Integration / storefront provider id.
- `last_listened_at` *string (date-time)*, **required** — RFC 3339 time of the last playback update.

## UserListEntry

One user in the list `GET /api/users` returns: the summary plus
presence, avatar and connection details for administration.

Includes every field of [`UserSummary`](#usersummary).

- `online` *boolean*, **required** — Whether the user has a portal session that has not expired.
- `last_active_at` *string or null* — RFC 3339 time of the most recent activity on an unexpired portal session, when the user is online.
- `last_seen_at` *string or null* — RFC 3339 time of the last authenticated portal activity; kept after the session ends.
- `has_avatar` *boolean*, **required** — Whether an uploaded avatar is stored for the user.
- `avatar_source` *string*, **required** — Which avatar the user shows: `auto`, `upload`, `gravatar`, or an identity's picture.
- `gravatar_hash` *string or null* — SHA-256 digest of the user's email for Gravatar, when they have one.
- `sso_pictures` *array of [`UserSsoPictureSummary`](#userssopicturesummary)*, **required** — Pictures the user's identity providers supplied.
- `listening` *[`UserListeningHint`](#userlisteninghint) or null* — The most recent unfinished listen within the last 30 minutes, if any.
- `integrations` *array of [`UserIntegrationHint`](#userintegrationhint)*, **required** — Storefront and integration accounts linked to the user.
- `identities` *array of [`UserIdentitySummary`](#useridentitysummary)*, **required** — External sign-in identities linked to the user.

## UserListResponse

Body of `GET /api/users`.

- `users` *array of [`UserListEntry`](#userlistentry)*, **required** — Every first-party user.

## UserPasswordResetResponse

Body of `POST /api/users/{id}/reset-password`.

- `ok` *boolean*, **required** — Always true; the status code says the password was cleared.
- `claim_ticket` *string*, **required** — Opaque single-use claim ticket that lets the user set a new password.
- `revoked_sessions` *integer (int64)*, **required** — How many of the user's portal sessions were signed out. At least 0.

## UserRole

First-party user role (`owner` | `administrator` | `host` | `member`).

Type: one of `owner`, `administrator`, `host`, `member`

## UserSsoPictureSummary

A profile picture an identity provider supplied for a user.

- `identity_id` *integer (int64)*, **required** — The linked identity the picture came from.
- `provider` *string*, **required** — Identity-broker id (`oidc:google`, `oidc:github`, …).
- `picture_url` *string*, **required** — HTTPS URL of the picture at the provider or its CDN.
- `last_used_at` *string or null* — RFC 3339 time the identity last signed in, when it has.

## UserStatus

First-party user status.

Type: one of `active`, `disabled`

## UserSummary

A first-party user as the Users routes return one: profile, role and
status, never the password hash.

- `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
- `role` *[`UserRole`](#userrole)*, **required** — First-party role.
- `status` *[`UserStatus`](#userstatus)*, **required** — Whether the account may sign in.
- `display_name` *string or null* — Human-readable name shown in the UI, when set.
- `login_name` *string or null* — Local username for password login, when set.
- `email` *string or null* — Contact email, when set.
- `has_password` *boolean*, **required** — Whether a local password is stored for the user.

## WishlistPurchaseHint

Snapshotted purchase link/price for a wishlist title.

- `source` *string*, **required** — Content-source plugin id (`audible`, `libro`, …).
- `product_id` *string*, **required** — Storefront product id (ASIN, ISBN, UUID, …).
- `title` *string or null* — Display title of the work or edition.
- `url` *string or null* — Storefront product or purchase URL when known.
- `price_cents` *integer (int64) or null* — Observed price in minor currency units, when known.
- `currency` *string or null* — ISO 4217 currency code for price fields.
- `price_label` *string or null* — Storefront-formatted price string for display.
- `list_price_cents` *integer (int64) or null* — List/MSRP price in minor units, when known.
- `list_price_label` *string or null* — Storefront-formatted list price for display.
- `member_price_cents` *integer (int64) or null* — Member/subscriber price in minor units, when known.
- `member_price_label` *string or null* — Storefront-formatted member price for display.

## WishlistStoreEdition

Storefront edition key for wishlist / queue payloads.

- `source` *string*, **required** — Content-source plugin id (`audible`, `libro`, …).
- `product_id` *string*, **required** — Storefront product id (ASIN, ISBN, UUID, …).
