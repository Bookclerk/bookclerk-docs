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

## ActiveVersionChanged

`PUT /api/config/plugins/{key}/active` body: the pointer after the move.

- `plugin_key` *string*, **required** — Canonical plugin key.
- `version` *string*, **required** — Version now active.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision of the pointer after the move.

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
- `deny_own_user_agent_hosts` *array of string* — Requested hosts to refuse the plugin's own User-Agent on; requests to them carry Bookclerk's agent instead. No duplicates.
- `deny_network` *boolean* — Deny all network access.

## AuthMeImpersonating

Target user shown while an operator session is impersonating.

- `user_id` *integer (int64)*, **required** — First-party `users.id` being impersonated.
- `display_name` *string or null* — Display name of the impersonated user when the row exists.

## AuthMeResponse

JSON body for `GET /api/auth/me`.

- `authenticated` *boolean*, **required** — Whether the request resolved to an operator, portal, or impersonated session.
- `role` *string or null* — `operator`, `owner`, `administrator`, `host`, or `member` (legacy clients may still send `portal`).
- `default_view` *string*, **required** — Configured post-auth landing view (`discover` / `library` / `accounts` / `wishlist`). Loaded from per-user DB preferences (not config.toml).
- `can_acquire` *boolean*, **required** — Whether this session may acquire / scan / manage jobs. True for operator and administrator.
- `elevated` *boolean*, **required** — True when this operator session was created via Owner elevate.
- `impersonating` *[`AuthMeImpersonating`](#authmeimpersonating) or null* — Present when the operator is impersonating a User.
- `portal` *[`PortalMeInfo`](#portalmeinfo) or null* — Linked portal identity when the caller is a portal (or impersonated) user.
- `user` *[`AuthMeUser`](#authmeuser) or null* — First-party user when the session is linked to a `users` row.
- `second_factor` *[`AuthSecondFactor`](#authsecondfactor) or null* — Password-login second-factor policy and this user's enrollment.

## AuthMeUser

First-party user snapshot attached to `/me` when a `users` row is linked.

- `id` *integer (int64)*, **required** — First-party `users.id`.
- `role` *string*, **required** — Role string (`owner`, `administrator`, `member`).
- `display_name` *string or null* — Optional human-readable name from the user row.
- `email` *string or null* — Optional email; omitted from JSON when unset.
- `has_password` *boolean*, **required** — True when a local password hash is stored (invite users may be false).
- `has_avatar` *boolean*, **required** — True when a JPEG avatar is stored under `files_dir/avatars/{id}.*`.
- `avatar_source` *string*, **required** — Selected picture source (`auto`, `monogram`, `gravatar`, `upload`, `sso:{id}`).
- `gravatar_hash` *string or null* — SHA-256 hex of the contact email for Gravatar, when an email is set.
- `sso_pictures` *array of [`AuthSsoPicture`](#authssopicture)* — IdP-supplied pictures the user can pick.

## AuthSecondFactor

Host MFA policy plus whether this user has TOTP and/or passkeys.

- `required` *boolean*, **required** — `daemon.auth.require_second_factor`.
- `totp` *boolean*, **required** — Confirmed authenticator-app TOTP.
- `passkey_count` *integer (int64)*, **required** — Number of registered passkeys. At least 0.
- `enrolled` *boolean*, **required** — True when TOTP is enabled or at least one passkey exists.

## AuthSsoPicture

HTTPS picture from a linked identity provider.

- `identity_id` *integer (int64)*, **required** — `portal_identities.id`.
- `provider` *string*, **required** — Identity-broker id (`oidc:google`, …).
- `picture_url` *string*, **required** — HTTPS URL to display.
- `picture_path` *string or null* — Same-origin 128 px copy of `picture_url` for the web app to show, when this node can sign it.
- `last_used_at` *string or null* — RFC 3339 last portal-session activity for this identity, when any.

## AvatarChanged

JSON body after the caller's avatar was stored or removed.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `has_avatar` *boolean*, **required** — Whether an uploaded picture is stored now.
- `avatar_source` *string or null* — Present only after an upload, always `upload`; absent after a removal (the stored source is then unchanged unless it was `upload`).

## BadBookAction

How to handle acquire failures (`BadBook` setting).

Type: one of `ask`, `abort`, `retry`, `ignore`

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
- `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL; never stored. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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

## BranchAvailability

One branch a title is available through.

- `node_id` *string*, **required** — The branch node id.
- `owner_name` *string or null* — The branch owner's display name, for "available through Riley's branch"; absent when the owner has none.
- `store` *string*, **required** — The store the branch has the book from.
- `product_id` *string*, **required** — The store's product id of the branch's copy.

## BranchConsentBody

`POST /api/auth/branch-consent` body.

Includes every field of [`BranchAuthorizeParams`](#branchauthorizeparams).

- `approve` *boolean*, **required** — True to let the branch join; false to deny.
- `connection` *string or null* — `branch` (default) or `clerk`.
- `step_up` *[`StepUp`](#stepup) or null* — The fresh second factor a clerk needs.
- `bring_connections` *boolean* — True to send the user's store connections that may travel to the branch when it joins. Ignored for a clerk. Defaults to false.

## BranchConsentPage

JSON body for `GET /api/auth/branch-consent`: what the consent page shows.

- `library_name` *string*, **required** — This library's name.
- `branch_name` *string*, **required** — The name the branch asks to join as.
- `redirect_origin` *string*, **required** — Origin of the branch's redirect URI, for the page to name.
- `enrollment` *boolean*, **required** — Whether this library lets new branches join right now.
- `user` *[`ConsentingUser`](#consentinguser)*, **required** — The signed-in user the branch would belong to.
- `connection` *[`ConnectionChoice`](#connectionchoice)*, **required** — The connection types the page may offer.
- `connections` *array of [`ConsentConnection`](#consentconnection)*, **required** — The user's store connections here, and whether each would travel to the branch if the user chooses to bring them.

## BranchesResponse

JSON body for `GET /api/config/branches`.

- `library_name` *string*, **required** — This library's name.
- `origin` *string*, **required** — The canonical origin branches connect to.
- `transport` *string*, **required** — The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).
- `enrollment` *boolean*, **required** — Whether new branches may join.
- `spki_fingerprint` *string or null* — In `self_signed` mode, the fingerprint a joining operator confirms.
- `branches` *array of [`BranchView`](#branchview)*, **required** — Every branch, newest first.

## BranchJoined

JSON body once this install joined a library as a branch.

- `cluster` *[`ClusterLinkView`](#clusterlinkview)*, **required** — The new link.
- `library_name` *string or null* — The library's name, when the join came from a sign-in (not an invite link).
- `running` *boolean*, **required** — Whether the link's client is running now.
- `connections` *[`BroughtConnectionsView`](#broughtconnectionsview) or null* — What became of the store connections the library's user chose to bring to this branch; left out when none came.

## BranchJoinSetting

JSON body after `PUT /api/config/branches`.

- `enrollment` *boolean*, **required** — "Let new branches join", as stored.

## BranchOnlyChoice

The consent page's connection block when only a branch is offered.

- `choice` *boolean*, **required** — Always `false`.
- `default` *string*, **required** — Always `branch`.

## BranchTitleCounts

Counts of one branch's shared titles.

- `shared` *integer (int64)*, **required** — Titles it shares now. At least 0.
- `delivered` *integer (int64)*, **required** — Of those, titles whose files arrived. At least 0.
- `detached` *integer (int64)*, **required** — Titles marked detached (it stopped sharing, left, or withdrew them). At least 0.

## BranchTitlesRemoved

JSON body after detached branch titles were removed.

- `removed` *integer (int64)*, **required** — How many title entries were removed. At least 0.

## BranchView

One branch (host node) of this library.

- `node_id` *string*, **required** — The branch's node id.
- `created_at` *string*, **required** — RFC 3339 time the node was created.
- `owner_user_id` *integer (int64) or null* — The user who owns the branch.
- `owner_name` *string or null* — That user's display or login name.
- `joined` *boolean*, **required** — Whether its credential is still valid (it has not left or been revoked).
- `share_books` *boolean or null* — "Share this branch's books" as it last reported it; `null` until its first catalog sync.
- `include_branches` *boolean or null* — "Include books from my own branches" as it last reported it; `null` until its first catalog sync.
- `titles` *[`BranchTitleCounts`](#branchtitlecounts)*, **required** — Counts of the titles it shared.

## BrandInfo

CSS colors and logo href for a storefront or integration brand.

- `bg` *string*, **required** — Background color (`#RRGGBB`).
- `fg` *string*, **required** — Foreground / text color (`#RRGGBB`).
- `accent` *string*, **required** — Accent color (`#RRGGBB`).
- `logo` *string*, **required** — Same-origin logo image (a 128 px copy of the brand's icon), or empty when there is none.

## BroughtConnectionsView

The store connections a library sent with its person, as this install
kept them.

- `offered` *integer*, **required** — How many came with the join. At least 0.
- `added` *integer*, **required** — How many are now connected here for the person. At least 0.
- `already_here` *integer*, **required** — How many the person had already connected here (left as they are). At least 0.
- `taken` *integer*, **required** — How many are connected to someone else here (left as they are). At least 0.
- `failed` *integer*, **required** — How many could not be kept: no secret came with them, or their write failed. At least 0.
- `not_kept` *string or null* — Why none were kept, when none were: `not_allowed` (this install's operator does not let the person connect store accounts), `no_owner` (no single Owner here to keep them for) or `failed`.

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

## CapacityBody

`POST /api/config/join/capacity` body: one library and the job cap to
set for it.

- `cluster_id` *string*, **required** — The library (owner cluster id).
- `max_jobs` *integer (int32) or null*, **required** — The most jobs to run for the library at once (0 runs none); `null` gives it an equal share of the machine. Required: a body without it is refused rather than read as `null`. At least 0; at most 16.

## CatalogListing

Whether the caller's books are listed to the libraries this install is
a branch of (`GET` / `PUT /api/auth/profile/catalog-listing`).

- `listed` *boolean*, **required** — `true` (the default): the libraries this install joined can see, when they ask, that it can supply the caller's books. `false` keeps them out of those answers; the owner's books are always listed.

## CatalogListingView

The caller's listing choice and how many libraries it applies to.

- `listed` *boolean*, **required** — The caller's choice; see [`CatalogListing::listed`].
- `libraries` *integer (int32)*, **required** — Libraries this install is currently a branch of (active links). With none, the choice has no effect yet, and the app hides it. At least 0.
- `always_listed` *boolean*, **required** — `true` for an active Owner: an Owner's books are always listed, so `listed` has no effect and the app shows that instead of the toggle.

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
- `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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
- `available_through` *array of [`BranchAvailability`](#branchavailability)* — Branches of this library that currently list this title, matched through a store listing the library already holds. Filled by the daemon after the search.

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
- `credential` *any JSON*, **required** — W3C PublicKeyCredential as the browser serializes it: a registration attestation (`response.attestationObject`) or an authentication assertion (`response.authenticatorData`, `signature`, `userHandle`), binary fields base64url-encoded.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

## ChapterJsonMode

Whether / which chapter JSON sidecars to write beside acquired audio.

Default is [`Self::Off`]. Replaces the old `save_chapter_json` bool +
`chapter_layout` pair for *sidecar output* (Audible API fetch layout is
chosen separately when chapters are needed for embedding).

Type: one of `off`, `flat`, `tree`, `both`

## ClerkJoined

JSON body once this install joined a library as a clerk.

- `connection` *string*, **required** — Always `clerk`.
- `cluster_id` *string*, **required** — The library's cluster id.
- `node_id` *string*, **required** — The node id the library minted for this install.
- `library_name` *string*, **required** — The library's name.
- `database` *string*, **required** — The database plugin this install now runs (`postgres` or `d1`).
- `master_key_wrapped` *boolean*, **required** — Whether the shared master key was wrapped with this install's password.
- `restarting` *boolean*, **required** — Always `true`: the daemon restarts as an owner node right after answering.

## ClerksResponse

`GET /api/config/clerks` body.

- `node_id` *string*, **required** — Id of the node that answered.
- `nodes` *array of [`NodeView`](#nodeview)*, **required** — This node and the clerks the caller may see.

## ClusterConsentsResponse

JSON body for `GET /api/config/cluster-consents`.

- `clusters` *array of [`ClusterLinkView`](#clusterlinkview)*, **required** — Every library this install is a branch of.
- `consents` *array of [`RemoteConsentView`](#remoteconsentview)*, **required** — Every plugin request from those libraries, pending or answered.

## ClusterJobsBody

`POST /api/config/clusters/{cluster_id}/jobs` body.

- `job_ids` *array of string*, **required** — Job ids on the owner cluster.

## ClusterJobsResponse

JSON body for `POST /api/config/clusters/{cluster_id}/jobs`.

- `jobs` *array of [`JobStatusEntry`](#jobstatusentry)*, **required** — The owner's state of each job this install may see, in request order.

## ClusterLinkView

A library this install is a branch of: ids, origin and state, never key material.

- `cluster_id` *string*, **required** — The library's cluster id.
- `owner_origin` *string*, **required** — The library's canonical origin.
- `node_id` *string*, **required** — The node id the library minted for this install.
- `pinned` *boolean*, **required** — Whether the link pins the library's self-signed certificate.
- `state` *string*, **required** — `active`, `revoked`, or `left`.
- `enrolled_at` *string*, **required** — RFC 3339 enrollment time.
- `share_books` *boolean*, **required** — "Share this branch's books with the library".
- `include_branches` *boolean*, **required** — "Include books from my own branches".
- `work_for_members` *boolean*, **required** — "Download for other members of the library": this install takes the library's work for its other members too, not only its owner's (off by default).
- `keep_member_copies` *boolean*, **required** — "Keep copies of other members' books": a book acquired here for another member is also written to this install's own destinations (off by default).
- `max_jobs` *integer or null* — The most jobs this install runs for the library at once, as its operator set it; `null` gives the library an equal share of the machine. At least 0; at most 16.
- `job_share` *integer or null* — How many jobs the library may have running here now: its share of the machine after this install's own work, within `max_jobs`. Only the listing and sharing routes report it, and only while the link's client runs. At least 0.
- `list_members_books` *boolean*, **required** — "List my members' books to the library": the library's catalog lookups are answered with members' books (except those who opted out) as well as the owner's.
- `running` *boolean or null* — Whether this install's client for the link is running now; only the join routes report it.

## ClusterScanBody

`POST /api/config/clusters/{cluster_id}/scan` body.

- `account` *string or null* — Scan only this store account of the owner cluster.
- `plugin_instance_id` *string or null* — Pin every planned job to this owner plugin instance.

## ClusterScanPlanned

JSON body after the owner library queued a scan for this install.

- `cluster_id` *string*, **required** — The owner library's cluster id.
- `created` *array of string*, **required** — Ids of the jobs the owner created.
- `duplicates` *array of string*, **required** — Ids of jobs that were already pending or running.
- `queue_full` *integer (int64)*, **required** — Planned jobs the owner's queue had no room for. At least 0.

## ClusterStatusResponse

`GET /api/cluster/status` body.

- `cluster` *[`ClusterSummary`](#clustersummary)*, **required** — Cluster-wide counters.
- `node_id` *string*, **required** — Id of the node that answered.
- `nodes` *array of [`NodeView`](#nodeview)*, **required** — The nodes the caller administers.
- `capabilities` *[`NodeCapabilities`](#nodecapabilities)*, **required** — What the caller may do with them.

## ClusterSummary

Cluster-wide counters every node administrator may see.

- `nodes` *integer*, **required** — Nodes in the library's node list. At least 0.
- `live` *integer*, **required** — Nodes whose last heartbeat is recent enough. At least 0.
- `drained` *integer*, **required** — Live nodes an operator drained. At least 0.
- `host_nodes` *integer*, **required** — Enrolled hosts (nodes with their own database). At least 0.
- `cpu_budget_millicores` *integer (int64)*, **required** — CPU the scheduler may commit across live nodes, in millicores.
- `cpu_reserved_millicores` *integer (int64)*, **required** — CPU the running guests' ceilings add up to, in millicores.
- `memory_budget_bytes` *integer (int64)*, **required** — Memory the scheduler may commit across live nodes, in bytes.
- `memory_reserved_bytes` *integer (int64)*, **required** — Memory the running guests' ceilings add up to, in bytes.
- `running_guests` *integer (int64)*, **required** — Guests running across live nodes.
- `active_jobs` *integer (int64)*, **required** — Jobs queued or running.
- `deliveries` *[`DeliveryBacklog`](#deliverybacklog)*, **required** — Event delivery backlog.
- `leases` *array of [`LeaseView`](#leaseview)*, **required** — Every cluster lease, held or expired.
- `schema` *[`SchemaStateView`](#schemastateview)*, **required** — The database schema against this binary.

## ConfigOptionValueInfo

One allowed value for a storefront config option.

- `id` *string*, **required** — Stored option value (for example `m4b`).
- `label` *string*, **required** — Operator-facing value label.

## ConnectionChoice

What the consent page may offer: branch only, or a choice between branch and clerk for an Owner.

Type: [`BranchOnlyChoice`](#branchonlychoice) or [`OwnerChoice`](#ownerchoice)

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

## ConsentChanged

JSON body after a consent was approved or revoked.

- `consent` *[`RemoteConsentView`](#remoteconsentview)*, **required** — The consent as stored now.

## ConsentConnection

One of the consenting user's store connections, as the consent page
lists it.

- `source` *string*, **required** — Display alias of the storefront.
- `account_id` *string*, **required** — Store account id.
- `label` *string or null* — The account's display label, when it has one.
- `travels` *boolean*, **required** — True when the storefront lets its login be used in more than one place, so the connection travels; false when it keeps one login that only this library refreshes.

## ConsentingUser

The user a branch consent is for.

- `id` *integer (int64)*, **required** — `users.id`.
- `display_name` *string or null* — Display name, when set.

## ConsentRedirect

JSON body after a branch consent was answered.

- `redirect` *string*, **required** — Where to send the browser: the branch's callback with a code, or with `error=access_denied`.

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

## DeliveryBacklog

Event delivery backlog across the cluster.

- `pending` *integer (int64)*, **required** — Deliveries waiting for a consumer.
- `running` *integer (int64)*, **required** — Deliveries a consumer is handling now.
- `dead_letter` *integer (int64)*, **required** — Deliveries that exhausted their retries.
- `oldest_pending_age_secs` *integer (int64) or null* — Age of the oldest pending delivery in seconds; `null` when none is pending.

## DeploymentConflict

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}` `409` body: the two conflicts a deployment write can hit.

Type: [`RevisionConflict`](#revisionconflict) or [`ErrorMessage`](#errormessage)

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
- `state` *[`FeedState`](#feedstate) or null* — Whether these are the caller's current shelves, set by the daemon's read of the stored feed. Absent in a stored feed and from the CLI.
- `computed_at` *string or null* — RFC 3339 time the shelves were ranked; absent while `state` is `preparing`.

## DiscoverShelf

One horizontal Discover row (series to finish, more by author, …).

- `id` *string*, **required** — Stable identifier for this item.
- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `subtitle` *string or null* — Optional subtitle when the catalog distinguishes it from the title.
- `items` *array of [`Recommendation`](#recommendation)*, **required** — Ordered collection of child entries for this response.

## DiscoverStatusResponse

Stored Discover state of the caller's shelves.

- `subject` *string*, **required** — The caller's subject key (`operator`, `user:<id>`, `portal:<id>`).
- `ready` *boolean*, **required** — True once the caller has stored shelves to show.
- `scope` *string or null* — Seeds the stored feed was ranked from (`own` or `household`), when ranked.
- `feed_computed_at` *string or null* — RFC 3339 time the stored feed was ranked, when ranked.
- `candidates_fetched_at` *string or null* — RFC 3339 time the caller's last storefront fetch finished, when any.
- `updating` *boolean*, **required** — True while a fetch or ranking for the caller is waiting or running.

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

## DomainReplayed

Body of a `PUT` to a configuration domain whose write committed but
could not be read back as the current document: another write landed
between the commit and the read-back, so only the committed revision is
reported (the library also returns it when a request id is replayed).

- `namespace` *string*, **required** — Domain namespace (`core.events` or `core.output`).
- `replayed` *boolean*, **required** — Always `true`: the write was a replay.
- `revision` *integer (int64)*, **required** — Revision the earlier write produced.

## DomainRevision

Where a configuration domain document lives and which revision it is at:
the metadata every domain response carries before its settings.

- `namespace` *string*, **required** — Domain namespace (`core.events` or `core.output`).
- `scope_type` *string*, **required** — Scope type: `cluster` for a shared document, `node` for one node's.
- `scope_id` *string*, **required** — Scope id: `singleton` for the cluster, else the node id.
- `schema_version` *integer (int64)*, **required** — Version of the document's schema, starting at 1.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision; send it back as `expected_revision`.
- `updated_at` *string*, **required** — RFC 3339 time of the last write.
- `updated_by` *string*, **required** — Audit identity of the last writer.

## Elevated

JSON body after a successful Owner elevation.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `elevated` *boolean*, **required** — Always `true`: the cookie the response sets is an elevated operator session.
- `expires_in_secs` *integer (int64)*, **required** — Seconds until the elevated session expires. At least 0.

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

## EnrolledNodeView

JSON body after an enrollment link or branch sign-in was redeemed.

- `cluster_id` *string*, **required** — The cluster the node joined.
- `node_id` *string*, **required** — The node id minted for it.

## EnrollFinish

First TOTP code that confirms the pending secret.

- `code` *string*, **required** — Six-digit code from the authenticator app.

## EnrollmentLinkView

An enrollment link as the invites routes list it; never its secret.

- `link_id` *string*, **required** — Public link id.
- `for_user_id` *integer (int64)*, **required** — User the enrolled node will belong to.
- `label` *string*, **required** — Label shown in listings.
- `created_by` *string*, **required** — Audit id of whoever minted it (`operator` or `user:<id>`).
- `created_at` *string*, **required** — RFC 3339 mint time.
- `expires_at` *string*, **required** — RFC 3339 expiry.
- `redeemed_at` *string or null* — RFC 3339 redeem time, once a host used it.
- `redeemed_node_id` *string or null* — Node the redeem created.
- `revoked_at` *string or null* — RFC 3339 revoke time.
- `state` *string*, **required** — `pending`, `redeemed`, `revoked`, or `expired` right now.

## ErrorAnswer

An error answer in one of several shapes: the standard body, `error` alone, or `error` with `detail`; the route's examples name each.

Type: [`ErrorJson`](#errorjson) or [`ErrorMessage`](#errormessage) or [`ErrorDetail`](#errordetail)

## ErrorDetail

JSON error body with a machine slug and a longer explanation (`detail`).

- `error` *string*, **required** — Stable machine slug clients branch on.
- `detail` *string*, **required** — What went wrong, for the person reading the answer.

## ErrorJson

JSON error object (`error`, `message`, numeric `status`) for API clients.

- `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
- `message` *string*, **required** — Operator-facing explanation.
- `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.

## ErrorMessage

JSON error object with a single `error` message, written by handlers
that report a library or plugin error in their own words.

- `error` *string*, **required** — What went wrong, as the handler describes it.

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

## EventsDomainChange

Body of `PUT /api/config/domains/core.events`: the settings after the
change, or a replay note.

Type: [`EventsDomainResponse`](#eventsdomainresponse) or [`DomainReplayed`](#domainreplayed)

## EventsDomainResponse

Body of `GET /api/config/domains/core.events` and of a `PUT` that was
applied: the revision metadata, then the `[events]` settings themselves.

Includes every field of [`DomainRevision`](#domainrevision), [`EventsSettingsV1`](#eventssettingsv1).

## EventsSettingsV1

Typed `[events]` document, schema version 1.

- `retention_days` *integer (int64)*, **required** — Days to keep acked and rejected deliveries. At least 0.
- `dead_letter_retention_days` *integer (int64)*, **required** — Days to keep dead-lettered deliveries. At least 0.
- `concurrency` *integer (int32)*, **required** — Local delivery workers and the cluster in-flight cap. At least 0.

## FeedState

How current a stored Discover feed is.

Type: one of `ready`, `refreshing`, `preparing`

## FileTimestampMode

Which timestamp to apply after acquire (`CreationTime` / `LastWriteTime`).

Type: one of `now`, `purchased`, `published`

## GuestReservation

CPU and memory one guest may use at most: its grant's ceilings after the
node's `[plugins.jail]` limits.

- `memory_bytes` *integer (int64)*, **required** — Memory ceiling in bytes.
- `cpu_millicores` *integer (int64)*, **required** — CPU ceiling in millicores.

## HealthResponse

Liveness payload for unauthenticated `GET /health`.

- `status` *string*, **required** — Always `ok` when the process can serve HTTP.

## HostEnrollment

The host-enrollment switch.

- `enabled` *boolean*, **required** — Whether host nodes may enroll into this cluster.
- `revision` *integer (int64) or null* — Stored revision; `null` while the default (enabled) applies.

## ImpersonateRequest

JSON body naming the first-party user to impersonate.

- `user_id` *integer (int64)*, **required** — Target `users.id` to impersonate.

## ImpersonationStarted

JSON body after an operator session started impersonating a user.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `impersonating` *[`AuthMeImpersonating`](#authmeimpersonating)*, **required** — The user the session now acts as.

## InspectBody

`POST /api/config/join/inspect` body.

- `address` *string*, **required** — The library's address (`https://owner.example`, `owner.lan:8443`).

## InspectedLibraryResponse

JSON body for `POST /api/config/join/inspect`.

- `library` *[`InspectedLibraryView`](#inspectedlibraryview)*, **required** — What the library said about itself.

## InspectedLibraryView

A library as `POST /api/config/join/inspect` saw it.

- `cluster_id` *string*, **required** — The library's cluster id.
- `library_name` *string*, **required** — A name to show for the library.
- `origin` *string*, **required** — Its canonical origin.
- `transport` *string*, **required** — The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).
- `enrollment` *boolean*, **required** — Whether it lets new branches join.
- `fingerprint` *string or null* — The SPKI fingerprint to confirm (self-signed libraries only).
- `needs_fingerprint` *boolean*, **required** — Whether the operator must confirm `fingerprint` before joining.

## InstalledPluginDetail

`GET /api/config/plugins/{key}` body: the plugin and its pinned schema.

Includes every field of [`InstalledPluginSummary`](#installedpluginsummary).

- `schema` *[`PinnedSchemaSummary`](#pinnedschemasummary) or null* — The pinned settings schema; `null` until a node pins one.

## InstalledPluginsResponse

`GET /api/config/plugins` body.

- `plugins` *array of [`InstalledPluginSummary`](#installedpluginsummary)*, **required** — Every plugin with at least one installed version, by key.

## InstalledPluginSummary

One installed plugin: its active version and every installed version,
newest first.

- `plugin_key` *string*, **required** — Canonical plugin key.
- `id` *string or null* — Manifest id (display alias) of the active version, or of the newest one when none is active; `null` without versions.
- `active_version` *string or null* — Active version; `null` when the plugin is installed but not activated.
- `revision` *integer (int64) or null* — Compare-and-swap revision of the active-version pointer; `null` when nothing is active.
- `versions` *array of [`PluginVersionSummary`](#pluginversionsummary)*, **required** — Every installed version, newest first.

## InstanceConfigView

Instance document as the API renders it: values and secret *names* only.

- `revision` *integer (int64)*, **required** — Compare-and-swap revision.
- `settings` *map of string to [`SettingValue`](#settingvalue)*, **required** — Scalar settings the operator wrote.
- `secret_refs` *array of [`InstanceSecretRefV1`](#instancesecretrefv1)*, **required** — Secret refs; ciphertext stays in `encrypted_secrets`.

## InstanceSecretRefV1

A name in `encrypted_secrets`. The ciphertext is not part of the document.

- `key` *string*, **required** — Key in the `SECRETS` object delivered to the guest.
- `name` *string*, **required** — `encrypted_secrets.name` for this instance.

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

Type: one of `scan`, `acquire`, `listen_sync`, `integration_scan`, `plugin_copy`, `plugin_scheduled`, `discover_candidates`, `discover_feed`, `discover_request_queue`, `cache_sweep`, `search_index_rebuild`, `invalid`

## JobState

Lifecycle state of a durable job row.

Type: one of `pending`, `running`, `succeeded`, `failed`, `cancelled`

## JobStatusEntry

One job's state in [`Body::JobStatuses`].

- `id` *string*, **required** — Job id.
- `kind` *string*, **required** — Job kind (`scan`, `acquire`, ...).
- `state` *string*, **required** — `pending`, `running`, `succeeded`, `failed`, or `cancelled`.
- `progress` *string or null* — Progress or result text.
- `error_kind` *string or null* — Structured error kind when it failed or was cancelled.
- `error_message` *string or null* — Error text when it failed or was cancelled.

## JoinCompleted

JSON body once a join completed: as a branch, or as a clerk.

Type: [`BranchJoined`](#branchjoined) or [`ClerkJoined`](#clerkjoined)

## JoinConflict

A `409` from a join step: the handler's error, or an origin mismatch that names the address to join with; the route's examples name each.

Type: [`ErrorDetail`](#errordetail) or [`OriginMismatch`](#originmismatch)

## JoinStarted

JSON body for `POST /api/config/join/start`.

- `authorize_url` *string*, **required** — The library's sign-in URL to open in the browser.
- `library` *[`InspectedLibraryView`](#inspectedlibraryview)*, **required** — What the library said about itself.
- `branch_name` *string*, **required** — The name this install asked to join as.
- `clerk_offered` *boolean*, **required** — Whether joining as a clerk was offered.
- `expires_in_secs` *integer (int64)*, **required** — Seconds the pending sign-in stays valid. At least 0.

## JoinStatus

JSON body for `GET /api/config/join`.

- `fresh` *boolean*, **required** — Whether this install has no users and no libraries yet.
- `has_owner` *boolean*, **required** — Whether an Owner (or legacy administrator) exists here.
- `clerk_eligible` *boolean*, **required** — Whether this install could still join a library as a clerk.
- `clerk_problem` *string or null* — Why it cannot join as a clerk, when it cannot.
- `default_branch_name` *string*, **required** — The name a join is offered under by default (this machine's name).
- `links` *array of [`ClusterLinkView`](#clusterlinkview)*, **required** — Every library this install is a branch of, with whether its client runs.

## LameConfig

LAME encoder tuning (classic `Lame*` settings).

- `target` *string* — `LameTarget`: `quality`, `bitrate`, `constant`, etc. Default `"quality"`.
- `vbr_quality` *integer (int32)* — `LameVBRQuality` (0–9, lower = better). At least 0; default `2`.
- `bitrate_kbps` *integer (int32)* — `LameBitrate` in kbps when target is bitrate/constant. At least 0; default `128`.
- `mode` *string* — `LameMode`: `default`, `mono`, `stereo`. Default `"default"`.
- `downsample_mono` *boolean* — `LameDownsampleMono`. Default `false`.
- `constant_bitrate` *boolean* — `LameConstantBitrate`. Default `false`.

## LeaseView

A cluster lease as the status page shows it.

- `name` *string*, **required** — Lease name.
- `holder` *string*, **required** — Holder (a node id).
- `token` *integer (int64)*, **required** — Fencing token; a new holder always gets a larger one.
- `held` *boolean*, **required** — True while the lease has not expired.
- `expires_at_ms` *integer (int64)*, **required** — Expiry in Unix milliseconds.

## LeaveBody

`POST /api/config/join/leave` body.

- `cluster_id` *string*, **required** — The library (owner cluster id) to leave.

## LeaveStarted

JSON body once leaving a library started.

- `cluster` *[`ClusterLinkView`](#clusterlinkview)*, **required** — The link, now `left`.
- `owner_notified` *boolean*, **required** — Whether the library was told (otherwise it notices the missed heartbeats).

## LinkedIdentitiesResponse

JSON body for `GET /api/auth/oidc/identities`.

- `identities` *array of [`LinkedIdentity`](#linkedidentity)*, **required** — Every sign-in identity linked to the caller's user, the local one included.

## LinkedIdentity

One sign-in identity linked to a user.

- `provider` *string*, **required** — Identity-broker id (`local`, `oidc:google`, …).
- `external_user_id` *string*, **required** — Provider-stable subject.
- `label` *string or null* — Operator-facing label, when the provider supplied one.

## LoginBegin

Login-name or email used to start a discoverable-less passkey login.

- `login` *string*, **required** — Login name or email looked up before issuing an authentication challenge.

## LoginRequest

JSON body for `POST /api/auth/login` (operator token).

- `token` *string*, **required** — Operator token from the client; compared in constant time and never persisted.

## LoginResponse

JSON body after a successful operator token login.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `role` *string*, **required** — Issued role (`operator` for token login).
- `default_view` *string*, **required** — Post-login SPA landing view from operator preferences (`discover` on lookup failure).

## LoginThrottled

JSON body of a 429 after too many failed sign-in attempts from one client.

- `error` *string*, **required** — Always `too_many_requests`.
- `message` *string*, **required** — Caller-facing explanation naming the wait.
- `status` *integer (int32)*, **required** — Always 429 (duplicates the response status for clients). At least 0.
- `retry_after_secs` *integer (int64)*, **required** — Seconds until this client may try again; the same value as `Retry-After`. At least 0.

## LoginVerify

TOTP code plus the challenge id from password login.

- `challenge_id` *string*, **required** — Server-issued challenge id (5-minute TTL, single use).
- `code` *string*, **required** — Six-digit authenticator code.

## MeResponse

JSON body for `GET /me`.

- `provider` *string*, **required** — Identity provider id (`local` or an integration plugin id).
- `external_user_id` *string*, **required** — Provider-scoped user id stored on the portal identity row.
- `label` *string or null* — Optional display label from the identity row.

## MfaPolicy

The host-wide second-factor policy.

- `require_second_factor` *boolean*, **required** — Whether every password sign-in must be followed by a passkey or authenticator code.

## MfaPolicyPut

PUT body for the host-wide second-factor policy.

- `require_second_factor` *boolean*, **required** — When true, password login requires TOTP or the user must use a passkey.
- `current_password` *string or null* — Current password for Owner step-up (operators omit this).

## MultiDestinationMode

Behavior when some output destinations already have a title and others do not.

Type: one of `sync_missing`, `refetch_missing`, `refetch_all`

## NamingProfile

Preset path-template profile.

Select via `output.naming_profile`. Individual `folder_template` /
`file_template` / `chapter_file_template` values override the profile when
set.

Type: one of `audiobookshelf`, `classic`

## NodeCapabilities

What the caller may do on the nodes it sees.

- `scope` *[`NodeScope`](#nodescope)*, **required** — `all` for the operator, `owned` for a node owner.
- `can_edit_runtime` *boolean*, **required** — True: every caller may edit the runtime settings of the nodes it sees.
- `can_edit_labels` *boolean*, **required** — True for the operator, who may replace placement labels.
- `can_assign_owner` *boolean*, **required** — True for the operator, who may assign or clear a node's owner.

## NodeCredentialRevoked

JSON body after a host's credential was revoked.

- `credential` *[`NodeCredentialView`](#nodecredentialview)*, **required** — The credential, with `revoked_at` set (revoking twice keeps the first time).

## NodeCredentialView

A host's node credential as the credentials routes show it; never its keys.

- `node_id` *string*, **required** — Host node.
- `link_id` *string*, **required** — Link that enrolled it.
- `enrolled_at` *string*, **required** — RFC 3339 enrollment time.
- `revoked_at` *string or null* — RFC 3339 revoke time.
- `last_connected_at` *string or null* — RFC 3339 time of the last authenticated connection.

## NodeInviteCreated

JSON body after an enrollment link was minted.

- `link` *[`EnrollmentLinkView`](#enrollmentlinkview)*, **required** — The link's record.
- `url` *string*, **required** — The full enrollment URL with the secret, shown once.

## NodeInviteRevoked

JSON body after an enrollment link was revoked.

- `link` *[`EnrollmentLinkView`](#enrollmentlinkview)*, **required** — The link, with `revoked_at` set.

## NodeInvitesResponse

JSON body for `GET /api/config/node-invites`.

- `links` *array of [`EnrollmentLinkView`](#enrollmentlinkview)*, **required** — The links the caller may see, newest first.

## NodeKind

How a node joined the cluster.

Type: one of `owner`, `host`

## NodeOwnerChanged

`PUT /api/config/nodes/{node_id}/owner` body: the owner after the
change.

- `node_id` *string*, **required** — Node id.
- `owner_user_id` *integer (int64) or null* — User id of the owner; `null` when cleared.

## NodeOwnerSummary

The user who owns a node, as the nodes listing shows them.

- `user_id` *integer (int64)*, **required** — The owner's user id.
- `display_name` *string or null* — Display name, or the login name when none is set; `null` when the user has neither.
- `role` *[`UserRole`](#userrole)*, **required** — The owner's role.

## NodeRefusal

A `403` from a node API route: the transport policy's refusal, the standard body, or the handler's own error; the route's examples name each.

Type: [`TransportRefused`](#transportrefused) or [`ErrorJson`](#errorjson) or [`ErrorMessage`](#errormessage) or [`ErrorDetail`](#errordetail)

## NodeRuntimeChanged

`PUT /api/config/nodes/{node_id}/runtime` body: the document after the
write.

- `node_id` *string*, **required** — Node id, trimmed.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision of the runtime document after the write.
- `runtime` *[`NodeRuntimeSettingsV1`](#noderuntimesettingsv1)*, **required** — The runtime document now stored.

## NodeRuntimeSettingsV1

Typed node-scoped runtime document, schema version 1.

Desired state for one node, editable from any node in the cluster: an
operator label, and the budget the scheduler may commit on this node.
Budgets are shares of the capacity the node detects at heartbeat
(CPU cores and memory, the tighter of the machine and its cgroup).

- `label` *string*, **required** — Operator label shown for this node. Empty is allowed.
- `cpu_budget_percent` *integer (int32)* — Share of detected CPU cores the scheduler may commit to plugin guests on this node (1..=100). At least 1; at most 100.
- `memory_budget_percent` *integer (int32)* — Share of detected memory the scheduler may commit (1..=100). At least 1; at most 100.
- `schedulable` *boolean* — False drains the node: the scheduler places nothing new here and moves its replicas elsewhere. Hand-managed deployments still run.
- `cache_budget_bytes` *integer (int64) or null* — Disk space, in bytes, this node's image caches may use together, overriding the cluster's `core.cache` budget (16 MiB to 1 TiB). Absent means the cluster budget applies. At least 16777216; at most 1099511627776.

## NodeScope

Which nodes a listing shows the caller.

Type: one of `all`, `owned`

## NodesResponse

`GET /api/config/nodes` body.

- `node_id` *string*, **required** — Id of the node that answered.
- `nodes` *array of [`NodeView`](#nodeview)*, **required** — The nodes the caller administers.
- `capabilities` *[`NodeCapabilities`](#nodecapabilities)*, **required** — What the caller may do with them.

## NodeStatus

A node's latest heartbeat with budget, accounting, and availability.

- `node_id` *string*, **required** — Reporting node.
- `incarnation` *string*, **required** — Daemon process incarnation.
- `started_at` *string*, **required** — RFC 3339 daemon start.
- `heartbeat_at` *string*, **required** — RFC 3339 heartbeat time.
- `heartbeat_ms` *integer (int64)*, **required** — Heartbeat time in Unix milliseconds.
- `interval_ms` *integer (int64)*, **required** — Promised interval in milliseconds.
- `cpu_millicores` *integer (int64)*, **required** — Detected CPU capacity in millicores.
- `memory_bytes` *integer (int64) or null* — Detected memory in bytes.
- `cpu_budget_millicores` *integer (int64)*, **required** — CPU the scheduler may commit, in millicores.
- `memory_budget_bytes` *integer (int64) or null* — Memory the scheduler may commit, in bytes.
- `schedulable` *boolean*, **required** — False when an operator drained the node.
- `cpu_used_millicores` *integer (int64) or null* — Measured CPU in use, in millicores.
- `memory_available_bytes` *integer (int64) or null* — Measured memory available, in bytes.
- `files_dir_free_bytes` *integer (int64) or null* — Free bytes on the files directory's filesystem.
- `reserved` *[`GuestReservation`](#guestreservation)*, **required** — Ceilings of the guests running here.
- `running_guests` *integer (int64)*, **required** — Guests running here.
- `availability_ppm` *integer (int64)*, **required** — Recent online fraction in parts per million.
- `online_since_ms` *integer (int64)*, **required** — Unix milliseconds the current online stretch began.
- `cache_bytes` *integer (int64) or null* — Bytes the node's image caches hold; `None` from a node that keeps none.
- `cache_budget_bytes` *integer (int64) or null* — Bytes the node's image caches may hold.
- `cache_swept_ms` *integer (int64) or null* — When the node's last cache sweep finished, in Unix milliseconds.

## NodeView

One node with its latest heartbeat, budget and owner.

- `node_id` *string*, **required** — Node id.
- `cluster_id` *string*, **required** — Cluster the node belongs to.
- `created_at` *string*, **required** — RFC 3339 time the node first enrolled.
- `heartbeat_at` *string*, **required** — RFC 3339 time of the last heartbeat the node row recorded.
- `software_version` *string*, **required** — Bookclerk version the node runs.
- `schema_state` *string*, **required** — Database schema the node last applied.
- `compatible` *boolean*, **required** — True when the node's own binary accepted the schema it reported; not a comparison with this daemon's schema.
- `target` *string*, **required** — Bookclerk target the node runs on.
- `labels` *array of string*, **required** — Operator-set placement labels.
- `implicit_labels` *array of string*, **required** — Labels the scheduler adds for the node (its id and target).
- `live` *boolean*, **required** — True while the last heartbeat is recent enough.
- `runtime` *[`NodeRuntimeSettingsV1`](#noderuntimesettingsv1)*, **required** — The node's runtime document, or the defaults when it never wrote one.
- `status` *[`NodeStatus`](#nodestatus) or null* — The latest heartbeat; `null` before the first one.
- `this_node` *boolean*, **required** — True for the node that answered the request.
- `kind` *[`NodeKind`](#nodekind)*, **required** — `owner` for a node sharing this library, `host` for an enrolled host.
- `owner_user_id` *integer (int64) or null* — User id of the node's owner; `null` when unassigned.
- `share_books` *boolean or null* — A branch's sharing setting, as it last reported it; `null` for an owner node.
- `include_branches` *boolean or null* — A branch's include-branches setting, as it last reported it; `null` for an owner node.
- `owner` *[`NodeOwnerSummary`](#nodeownersummary) or null* — The owner, when the node has one and the user still exists.

## OAuthEnrollAnswer

`POST /api/node/v1/enroll/oauth` answer.

- `cluster_id` *string*, **required** — The library's cluster id.
- `node_id` *string*, **required** — The node id recorded for the joining install.
- `connection` *[`ConnectionType`](#connectiontype)* — The connection type recorded (`branch` when left out).
- `bootstrap` *string or null* — For a clerk: the [`crate::ClerkBootstrap`] sealed to the node's box key ([`crate::seal_clerk_bootstrap`]), as base64url without padding.
- `accounts` *string or null* — For a branch whose user chose "Bring my store connections to this branch": the user's own store accounts as a [`crate::AccountBundle`] sealed to the node's box key ([`crate::seal_account_bundle`]), as base64url without padding. Left out when the user did not choose it or had none to bring.

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

## OAuthStartUrl

JSON body of an OAuth start: the browser login URL to open.

- `url` *string*, **required** — The store's login page, bound to this sign-in.

## OidcClientsResponse

JSON body for `GET /api/auth/oidc/clients`.

- `clients` *array of [`OidcClientView`](#oidcclientview)*, **required** — Every registered client, without secrets.

## OidcClientView

A Bookclerk-as-IdP client as operators see it; the secret appears only when freshly minted.

- `client_id` *string*, **required** — Public client id used at the authorize and token endpoints.
- `name` *string or null* — Operator-facing display name.
- `redirect_uris` *array of string*, **required** — Allowed OAuth redirect URIs.
- `confidential` *boolean*, **required** — Whether the client authenticates with a secret (otherwise it is a public PKCE client).
- `issue_refresh_token` *boolean*, **required** — Whether token responses include a refresh token.
- `allowed_scopes` *array of string*, **required** — Scopes the client may be granted.
- `has_secret` *boolean*, **required** — Same as `confidential`: a secret hash is stored.
- `enabled` *boolean*, **required** — Whether the authorize and token endpoints accept this client.
- `plugin_id` *string or null* — Plugin that provides this client, or `null` for an operator-created one.
- `client_secret` *string or null* — The new secret, shown once after create, rotate-secret, or a PUT that makes a public client confidential; never stored in clear.

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

## OidcProvidersResponse

JSON body for `GET /api/auth/oidc/providers`.

- `enabled` *boolean*, **required** — Whether the OIDC broker is switched on at all.
- `providers` *array of [`SigninOidcProvider`](#signinoidcprovider)*, **required** — Providers enabled in configuration, in configuration order (empty while the broker is off).

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

## OkResponse

Acknowledgement body (`{"ok": true}`) for actions with nothing else to report.

- `ok` *boolean*, **required** — Always `true`: a failure answers with an error status instead.

## OpenApiDocument

An OpenAPI document as this daemon serves it: the `openapi` version string, `info` about the daemon, every route under `paths` and the schemas under `components`.

Type: object

## OpenApiV30Unavailable

Body of the `500` from `GET /api/openapi-3.0.json` when the 3.1
document could not be restated as 3.0.3.

- `error` *string*, **required** — Always `openapi_v30_unavailable`.
- `message` *string*, **required** — What to do: the daemon log names the form that could not be restated.

## OperatorSessionView

An operator session as listed by `GET /api/auth/sessions`.

- `id` *integer (int64)*, **required** — Session id, for `DELETE /api/auth/sessions/{id}`.
- `created_at` *string*, **required** — RFC 3339 time the session was created.
- `last_used_at` *string or null* — RFC 3339 time of the last authenticated use, when any.
- `expires_at` *string*, **required** — RFC 3339 expiry.
- `elevated` *boolean*, **required** — Whether an Owner elevated into this session (rather than the operator token).
- `impersonating_user_id` *integer (int64) or null* — User the session is impersonating, when it is. Never set on the session making the request: while that session impersonates, the list shows the target's portal sessions instead.
- `is_current` *boolean*, **required** — Whether this is the session making the request.
- `client_label` *string or null* — Best-effort OS or client label from the User-Agent.
- `device_type` *string or null* — Best-effort device class (`desktop`, `mobile`, `tablet`, `api`).
- `user_agent` *string or null* — Raw User-Agent captured when the session was created.

## OriginMismatch

`409 origin_mismatch`: the library's canonical address is not the one
the join was started with. An [`crate::http_error::ErrorDetail`] that
also carries the address to use.

- `error` *string*, **required** — Always `origin_mismatch`.
- `detail` *string*, **required** — Why, naming the address to join with instead.
- `origin` *string*, **required** — The library's canonical origin, to join with instead.

## OutputDomainChange

Body of `PUT /api/config/domains/core.output`: the settings after the
change, or a replay note.

Type: [`OutputDomainResponse`](#outputdomainresponse) or [`DomainReplayed`](#domainreplayed)

## OutputDomainResponse

Body of `GET /api/config/domains/core.output` and of a `PUT` that was
applied: the revision metadata and the `[output]` document.

Includes every field of [`DomainRevision`](#domainrevision).

- `document` *[`OutputSettingsV1`](#outputsettingsv1)*, **required** — The settings at that revision (schema version 1).

## OutputFormat

Post-download output formatting. Default is enriched M4B (tags + chapters).
`split_mp3_by_size` and `opus` parse but are not implemented yet: the
acquire pipeline rejects a job that selects either before downloading.

Type: one of `enriched_m4b`, `single_mp3`, `split_mp3_by_chapter`, `split_mp3_by_size`, `opus`, `none`

## OutputSettingsV1

Typed `core.output` document, schema version 1.

Field meanings match [`OutputConfig`]. Unknown fields are rejected, inside
the nested `lame` and `replacement_characters` objects too, so a document
written by a newer binary under the same schema version fails closed
instead of being silently truncated on the next write.

- `format` *[`OutputFormat`](#outputformat)*, **required** — Post-acquire packaging format.
- `widevine` *boolean*, **required** — Prefer Widevine/CENC.
- `xhe_aac` *boolean*, **required** — Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm` *string or null* — Optional `.wvd` path; relative paths resolve under each host's files dir.
- `widevine_cdm_provider` *string or null* — Remote L3 CDM provider endpoint URL (`None` = the built-in AudibleCdm provider, empty/`off` disables provisioning).
- `naming_profile` *[`NamingProfile`](#namingprofile)*, **required** — Named path-template preset.
- `folder_template` *string or null* — Folder-path template override.
- `file_template` *string or null* — File-stem template override.
- `chapter_file_template` *string or null* — Per-chapter file-stem template override.
- `chapter_title_template` *string or null* — Embedded chapter-title template.
- `download_cover` *boolean*, **required** — Save cover JPEG alongside audio.
- `download_pdf` *boolean*, **required** — Download companion PDF when available.
- `create_cue` *boolean*, **required** — Write a `.cue` sidecar.
- `fixup_metadata` *boolean*, **required** — Embed tags, cover, and chapters natively.
- `chapter_json` *[`ChapterJsonMode`](#chapterjsonmode)*, **required** — Chapter JSON sidecars.
- `save_metadata_json` *boolean*, **required** — Persist raw catalog API JSON.
- `cover_size` *string*, **required** — Cover image size (`500`, `1215`, another pixel size, or `native`).
- `chapter_layout` *string*, **required** — Preferred chapter API layout (`tree`, `flat`, or `both`).
- `overwrite_existing` *boolean*, **required** — Re-acquire when media already exists at the destination.
- `multi_destination` *[`MultiDestinationMode`](#multidestinationmode)*, **required** — Policy when only some enabled destinations have a title.
- `bad_book_action` *[`BadBookAction`](#badbookaction)*, **required** — Operator action when a title fails to acquire.
- `split_mp3_max_mb` *integer (int32)*, **required** — Max MP3 part size in MiB for `split_mp3_by_size`. At least 0.
- `minimum_file_duration_minutes` *integer (int32)*, **required** — Drop chapter splits shorter than this many minutes. At least 0.
- `combine_nested_chapter_titles` *boolean*, **required** — Flatten nested chapter titles.
- `merge_opening_and_end_credits` *boolean*, **required** — Merge opening/end credit chapters into neighbours.
- `strip_unabridged` *boolean*, **required** — Strip an "Unabridged" suffix from titles used in naming.
- `strip_audible_brand_audio` *boolean*, **required** — Trim Audible brand intro/outro audio.
- `download_clips_bookmarks` *boolean*, **required** — Download Audible clips/bookmarks sidecars.
- `retain_aax_file` *boolean*, **required** — Keep the encrypted download.
- `download_speed_limit_kbps` *integer (int32)*, **required** — Fetch speed cap in KB/s (`0` = unlimited). At least 0.
- `lame` *[`LameConfig`](#lameconfig)*, **required** — LAME encoder knobs.
- `max_sample_rate` *integer (int32) or null* — Optional output sample-rate ceiling in Hz. At least 0.
- `creation_time` *[`FileTimestampMode`](#filetimestampmode)*, **required** — File creation timestamp policy.
- `last_write_time` *[`FileTimestampMode`](#filetimestampmode)*, **required** — File last-write timestamp policy.
- `path_sanitization` *[`PathSanitizationMode`](#pathsanitizationmode)*, **required** — Sanitization profile used when no explicit rules are set.
- `replacement_characters` *array of [`ReplacementRule`](#replacementrule)*, **required** — Explicit find/replace rules for path segments.
- `max_filename_length` *integer (int32)*, **required** — Max length per path segment (`0` disables truncation). At least 0.

## OwnerBootstrapped

JSON body after the first Owner was created.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `user_id` *integer (int64)*, **required** — `users.id` of the new Owner.
- `claim_ticket` *string*, **required** — Single-use claim ticket that signs the Owner in once (valid for seven days).
- `invite_url` *string*, **required** — Invite link carrying the ticket, for the browser.
- `login_name` *string or null* — Login name the Owner signs in with, when one was given.
- `has_password` *boolean*, **required** — Whether a password was set, so password sign-in works right away.

## OwnerChoice

The consent page's connection block for an Owner, who may pick clerk.

- `choice` *boolean*, **required** — Always `true`.
- `default` *string*, **required** — Always `branch`.
- `clerk_available` *boolean*, **required** — Whether clerk may be chosen right now.
- `clerk_reason` *string or null* — Why clerk is shown disabled, when it is.
- `step_up` *array of string*, **required** — The second factors the Owner can step up with (`totp`, `passkey`).

## OwnerInfo

`GET /api/node/v1/info` answer.

- `cluster_id` *string*, **required** — The library's cluster id.
- `library_name` *string*, **required** — A name to show for the library.
- `origin` *string*, **required** — The library's canonical origin: the one the node handshake signs and the authorize endpoint lives under.
- `transport` *string*, **required** — The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).
- `enrollment` *boolean*, **required** — Whether the library lets new branches join (`core.host_enrollment`).
- `spki_fingerprint` *string or null* — In `self_signed` mode: base64url SHA-256 of the certificate's SubjectPublicKeyInfo, which the joining operator confirms against the owner's Settings → Branches page.
- `software_version` *string* — The library's Bookclerk version.
- `ancestry` *array of string* — The library's ancestry: its own cluster id first, then the cluster id of every library above it up to the root. A joining library refuses a parent whose ancestry names its own cluster id, so a tree of libraries never becomes a cycle.

## OwnerNodeRemoved

`DELETE /api/config/nodes/{node_id}` body.

- `node_id` *string*, **required** — Id of the node that was removed, trimmed.
- `removed` *boolean*, **required** — Always true: a refused removal is an error status instead.

## PasskeyCreationOptions

WebAuthn creation options for `navigator.credentials.create()`, plus the challenge id for the finish step.

- `challenge_id` *string*, **required** — Server-issued challenge id to send back with the attestation; consumed once, five-minute lifetime.
- `publicKey` *any JSON*, **required** — W3C PublicKeyCredentialCreationOptions (relying party, user, challenge, algorithms, excluded credentials), binary fields base64url-encoded.

## PasskeyReauthBody

Optional current password for step-up before register or delete.

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

## PasskeyRequestOptions

WebAuthn request options for `navigator.credentials.get()`, plus the challenge id for the finish step.

- `challenge_id` *string*, **required** — Server-issued challenge id to send back with the assertion; consumed once, five-minute lifetime.
- `publicKey` *any JSON*, **required** — W3C PublicKeyCredentialRequestOptions (challenge, relying party id, allowed credentials), binary fields base64url-encoded.
- `mediation` *string or null* — Credential mediation the daemon asks the browser for; absent unless conditional mediation was requested.

## PasskeysResponse

JSON body for `GET /api/auth/passkeys`.

- `passkeys` *array of [`PasskeySummary`](#passkeysummary)*, **required** — The caller's passkeys, oldest first.

## PasskeySummary

One stored passkey.

- `id` *integer (int64)*, **required** — Row id, for `DELETE /api/auth/passkeys/{id}`.
- `credential_id` *string*, **required** — Base64url credential id the authenticator reports.
- `name` *string*, **required** — Label chosen at registration (`Passkey` when none was given).

## PasswordLoginBody

JSON body for password-store connect (`email` + `password`).

- `email` *string*, **required** — Store account email / username.
- `password` *string*, **required** — Store account password; never logged.

## PasswordLoginOutcome

JSON body of a successful `POST /api/auth/password`.

Type: [`PortalSessionIssued`](#portalsessionissued) or [`TotpChallenge`](#totpchallenge)

## PasswordLoginRequest

JSON body for local password login (login name or email plus password).

- `login` *string*, **required** — Login name or email.
- `password` *string*, **required** — Password verified against the stored hash; mismatch fails closed and counts toward lockout.

## PasswordSet

JSON body after a password was set.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `revoked_sessions` *integer (int64)*, **required** — Portal sessions of that user revoked by the change (the caller signs in again). At least 0.

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
- `may_connect_accounts` *boolean or null* — Whether the user may connect store accounts on this library, by hand or by bringing them from another library, when present.

## PathSanitizationMode

How to sanitize storage path segments when [`crate::OutputConfig::replacement_characters`]
is empty (explicit rules always win).

Type: one of `auto`, `windows`, `posix`, `s3`, `none`

## PinnedSchemaSummary

The settings schema pinned for a plugin, as the API renders it.

- `manifest_id` *string*, **required** — Manifest id (display alias) at pin time.
- `schema_sha256` *string*, **required** — Fingerprint of the normalized schema.
- `package_version` *string*, **required** — Semver package version that produced the pinned schema.
- `recorded_at` *string*, **required** — RFC 3339 time of the last pin.
- `recorded_by_node` *string*, **required** — Node that recorded the last pin.
- `settings` *array of [`SettingSpec`](#settingspec)*, **required** — Declared settings.

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

## PlanConflict

`POST /api/library/scan` and `POST /api/library/acquire` `409` body: the two conflicts a plan can hit.

Type: [`ScanPlanResponse`](#scanplanresponse) or [`ActionResponse`](#actionresponse)

## PluginArtifactSummary

One downloadable artifact of an installed version, as the API renders
it.

- `target` *string*, **required** — Bookclerk target (`linux-x64-gnu`, …; `any` for workerd packages).
- `url` *string*, **required** — Where nodes fetch the archive: `https:`, `file:`, or `blob:sha256:<hex>`.
- `archive_sha256` *string*, **required** — Lowercase hex SHA-256 of the archive bytes.
- `payload_root_sha256` *string*, **required** — Payload-root digest of the extracted plugin tree.

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
- `logo` *string or null* — Same-origin logo: the host-served embed path, or a 128 px copy of the remote logo the plugin names.

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

## PluginConsentPending

The difference between a plugin's request and its stored grant.

- `lines` *array of string*, **required** — Operator-facing lines naming every capability awaiting consent, the same lines `bookclerk plugins info` and `bookclerk plugins approve` print; empty when the stored grant covers the request.
- `domains` *array of string*, **required** — Fetch host patterns the request adds (`*` is any public host).
- `tcp` *array of [`TcpGrantView`](#tcpgrantview)*, **required** — TCP `host:ports` the request adds, narrowed to the new ports.
- `addressCidrs` *array of string*, **required** — CIDRs the request adds.
- `allowUndeclaredPublicRedirects` *boolean*, **required** — Whether the request newly asks to follow undeclared public redirects.

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
- `pending` *[`PluginConsentPending`](#pluginconsentpending) or null* — What the request asks for beyond the stored grant (a plugin upgrade that widened its capabilities); absent when no grant is stored. The plugin keeps running under the stored grant until these are approved.
- `brand` *[`PluginConsentBrand`](#pluginconsentbrand) or null* — Optional portal or manifest branding for the dialog.
- `limits` *[`PluginConsentLimits`](#pluginconsentlimits)*, **required** — Host-capped resource defaults (workerd budgets + shared disk). Always set.
- `host_user_agent` *string*, **required** — The `User-Agent` this host sends for the plugin's requests, except to the request's `ownUserAgentHosts`.

## PluginDeploymentChanged

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}` body: the
deployment row after the write.

- `deployment_id` *string*, **required** — Deployment row id.
- `plugin_instance_id` *string*, **required** — Instance the row belongs to.
- `node_id` *string*, **required** — Node the row targets.
- `desired` *[`DesiredState`](#desiredstate)*, **required** — Desired state now stored.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision after the write.

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

- `networkMode` *string or null* — Override egress mode (`deny` or `outbound`) when set.
- `domains` *array of string or null* — Override approved hostnames when set.
- `tcp` *array of [`TcpGrantView`](#tcpgrantview) or null* — Override raw TCP grants when set.
- `addressCidrs` *array of string or null* — Override address-space CIDRs when set.
- `allowUndeclaredPublicRedirects` *boolean or null* — Override undeclared-public-redirect permission when set.
- `ownUserAgentHosts` *array of string or null* — Override the hosts that keep the plugin's own `User-Agent` when set. Each entry must be one the request lists; leaving one out sends the host's agent there instead.
- `bindings` *array of string or null* — Override approved bindings when set. Entries are host binding names (`config`, `secrets`, `plugin_kv`, `work_fs`, `oauth`) and the manifest tokens the consent request lists (`database:<NAME>`, `queue:<BINDING>`, `queue_consumer:<queue>`, `cron:<schedule>`). A manifest token must match one in the request exactly; leaving one out narrows the grant.
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
- `networkMode` *string*, **required** — Approved egress mode (`deny` or `outbound`).
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
- `ownUserAgentHosts` *array of string or null* — Hosts (exact or `*.` wildcard) that receive the plugin's own `User-Agent`; every other request carries the host's agent. Absent on a grant approved before this permission existed, which keeps the hosts the plugin's manifest declares.
- `ownUserAgentReason` *string or null* — The plugin's reason for [`Self::own_user_agent_hosts`], from `plugin.toml`.
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

## PluginInstalled

`POST /api/config/plugins` body: what the install did.

- `created` *boolean*, **required** — True when the version row was written now; false when the same version with the same artifacts was installed already.
- `plugin_instance_id` *string*, **required** — Id of the plugin's default instance.
- `version` *[`PluginVersionSummary`](#pluginversionsummary)*, **required** — The installed version.

## PluginInstanceConfigChanged

`PUT /api/config/plugin-instances/{id}/config` body: the document after
the write.

- `plugin_instance_id` *string*, **required** — Instance id.
- `revision` *integer (int64)*, **required** — Revision after the write; unchanged when the same write was replayed.
- `settings` *map of string to [`SettingValue`](#settingvalue)*, **required** — Scalar settings now stored.
- `secret_refs` *array of [`InstanceSecretRefV1`](#instancesecretrefv1)*, **required** — Secret refs now stored, the newly sealed secrets included.

## PluginInstanceConfigResponse

`GET /api/config/plugin-instances/{id}/config` body: the stored document
and the configuration the guest receives.

- `plugin_instance_id` *string*, **required** — Instance id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision of the document.
- `settings` *map of string to [`SettingValue`](#settingvalue)*, **required** — Scalar settings the operator wrote.
- `secret_refs` *array of [`InstanceSecretRefV1`](#instancesecretrefv1)*, **required** — Secret refs; ciphertext stays in `encrypted_secrets`.
- `effective` *map of string to any JSON*, **required** — Settings as the guest sees them: schema defaults overlaid with the stored values. Empty when the plugin has no pinned schema.
- `schema` *array of [`SettingSpec`](#settingspec)*, **required** — Declared settings from the pinned schema; empty when there is none.

## PluginInstanceCreated

`POST /api/config/plugin-instances` body: the instance that was created.

- `plugin_instance_id` *string*, **required** — Id of the new instance.
- `plugin_key` *string*, **required** — Canonical plugin key the instance belongs to.
- `display_name` *string*, **required** — Operator-facing name, trimmed.
- `owner_node_id` *string or null* — Node the instance is local to; `null` for a cluster-wide instance.

## PluginInstanceDeleted

`DELETE /api/config/plugin-instances/{id}` body.

- `plugin_instance_id` *string*, **required** — Id of the instance that was removed.
- `deleted` *boolean*, **required** — Always true: removing an instance that is already gone answers the same way.

## PluginInstancesResponse

`GET /api/config/plugin-instances` body: every instance as this node
sees it.

- `node_id` *string*, **required** — Id of the node that answered; its deployments carry its own observations.
- `instances` *array of [`PluginInstanceView`](#plugininstanceview)*, **required** — Every plugin instance in the cluster.

## PluginInstanceView

One plugin instance with its schema, document, and deployments.

- `plugin_instance_id` *string*, **required** — Instance id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `alias` *string*, **required** — Manifest id recorded with the pinned schema (display alias).
- `display_name` *string*, **required** — Operator-facing name; empty for the default instance.
- `is_default` *boolean*, **required** — True when a bare plugin key or alias routes to this instance.
- `created_at` *string*, **required** — RFC 3339 creation time.
- `schema` *array of [`SettingSpec`](#settingspec)*, **required** — Declared settings from the pinned schema.
- `config` *[`InstanceConfigView`](#instanceconfigview) or null* — Current document, when the instance has one.
- `deployments` *array of [`PluginDeploymentView`](#plugindeploymentview)*, **required** — Deployment rows across nodes.
- `placement` *[`PlacementView`](#placementview) or null* — Scheduler placement; absent when the instance is hand-managed.

## PluginPlacementChanged

`PUT /api/config/plugin-instances/{id}/placement` body: the placement
after the write.

- `plugin_instance_id` *string*, **required** — Instance the scheduler now places.
- `placement` *[`PlacementView`](#placementview)*, **required** — The stored intent and the last scheduler pass's result.

## PluginPlacementDeleted

`DELETE /api/config/plugin-instances/{id}/placement` body.

- `plugin_instance_id` *string*, **required** — Instance that is hand-managed again.
- `deleted` *boolean*, **required** — True when a placement existed and was removed; false when the instance was hand-managed already.

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
- `logo` *string or null* — Same-origin logo for Settings list rows: the plugin's embedded logo, or a 128 px copy of the remote logo its manifest names.
- `settings` *array of [`PluginSettingOption`](#pluginsettingoption)*, **required** — Editable options for this plugin, including `.enabled`.
- `plugin_instance_id` *string or null* — Plugin instance this group edits; absent for a bootstrap database adapter.
- `display_name` *string or null* — Operator-facing instance name, when the operator set one.
- `is_default` *boolean* — True when a bare plugin key or alias routes to this instance.
- `config_revision` *integer (int64) or null* — Instance document revision for compare-and-swap writes.
- `deployment` *[`PluginDeploymentView`](#plugindeploymentview) or null* — This node's deployment of the instance, when one exists.
- `placement` *[`PlacementView`](#placementview) or null* — Scheduler placement; absent when the instance is hand-managed (the Enabled switch applies only then).
- `present_on` *array of string* — Nodes whose deployment of the instance is `present`.

## PluginUninstalled

`DELETE /api/config/plugins/{key}` body.

- `plugin_key` *string*, **required** — Canonical plugin key.
- `uninstalled` *boolean*, **required** — True when an active-version pointer existed and was cleared.
- `deployments_undeployed` *integer*, **required** — Deployment rows switched to `absent` across the plugin's instances. At least 0.

## PluginVersionApproved

`POST /api/config/plugins/{key}/versions/{version}/approve` body.

- `plugin_key` *string*, **required** — Canonical plugin key.
- `version` *string*, **required** — Semver version whose grant was approved.
- `approved` *boolean*, **required** — Always true: a version that cannot be approved is an error instead.
- `grant` *object*, **required** — The grant document, as the consent store keeps it: approved entrypoints, event producers and consumers, jobs, the network mode with the effective, manifest, operator-added and operator-denied domains, TCP targets and CIDRs, host bindings, compatibility flags, resource budgets and the approval time.

## PluginVersionSummary

One installed version of a plugin, as the API renders it.

- `plugin_key` *string*, **required** — Canonical plugin key.
- `id` *string*, **required** — Manifest id (display alias).
- `version` *string*, **required** — Semver package version.
- `active` *boolean*, **required** — True when this is the plugin's active version.
- `approved` *boolean*, **required** — True once an operator approved the version's consent request.
- `publisher_verified` *boolean*, **required** — True when the publisher was verified at install time.
- `coordinate` *string*, **required** — Package coordinate the version was resolved from; empty for a local archive.
- `installed_at` *string*, **required** — RFC 3339 install time.
- `installed_by` *string*, **required** — Audit identity of the installer.
- `host_payload_root_sha256` *string or null* — Payload-root digest of the artifact this host would run; `null` when no artifact fits the host's target.
- `artifacts` *array of [`PluginArtifactSummary`](#pluginartifactsummary)*, **required** — Per-target artifacts.

## PortalErrorBody

JSON body of every portal error response.

- `error` *string*, **required** — Operator-facing error text (no structured code).

## PortalMeInfo

Portal identity fields exposed on `/me` for SSO / local-portal callers.

- `identity_id` *integer (int64)*, **required** — `portal_identities.id` for this session.
- `provider` *string*, **required** — Identity-broker id (`local`, OIDC provider name, …).
- `external_user_id` *string*, **required** — Provider-stable subject / external user id.
- `label` *string or null* — Optional operator-facing label for the identity.

## PortalOk

JSON body of a portal route that has nothing to report beyond success.

- `ok` *boolean*, **required** — Always `true`.

## PortalSessionIssued

JSON body after a portal session was minted for a first-party user.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `role` *string*, **required** — The user's role (`owner`, `administrator`, `host`, or `member`).

## PortalSessionView

A portal (user) session as listed by `GET /api/auth/sessions`.

- `id` *integer (int64)*, **required** — Session id, for `DELETE /api/auth/sessions/{id}`.
- `created_at` *string*, **required** — RFC 3339 time the session was created.
- `expires_at` *string*, **required** — RFC 3339 expiry.
- `last_used_at` *string or null* — RFC 3339 time of the last authenticated use, when any.
- `is_current` *boolean*, **required** — Whether this is the session making the request.
- `client_label` *string or null* — Best-effort OS or client label from the User-Agent.
- `device_type` *string or null* — Best-effort device class (`desktop`, `mobile`, `tablet`, `api`).
- `user_agent` *string or null* — Raw User-Agent captured when the session was created.

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

## ProfileChanged

JSON body after `PATCH /api/auth/profile`.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `user` *[`AuthMeUser`](#authmeuser)*, **required** — The profile after the change, as `GET /api/auth/me` reports it.

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
- `incomplete` *boolean* — True when the batch deadline cut this lookup short: the hints are the query's trusted seeds alone, so a client should not keep the answer and asks again later. Omitted (false) for a finished lookup.

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
- `cache_budget_bytes` *integer (int64) or null* — This node's image cache budget in bytes (16 MiB to 1 TiB), overriding the cluster's `cache.budget_bytes`. `null` removes the override; leaving the field out keeps it as it is. At least 0.

## PutOutputDomainRequest

JSON body for `PUT /api/config/domains/core.output`.

- `expected_revision` *integer (int64)*, **required** — Revision the caller read. A mismatch is a conflict.
- `document` *[`OutputSettingsV1`](#outputsettingsv1)*, **required** — Complete replacement document (schema version 1).

## PutPluginDeployment

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}` body.

- `desired` *[`DesiredState`](#desiredstate)*, **required** — What the node should do: `present` or `absent` (the scheduler's `placed` / `unplaced` are accepted too).
- `expected_revision` *integer (int64) or null* — Revision the caller last observed; required when the row exists and the caller wants compare-and-swap. Omitted means "replace whatever is current".

## PutPluginInstanceConfig

`PUT /api/config/plugin-instances/{id}/config` body.

- `expected_revision` *integer (int64)*, **required** — Revision the caller last observed.
- `settings` *map of string to [`SettingValue`](#settingvalue)* — Scalar settings. Secret values are not accepted here.
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
- `picture_path` *string or null* — Same-origin 128 px copy of `picture_url` for the web app to show (filled by the daemon; never stored).

## RankedQueueEntry

Global wishlist queue entry ranked by overall/operator recommend taste +
heavy wish-count weight (shared order; not per-viewer personalized).

- `work_key` *string*, **required** — Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
- `work_id` *string*, **required** — The work every wish of this entry is for. Wishes for one work are one entry even when their `work_key`s differ, so a client matches its own wishes on this id first and on `work_key` otherwise.
- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
- `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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
- `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
- `subtitle` *string or null* — Optional bibliographic extras from storefront catalog / Audnexus.
- `description` *string or null* — Publisher or storefront synopsis; may be truncated for embeddings.
- `publisher` *string or null* — Publisher imprint as reported by the catalog.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `published_at` *string or null* — Publication date string from the catalog (ISO or storefront format).
- `genres` *string or null* — Candidate genres (not taste-seed `seed_categories`).
- `language` *string or null* — BCP-47 / storefront language code (`en`, `de`, …) when known.
- `wishers` *array of [`QueueWisher`](#queuewisher)* — Distinct people who have this title on an open wishlist.
- `wish_count` *integer (int64)* — Number of distinct users who wishlisted this title (may exceed `wishers.len()`).
- `seed_title` *string or null* — Title of the book or wish the storefront was asked about to find this card, when it came from such an expansion.
- `starter_genre` *string or null* — Starter genre this card was browsed from while the taste was thin.
- `available_through` *array of [`BranchAvailability`](#branchavailability)* — Branches of this library that currently list this title (they answered a catalog lookup for its work). Filled when the feed is read, never stored with it.

## RedeemBody

JSON body for `POST /redeem` (claim ticket plus retry nonce).

- `ticket` *string*, **required** — Raw claim-ticket token to redeem into a portal session.
- `nonce` *string*, **required** — Browser-generated one-time nonce persisted across HTTP retries.
- `password` *string or null* — Required when the linked local user has no password (invite / reset).

## RemoteConsentView

An owner library's request to run a plugin on this install, and what the operator answered.

- `cluster_id` *string*, **required** — The owner library's cluster id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `requested_version` *string*, **required** — Version the owner deploys here now.
- `requested_grant` *any JSON*, **required** — A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.
- `requested_artifact` *[`RequestedArtifact`](#requestedartifact)*, **required** — The archive the owner deploys to this install.
- `request_digest` *string*, **required** — Digest of the request as a whole; an approval names it so a changed request is not approved by mistake.
- `approved_version` *string or null* — Version this install's operator approved, when any.
- `grant` *any JSON*, **required** — A plugin grant document (the plugin host's `PluginGrant`): the plugin key, the entrypoints, network mode and domains, and the bindings it may use; `null` when nothing was approved.
- `approved_at` *string or null* — RFC 3339 approval time.
- `approved_by` *string or null* — Who approved (`operator`, or `platform` for an automatic approval).
- `revoked_at` *string or null* — RFC 3339 time the operator revoked the approval.
- `updated_at` *string*, **required** — RFC 3339 time of the last write.
- `state` *string*, **required** — How this host stands on the request: `consented`, `awaiting_host_consent` (no approval, or a revoked one), `awaiting_host_reapproval` (the owner changed the request or the archive since the approval), or `invalid_grant` (a grant that does not parse).
- `pending` *array of string*, **required** — What the owner's current request asks for beyond the request the operator approved, as the operator-facing lines the CLI prints; empty when nothing was approved yet or the request stays within the approval.

## RemoveBranchTitles

`POST /api/config/branches/titles/remove` body.

- `node_id` *string or null* — Only this branch's entries; every branch's when absent.

## ReplacementRule

Character replacement rule (`ReplacementCharacters` in classic Settings.json).

- `find` *string*, **required** — Substring to search for in each path segment / tag value.
- `replace` *string*, **required** — Replacement text (may be empty to strip `find`).

## RequestedArtifact

The archive an owner library deploys to this install.

- `url` *string*, **required** — Where the owner fetches it from.
- `archive_sha256` *string*, **required** — SHA-256 of the archive.
- `payload_root_sha256` *string*, **required** — Payload-root digest of the tree it unpacks to.

## RequestStatus

Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).

Type: one of `open`, `cancelled`

## RevisionConflict

Body of a `409 Conflict` from a compare-and-swap write whose
`expected_revision` lost to a newer revision.

- `error` *string*, **required** — Always `revision_conflict`.
- `current_revision` *integer (int64)*, **required** — The revision to read and retry against.

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

## SchemaStateView

Whether the database carries exactly the schema this binary applies.

The object example is the successful read; a failed read has `state`
null, `compatible` false and the `error` text.

- `state` *string or null* — The schema state (`uninitialized`, `unreleased@base<n>+<checksum>` or `frozen@<n>+<checksum>`); `null` when it could not be read.
- `compatible` *boolean*, **required** — True when this binary can run against that schema.
- `error` *string or null* — Why the state could not be read; omitted when it was.

## SessionsResponse

JSON body for `GET /api/auth/sessions`.

- `sessions` *array of [`SessionView`](#sessionview)*, **required** — The caller's sessions, newest first; empty for a Bearer-only caller, who has no session rows.

## SessionView

One session in the caller's list, told apart by `kind`: an operator
session or a portal (user) session.

Type: [`OperatorSessionView`](#operatorsessionview) and object or [`PortalSessionView`](#portalsessionview) and object

## SetPasswordRequest

JSON body for setting a password (self or provisioner target).

- `password` *string*, **required** — New password; hashed before persist. Empty is rejected.
- `current_password` *string or null* — Required when the caller already has a password (not first-time invite setup).
- `user_id` *integer (int64) or null* — Target `users.id` when a provisioner sets another user's password.

## SettingKind

Value type of a declared plugin setting.

Type: one of `string`, `bool`, `integer`, `enum`, `secret`

## SettingNumber

A finite JSON number; an integer for integer settings.

Type: number

## SettingSpec

One setting a plugin declares in its `plugin.toml` `[[settings]]`:
its key, value type, default and help text.

- `key` *string*, **required** — Setting key (`[a-z][a-z0-9_]*`, at most 64 bytes).
- `label` *string or null* — Operator-facing label; omitted when the key is shown.
- `type` *[`SettingKind`](#settingkind)*, **required** — Value type: `string`, `bool`, `integer`, `enum` or `secret`.
- `values` *array of [`SettingValueSpec`](#settingvaluespec) or null* — Allowed values of an `enum` setting; omitted for other kinds.
- `default` *any JSON* — Default applied when the instance document has no value; omitted when there is none. A string, boolean or integer, as the setting's `type` says.
- `required` *boolean or null* — True when the instance cannot run without a value; omitted when false.
- `description` *string or null* — Help text shown beside the setting; omitted when there is none.

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

## SettingValue

One scalar setting. Nested objects and arrays are not representable.

Type: boolean or [`SettingNumber`](#settingnumber) or string

## SettingValueSpec

One allowed value of an `enum` setting: the stored id and its label.

- `id` *string*, **required** — Stored value.
- `label` *string or null* — Operator-facing label; omitted when the id is shown.

## SharingBody

`POST /api/config/join/sharing` body: one library and the settings to
change for it.

- `cluster_id` *string*, **required** — The library (owner cluster id).
- `share_books` *boolean*, **required** — "Share this branch's books with *library*".
- `include_branches` *boolean*, **required** — "Include books from my own branches".
- `list_members_books` *boolean or null* — "List my members' books to *library*"; absent or `null` keeps the stored setting.
- `work_for_members` *boolean or null* — "Download for other members of *library*": take the library's work for its other members too, not only this install owner's. Absent or `null` keeps the stored setting.
- `keep_member_copies` *boolean or null* — "Keep copies of other members' books": also write a book acquired here for another member to this install's own destinations. Absent or `null` keeps the stored setting.

## SharingChanged

JSON body after `POST /api/config/join/sharing`.

- `cluster` *[`ClusterLinkView`](#clusterlinkview)*, **required** — The link with its settings as stored now.

## SharingSettings

JSON body for `GET /api/config/join/sharing`.

- `clusters` *array of [`ClusterLinkView`](#clusterlinkview)*, **required** — Every library this install is a branch of, with its sharing and work settings.

## ShelfKindInfo

A shelf kind the operator can ignore (all offered by default).

- `id` *string*, **required** — Stable kind id (`finish_series`, `author`, `genre`, `from_store`, …).
- `label` *string*, **required** — Operator-visible label for this account or item.

## SigninIntegration

One integration that signs people in with its own credentials.

- `id` *string*, **required** — Integration id, used in `/api/integrations/{id}/login`.
- `name` *string*, **required** — Button label.

## SigninMethods

JSON body for `GET /api/auth/signin`: the ways this library lets people sign in.

- `operator_token` *boolean*, **required** — Whether the operator token may be entered in the SPA (only before an Owner exists).
- `oidc` *array of [`SigninOidcProvider`](#signinoidcprovider)*, **required** — Enabled OIDC identity providers, in configuration order.
- `integrations` *array of [`SigninIntegration`](#signinintegration)*, **required** — Integrations that accept their own credentials at sign-in.
- `require_second_factor` *boolean*, **required** — Whether `daemon.auth.require_second_factor` is on, so clients can explain the enrollment step.

## SigninOidcProvider

One OIDC identity provider offered on the sign-in page.

- `id` *string*, **required** — Provider id from `[auth.oidc.providers]`, used in `/api/auth/oidc/{id}/start`.
- `name` *string*, **required** — Button label.
- `preset` *string or null* — Known social preset the client may style (`google`, `github`, …), when the provider matches one.

## SourceConfigOptionInfo

One storefront config option (key, label, allowed values).

- `key` *string*, **required** — Config table key (for example `container`).
- `label` *string*, **required** — Operator-facing option name.
- `values` *array of [`ConfigOptionValueInfo`](#configoptionvalueinfo)*, **required** — Allowed values the SPA may offer for this option.

## SourceConnected

JSON body of a connected store account.

- `ok` *boolean*, **required** — Always `true`.
- `account_id` *string*, **required** — The store's id for the account.
- `source` *string*, **required** — The source's id, as listed by `GET /api/portal/sources`.
- `plugin_key` *string*, **required** — The plugin that owns the account.

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
- `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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
- `title` *string*, **required** — Display title: the title of the wish's work.
- `authors` *string or null* — Comma-separated authors: the work's author credits in billing order.
- `asin` *string or null* — Amazon ASIN from the wish's storefront sources, when one carries it.
- `isbn` *string or null* — ISBN-10/13 from the wish's storefront sources, when one carries it.
- `notes` *string or null* — Free-form operator or requester notes.
- `status` *[`RequestStatus`](#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
- `work_key` *string* — Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use to match a wish to a catalog card. Derived from the work's title and authors and the sources' identifiers on every read, never stored.
- `work_id` *string*, **required** — The work the wish is for; wishes for one work are one queue entry.
- `edition_id` *string or null* — The edition the wish named, when the library knows it.
- `resolved_book_uuid` *string or null* — Library `books.uuid` once the wishlist item is fulfilled.
- `cover_url` *string or null* — HTTPS URL for cover art when known.
- `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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
- `sort_by` *string*, **required** — Remote sort key for reviews (`MostHelpful` or `MostRecent`, as applied).
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

## TotpChallenge

Password accepted, but an authenticator-app code is needed before a session is issued.

- `ok` *boolean*, **required** — Always `true`: the password step succeeded.
- `mfa` *[`TotpChallengeDetails`](#totpchallengedetails)*, **required** — The second step the client must complete.

## TotpChallengeDetails

The pending second-factor step after a password login.

- `method` *string*, **required** — Always `totp`: finish with `POST /api/auth/totp/login`.
- `challenge_id` *string*, **required** — Challenge to send with the code; expires after five minutes.

## TotpEnrollment

JSON body after TOTP enrollment began: what the authenticator app needs.

- `secret` *string*, **required** — Pending base32 secret, for typing into an app by hand.
- `otpauth_url` *string*, **required** — `otpauth://` URL the QR code encodes.
- `qr_svg` *string*, **required** — The QR code as an SVG document, ready to inline.

## TotpReauthBody

Optional current password for enroll/disable step-up.

- `current_password` *string or null* — Password used for recent-reauth when the user already has a credential.

## TotpStatus

JSON body for `GET /api/auth/totp`.

- `enabled` *boolean*, **required** — Whether an authenticator app is confirmed for the caller.

## TransportRefused

JSON body of a `403` from the node API's transport policy: the
connection is plain HTTP off loopback, or arrived through a proxy the
policy does not trust, and `reason` names the check that refused it.

- `error` *string*, **required** — Always `transport_insecure`.
- `reason` *string*, **required** — The transport check that refused the connection (`plain_http`, `unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`, `proxy_proto_ambiguous`, `proxy_proto_not_https`, `public_origin_not_https`, `listener_mode_mismatch`, `node_api_misconfigured`).

## TrayHandoffPrepareResponse

JSON returned to the tray after a successful prepare.

- `code` *string*, **required** — Single-use loopback handoff code for `GET /api/auth/tray-handoff?code=`.
- `expires_in_secs` *integer (int64)*, **required** — Seconds until this ticket expires (matches the in-process deadline). At least 0.

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

## UserinfoResponse

Standard OIDC claims for `GET /oidc/userinfo`, limited to the token's scopes.

- `sub` *string*, **required** — Subject: the Bookclerk `users.id` as a string.
- `name` *string or null* — Display name (`profile` scope), when the user has one.
- `preferred_username` *string or null* — Login name (`profile` scope), when the user has one.
- `email` *string or null* — Email (`email` scope), when the user has one.
- `email_verified` *boolean or null* — Always `true` when `email` is present: Bookclerk trusts the address the Owner recorded.

## UserIntegrationHint

Storefront / integration connection summary for admin user lists.

- `source` *string*, **required** — Content-source or integration plugin id (`audible`, `libro`, …).
- `account_id` *string*, **required** — Store or operator account id this row belongs to.
- `label` *string or null* — Optional operator-facing account label.

## UserListeningHint

Recent unfinished listening hint for admin user lists.

- `title` *string or null* — Title of the work being listened to, when the library knows it.
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
- `may_connect_accounts` *boolean*, **required** — Whether the operator lets the user connect store accounts here, by hand or by bringing them from another library. True unless an administrator turned it off for this user.
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
- `picture_path` *string or null* — Same-origin 128 px copy of `picture_url` for the web app to show, when this node can sign it.
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
