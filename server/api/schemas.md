# Schemas

The request and response bodies of the [web API](README.md), in alphabetical order.

## AbridgementJson

Whether an expression is abridged.

Type: one of `unknown`, `unabridged`, `abridged`

## AccountRef

One store account, by storefront plugin key and store account id.

- `plugin_key` *string*, **required** — Provenance-qualified plugin key of the storefront, as sent.
- `account_id` *string*, **required** — Store account id, as sent.

## AcquireRequestBody

Optional body for `POST /api/library/acquire`.

- `asin` *string or null* — Audible ASIN filter; first of uuid/asin/isbn/product_id wins.
- `uuid` *string or null* — Library title UUID filter.
- `isbn` *string or null* — ISBN filter for non-Audible stores.
- `product_id` *string or null* — Store product id filter.
- `account` *string or null* — Restrict acquire to this store account.
- `plugin_instance_id` *string or null* — When set, fetch through this deployed plugin instance.
- `titles` *array of string* — More title ids; each (with the single filter above) becomes its own job.
- `editions` *array of string* — Editions to acquire (`id` of an edition in a work's detail). Each becomes one job for a title that sells it in an account the caller may acquire for (in `account` when that is set); the edition's titles share its files.
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

## AddConnectionsBody

`POST /api/config/join/connections` body.

- `library_id` *string*, **required** — The library (parent library id) to add the connections to.
- `accounts` *array of [`AccountRef`](#accountref)*, **required** — The person's accounts to add; 1 to 64.

## AddressFieldJson

One field a patron fills in for a personal destination.

- `key` *string*, **required** — Key in the address document.
- `label` *string*, **required** — Label shown on the form.
- `required` *boolean*, **required** — Whether a destination must carry it.
- `secret` *boolean*, **required** — Whether the value is a secret: shown masked, never logged.

## AdminControlOverrides

One Administrator's overrides: a value for some switches, and the
library's default for the rest (an absent or `null` field).

- `manage_members` *boolean or null* — Overrides [`AdminControls::manage_members`].
- `set_patron_limits` *boolean or null* — Overrides [`AdminControls::set_patron_limits`].
- `library_scan` *boolean or null* — Overrides [`AdminControls::library_scan`].
- `run_jobs` *boolean or null* — Overrides [`AdminControls::run_jobs`].
- `fix_catalog` *boolean or null* — Overrides [`AdminControls::fix_catalog`].
- `invite_branches` *boolean or null* — Overrides [`AdminControls::invite_branches`].
- `approve_requests` *boolean or null* — Overrides [`AdminControls::approve_requests`].

## AdminControls

A value for each switch: the library's default, or what applies to one
Administrator.

- `manage_members` *boolean* — May manage Members ([`AdminControl::ManageMembers`]).
- `set_patron_limits` *boolean* — May set patron limits ([`AdminControl::SetPatronLimits`]).
- `library_scan` *boolean* — May scan and acquire library-wide ([`AdminControl::LibraryScan`]).
- `run_jobs` *boolean* — May run the job queue ([`AdminControl::RunJobs`]).
- `fix_catalog` *boolean* — May fix the catalog ([`AdminControl::FixCatalog`]).
- `invite_branches` *boolean* — May invite branches ([`AdminControl::InviteBranches`]).
- `approve_requests` *boolean* — May approve interlibrary requests ([`AdminControl::ApproveRequests`]).

## AdminSwitchChoice

What a change does to one switch of one Administrator.

Type: one of `on`, `off`, `default`

## AfterIngest

What happens to a drop folder item once it has been ingested.

Type: one of `move`, `delete`, `leave`

## ApiError

An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.

- `error` *string*, **required** — Stable machine slug.
- `message` *string* — Explanation for people.
- `status` *integer* — The HTTP status, repeated.

## ApproveBody

`POST /api/config/library-consents/approve` body.

- `library_id` *string*, **required** — Library id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `request_digest` *string*, **required** — The `request_digest` of the request the operator reviewed, as the listing shows it (refused when the recorded request is another one).
- `deny_domains` *array of string* — Domains to take out of the request. No duplicates.
- `deny_bindings` *array of string* — Bindings to take out of the request. No duplicates.
- `deny_own_user_agent_hosts` *array of string* — Requested hosts to refuse the plugin's own User-Agent on; requests to them carry Bookclerk's agent instead. No duplicates.
- `deny_network` *boolean* — Deny all network access.

## AssertionFinish

`POST /api/auth/passkeys/login/finish` and `elevate/finish` body: the
browser's assertion plus the challenge id from the matching `begin`.

The credential is read straight into WebAuthn's typed assertion, so the
handler never decodes caller JSON itself.

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — W3C PublicKeyCredential as the browser serializes it: a registration attestation (`response.attestationObject`) or an authentication assertion (`response.authenticatorData`, `signature`, `userHandle`), binary fields base64url-encoded.

## AudioFile

An audio file of a book.

- `index` *integer (int64)*, **required** — The file's place in play order, from 1. At least 0.
- `ino` *string*, **required** — The file's `ino` ([`crate::ids::file_ino`]).
- `metadata` *[`FileMetadata`](#filemetadata)*, **required** — The file's name, size and times.
- `addedAt` *integer (int64)*, **required** — When the file was stored, in epoch milliseconds.
- `updatedAt` *integer (int64)*, **required** — When the file was stored, in epoch milliseconds.
- `trackNumFromMeta` *integer (int64) or null* — The track number the file's tags state: not read. At least 0.
- `discNumFromMeta` *integer (int64) or null* — The disc number the file's tags state: not read. At least 0.
- `trackNumFromFilename` *integer (int64) or null* — The track number in the file name: not read. At least 0.
- `discNumFromFilename` *integer (int64) or null* — The disc number in the file name: not read. At least 0.
- `manuallyVerified` *boolean*, **required** — Always false.
- `exclude` *boolean*, **required** — Always false.
- `error` *string or null* — Always null.
- `format` *string or null* — The container format, such as `mp4` or `mp3`.
- `duration` *number (double)*, **required** — The file's length in seconds.
- `bitRate` *integer (int64) or null* — Bits per second, if known. At least 0.
- `language` *string or null* — The file's language, if known.
- `codec` *string or null* — The codec, if known.
- `timeBase` *string or null* — The time base, if known.
- `channels` *integer (int64) or null* — Audio channels, if known. At least 0.
- `channelLayout` *string or null* — The channel layout, if known.
- `chapters` *array of [`Chapter`](#chapter)*, **required** — Chapters embedded in the file: none are read.
- `embeddedCoverArt` *string or null* — Always null.
- `metaTags` *[`MetaTags`](#metatags)*, **required** — Tags read from the file: none until the stored media probe reads them.
- `mimeType` *string*, **required** — The file's media type, such as `audio/mp4`.

## AudioTrack

A track with its place in the book.

- `index` *integer (int64)*, **required** — The track's place in play order, from 1. At least 0.
- `startOffset` *number (double)*, **required** — Where the track starts in the book, in seconds.
- `duration` *number (double)*, **required** — The track's length in seconds.
- `title` *string*, **required** — The track's title: its file name.
- `contentUrl` *string*, **required** — The path the track streams from, under the API's base address (`/api/items/{id}/file/{ino}`); apps add `?token=`.
- `mimeType` *string*, **required** — The track's media type.
- `codec` *string or null* — The codec, if known.
- `metadata` *[`FileMetadata`](#filemetadata) or null* — The file's name, size and times.

## AuthFormData

Text an app shows on its sign-in screen.

- `authLoginCustomMessage` *string or null* — A note under the sign-in form: where to get a player password.
- `authOpenIDButtonText` *string*, **required** — The label of the browser sign-in button.
- `authOpenIDAutoLaunch` *boolean*, **required** — Whether apps open the browser sign-in without a tap: never.

## AuthMeImpersonating

Target user shown while an operator session is impersonating.

- `user_id` *integer (int64)*, **required** — First-party `users.id` being impersonated.
- `display_name` *string or null* — Display name of the impersonated user when the row exists.

## AuthMeResponse

JSON body for `GET /api/auth/me`.

- `authenticated` *boolean*, **required** — Whether the request resolved to an operator, portal, or impersonated session.
- `role` *string or null* — `operator`, `owner`, `administrator`, or `member` (legacy clients may still send `portal`).
- `default_view` *string*, **required** — Configured post-auth landing view (`discover` / `library` / `accounts` / `wishlist`). Loaded from per-user DB preferences (not config.toml).
- `can_acquire` *boolean*, **required** — Whether this session may acquire / scan / manage jobs. True for operator and administrator.
- `elevated` *boolean*, **required** — True when this operator session was created via Owner elevate.
- `administers_nodes` *boolean*, **required** — Whether this session reaches the node administration routes and the library Status page: the operator, an Owner or Administrator, or a Member who owns at least one node (a branch they linked).
- `impersonating` *[`AuthMeImpersonating`](#authmeimpersonating) or null* — Present when the operator is impersonating a User.
- `portal` *[`PortalMeInfo`](#portalmeinfo) or null* — Linked portal identity when the caller is a portal (or impersonated) user.
- `user` *[`AuthMeUser`](#authmeuser) or null* — First-party user when the session is linked to a `users` row.
- `second_factor` *[`AuthSecondFactor`](#authsecondfactor) or null* — Password-login second-factor policy and this user's enrollment.
- `see_prices` *boolean*, **required** — Whether the caller's "See prices and store links" member control allows prices and store links. The library already drops them from its answers when it does not; the app uses this to show no store links of its own either. Always true for the operator.

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

## Author

An author.

- `id` *string*, **required** — The contributor id.
- `asin` *string or null* — Always null.
- `name` *string*, **required** — The author's name.
- `lastFirst` *string*, **required** — The name family name first.
- `description` *string or null* — Always null.
- `imagePath` *string or null* — Always null: no author images.
- `libraryId` *string*, **required** — The library it is listed in.
- `addedAt` *integer (int64)*, **required** — When the library first saw the author, in epoch milliseconds.
- `updatedAt` *integer (int64)*, **required** — When the record last changed, in epoch milliseconds.
- `numBooks` *integer (int64)*, **required** — Items by the author the patron can see. At least 0.
- `libraryItems` *array of [`LibraryItem`](#libraryitem) or null* — The author's items, when asked for.
- `series` *array of [`AuthorSeries`](#authorseries) or null* — The author's series with their items, when asked for.

## Authors

`GET /api/libraries/{id}/authors` without paging.

- `authors` *array of [`Author`](#author)*, **required** — The authors.

## AuthorSeries

A series of an author's, with the author's items in it.

- `id` *string*, **required** — The series id.
- `name` *string*, **required** — The series name.
- `items` *array of [`LibraryItem`](#libraryitem)*, **required** — The items, in series order.

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

## BatchGetBody

`POST /player/api/items/batch/get`.

- `libraryItemIds` *array of string*, **required** — The items, at most 100.

## BinaryBody

A request body of raw bytes (a file upload), documented as
`type: string, format: binary` so generated clients send the bytes
rather than a JSON array.

Type: binary

## Bookmark

A bookmark.

- `libraryItemId` *string*, **required** — The item it is in.
- `title` *string*, **required** — Its title.
- `time` *number (double)*, **required** — Where it is, in seconds.
- `createdAt` *integer (int64)*, **required** — When it was made, in epoch milliseconds.

## BookmarkBody

`POST` and `PATCH /player/api/me/item/{id}/bookmark`.

- `time` *number (double)*, **required** — The place, in seconds; on `PATCH`, which bookmark to rename.
- `title` *string or null* — The bookmark's title.

## BookmarkEditBody

`PUT /api/playback/bookmarks/{id}` body: what to change. A field left
out keeps its stored value.

- `position_ms` *integer (int64) or null* — The new place, in milliseconds. At least 0.
- `title` *string or null* — The new note; an empty string clears it.

## BookmarkJson

A place the caller marked in an edition.

- `id` *string*, **required** — Bookmark id.
- `edition_id` *string*, **required** — The edition it is in.
- `position_ms` *integer (int64)*, **required** — The marked place, in milliseconds from the edition's start. At least 0.
- `title` *string*, **required** — The caller's note; may be empty.
- `created_at` *string*, **required** — When it was made (RFC 3339).

## BookmarksJson

`GET /api/library/editions/{id}/bookmarks` answer.

- `bookmarks` *array of [`BookmarkJson`](#bookmarkjson)*, **required** — The caller's bookmarks in the edition, in timeline order.

## BookMedia

A book's media: metadata, tracks and chapters.

- `id` *string*, **required** — The media id: the edition id.
- `libraryItemId` *string or null* — The item id, when fetched expanded.
- `metadata` *[`BookMetadata`](#bookmetadata)*, **required** — The book's metadata.
- `coverPath` *string or null* — A cover path apps only test for presence; covers are fetched from `/api/items/{id}/cover`.
- `tags` *array of string*, **required** — Tags: none.
- `numTracks` *integer (int64)*, **required** — Tracks in play order. At least 0.
- `numAudioFiles` *integer (int64)*, **required** — Audio files. At least 0.
- `numChapters` *integer (int64)*, **required** — Chapters. At least 0.
- `duration` *number (double)*, **required** — The book's length in seconds.
- `size` *integer (int64)*, **required** — Bytes in its audio files. At least 0.
- `audioFiles` *array of [`AudioFile`](#audiofile) or null* — The audio files, when fetched expanded.
- `chapters` *array of [`Chapter`](#chapter) or null* — The chapters, when fetched expanded.
- `tracks` *array of [`AudioTrack`](#audiotrack) or null* — The tracks with their place in the book, when fetched expanded.

## BookMetadata

A book's metadata.

- `title` *string*, **required** — The title.
- `titleIgnorePrefix` *string*, **required** — The title for sorting.
- `subtitle` *string or null* — The subtitle, if any.
- `authors` *array of [`NamedRef`](#namedref)*, **required** — The authors.
- `authorName` *string*, **required** — The authors' names joined with `, `.
- `authorNameLF` *string*, **required** — The authors' names, family name first, joined with `, `.
- `narrators` *array of string*, **required** — The narrators' names.
- `narratorName` *string*, **required** — The narrators' names joined with `, `.
- `series` *array of [`SeriesRef`](#seriesref)*, **required** — The series the book is in.
- `seriesName` *string*, **required** — The series names with the book's place (`Name #2`), joined.
- `genres` *array of string*, **required** — Genres.
- `publishedYear` *string or null* — The year of publication, as a string (`"2019"`).
- `publishedDate` *string or null* — The date of publication, if known.
- `publisher` *string or null* — The publisher.
- `description` *string or null* — The description, as the library stores it.
- `descriptionPlain` *string or null* — The description with markup removed.
- `isbn` *string or null* — The ISBN, if known.
- `asin` *string or null* — The ASIN, if known.
- `language` *string or null* — The language.
- `explicit` *boolean*, **required** — Always false.
- `abridged` *boolean*, **required** — Whether the edition is abridged.

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
- `shared_branch_offer` *string or null* — `1` when the joining install offers to join as a shared branch ([`bookclerk_node_proto::SHARED_BRANCH_OFFER_PARAM`]).

## BranchAvailability

One branch a title is available through.

- `work_id` *string*, **required** — This library's work the branch supplies: the `work_id` to send `POST /api/branch-requests` to ask the branch for it.
- `node_id` *string*, **required** — The branch node id.
- `owner_name` *string or null* — The branch owner's display name, for "available through Riley's branch"; absent when the owner has none.
- `store` *string*, **required** — The store the branch has the book from.
- `product_id` *string*, **required** — The store's product id of the branch's copy.

## BranchConsentBody

`POST /api/auth/branch-consent` body.

Includes every field of [`BranchAuthorizeParams`](#branchauthorizeparams).

- `approve` *boolean*, **required** — True to let the branch join; false to deny.
- `connection` *string or null* — `branch` (default) or `shared_branch`.
- `step_up` *[`StepUp`](#stepup) or null* — The fresh second factor a shared branch needs.
- `bring_connections` *boolean* — True to send the user's store connections that may travel to the branch when it joins. Ignored for a shared branch. Defaults to false.

## BranchConsentPage

JSON body for `GET /api/auth/branch-consent`: what the consent page shows.

- `library_name` *string*, **required** — This library's name.
- `branch_name` *string*, **required** — The name the branch asks to join as.
- `redirect_origin` *string*, **required** — Origin of the branch's redirect URI, for the page to name.
- `enrollment` *boolean*, **required** — Whether this library lets new branches join right now.
- `user` *[`ConsentingUser`](#consentinguser)*, **required** — The signed-in user the branch would belong to.
- `connection` *[`ConnectionChoice`](#connectionchoice)*, **required** — The connection types the page may offer.
- `connections` *array of [`ConsentConnection`](#consentconnection)*, **required** — The user's store connections here, and whether each would travel to the branch if the user chooses to bring them.

## BranchEnrollment

The branch-enrollment switch.

- `enabled` *boolean*, **required** — Whether branch nodes may enroll into this library.
- `revision` *integer (int64) or null* — Stored revision; `null` while the default (enabled) applies.

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

- `library` *[`LibraryLinkView`](#librarylinkview)*, **required** — The new link.
- `library_name` *string or null* — The library's name, when the join came from a sign-in (not an invite link).
- `running` *boolean*, **required** — Whether the link's client is running now.
- `connections` *[`BroughtConnectionsView`](#broughtconnectionsview) or null* — What became of the store connections the library's user chose to bring to this branch; left out when none came.
- `replaced_card` *[`ReplacedCard`](#replacedcard) or null* — Set when this install held a library card for the library: the branch replaced it. Whether the card is gone here and whether the library ended its session are in [`ReplacedCard`]. Left out when there was none.

## BranchJoinSetting

JSON body after `PUT /api/config/branches`.

- `enrollment` *boolean*, **required** — "Let new branches join", as stored.

## BranchMembersBooks

JSON body after `PUT /api/config/branches/members-books`.

- `node_id` *string*, **required** — The branch.
- `acquire_members_books` *boolean*, **required** — "Acquire all of this branch's members' books", as stored.

## BranchOnlyChoice

The consent page's connection block when only a branch is offered.

- `choice` *boolean*, **required** — Always `false`.
- `default` *string*, **required** — Always `branch`.

## BranchRelationship

How a node relates to the library, as people are told: a shared branch
serves the library from its database, and a branch with its own
database is a remote branch when it reports a public address (it is a
core branch other installs can reach) and a local branch when it does
not.

Type: one of `shared_branch`, `remote_branch`, `local_branch`

## BranchRequestBody

`POST /api/branch-requests` body: the work to ask a branch for, and
optionally which branch.

- `work_id` *string*, **required** — This library's work id, as Discover and search name it.
- `node_id` *string or null* — The branch to ask, from the work's `available_through`; absent or `null` asks the first branch that lists the work.

## BranchRequestQueued

`POST /api/branch-requests` answer.

- `job_id` *string*, **required** — The request's job (kind `branch_request`), as `/api/jobs` lists it.
- `node_id` *string*, **required** — The branch the request went to.
- `already_queued` *boolean*, **required** — `true` when a request for this work to this branch was already waiting or being taken; `job_id` is that request.

## BranchTitleCounts

Counts of one branch's shared titles.

- `shared` *integer (int64)*, **required** — Titles it shares now. At least 0.
- `delivered` *integer (int64)*, **required** — Of those, titles whose files arrived. At least 0.
- `detached` *integer (int64)*, **required** — Editions whose files it delivered that nothing in this library holds any more (it stopped sharing, left, or withdrew them, and no store account or other branch here holds them). At least 0.

## BranchTitlesRemoved

JSON body after detached branch titles were removed.

- `removed` *integer (int64)*, **required** — How many title entries were removed. At least 0.

## BranchView

One branch (branch node) of this library.

- `node_id` *string*, **required** — The branch's node id.
- `created_at` *string*, **required** — RFC 3339 time the node was created.
- `owner_user_id` *integer (int64) or null* — The user who owns the branch.
- `owner_name` *string or null* — That user's display or login name.
- `joined` *boolean*, **required** — Whether its credential is still valid (it has not left or been revoked).
- `share_books` *boolean or null* — "Share this branch's books" as it last reported it; `null` until its first catalog sync.
- `include_branches` *boolean or null* — "Include books from my own branches" as it last reported it; `null` until its first catalog sync.
- `acquire_members_books` *boolean*, **required** — This library's "Acquire all of this branch's members' books": the branch also sends the books its members acquired that it lists here, not only its owner's.
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
- `connection` *string or null* — The connection type the library's user chose (`shared_branch`; a branch when left out). The code is bound to it, so a changed value only fails the redeem.

## CallbackParams

Query or form_post fields from the IdP callback.

- `code` *string or null* — Authorization code to exchange; required unless `error` is set.
- `state` *string or null* — CSRF `state` that must match the `bookclerk_oidc_tx` cookie.
- `error` *string or null* — IdP-reported deny/error; redirects the browser to `/?sso_error=denied`.
- `user` *string or null* — Apple first-login name/email JSON (form_post only).

## CapacityBody

`POST /api/config/join/capacity` body: one library and the job cap to
set for it.

- `library_id` *string*, **required** — The library (parent library id).
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
- `is_explicit` *boolean or null* — When `Some(true)`, a storefront marks the edition explicit (adult) content; `None` when none said.
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

## ChannelList

The library's channels.

- `channels` *array of [`DestinationJson`](#destinationjson)*, **required** — Oldest first.

## Chapter

A chapter of a book.

- `id` *integer (int64)*, **required** — The chapter's place, from 0. At least 0.
- `start` *number (double)*, **required** — Where it starts, in seconds.
- `end` *number (double)*, **required** — Where it ends, in seconds.
- `title` *string*, **required** — Its title.

## ChapterAlignJson

A queued chapter alignment.

- `job_id` *string*, **required** — The `chapter_align` job that does the work; it answers through the jobs API. An alignment already queued for the edition is returned rather than a second one.

## ChapterClaimJson

One source's chapter claim for an edition.

- `source_kind` *[`ChapterSourceKind`](#chaptersourcekind)*, **required** — The kind of source.
- `source_id` *string*, **required** — Which source of that kind (a PluginKey; empty for an edit or for Bookclerk's own processing).
- `quality` *string*, **required** — Where a machine source ranks within its kind: `strong_accurate`, `strong_estimated` or `weak` for a provider, `chapters`, `tracks` or `unknown` for a store listing; empty otherwise.
- `digest` *string*, **required** — SHA-256 of the claimed list, 64 lower-case hex characters.
- `chapters` *array of [`EditionChapter`](#editionchapter) or null* — The claimed chapters; absent while the list's write is unfinished.
- `runtime_ms` *integer (int64) or null* — Length of the timeline the list was measured on, in milliseconds. At least 0.
- `input_digest` *string or null* — For a processed claim, the digest of the provider list it was aligned from.
- `plays` *boolean*, **required** — Whether the edition plays this claim now.
- `pinned_at` *string or null* — When an Administrator pinned it (RFC 3339); absent when unpinned.
- `created_by_user_id` *integer (int64) or null* — The person whose edit it is; absent for a machine source or the operator.
- `updated_at` *string*, **required** — When it last changed (RFC 3339).

## ChapterClaimsJson

Every chapter claim for an edition.

- `claims` *array of [`ChapterClaimJson`](#chapterclaimjson)*, **required** — The claims, most recently changed first.

## ChapterEditBody

`PUT /api/library/editions/{id}/chapters` body: the edition's chapters
as an Administrator sets them, in pre-order. An empty list clears them.

- `chapters` *array of [`ChapterEditEntry`](#chaptereditentry)*, **required** — The chapters, at most 2,000. At most 2000 items.

## ChapterEditEntry

One chapter of an edit, as JSON carries it.

- `title` *string*, **required** — Chapter title: 1 to 512 bytes, no control characters, no leading or trailing white space.
- `start_ms` *integer (int64)*, **required** — Start, in milliseconds from the start of the edition's timeline; no earlier than the chapter before it. At least 0.
- `depth` *integer (int32)* — Nesting depth, 0 for a top-level chapter; at most one more than the chapter before it, and at most 7. Left out reads as 0. At least 0.

## ChapterJsonMode

Whether / which chapter JSON sidecars to write beside acquired audio.

Default is [`Self::Off`]. Replaces the old `save_chapter_json` bool +
`chapter_layout` pair for *sidecar output* (Audible API fetch layout is
chosen separately when chapters are needed for embedding).

Type: one of `off`, `flat`, `tree`, `both`

## ChapterPinBody

`PUT /api/library/editions/{id}/chapters/pin` body: the claim to pin.

- `source_kind` *[`ChapterSourceKind`](#chaptersourcekind)*, **required** — The claim's source kind, as the claims list names it.
- `source_id` *string or null* — The claim's source id (a PluginKey; empty for an edit or for Bookclerk's own processing). Left out reads as empty.

## ChapterSourceKind

The kind of source a chapter claim came from (`source_kind`).

Type: one of `person`, `outside_edit`, `processed`, `provider`, `listing`, `file`, `part_names`

## ChapterSpan

One chapter as a player seeks between them: where it starts and ends
on the edition's timeline ([`ChapterList::spans`]).

- `title` *string*, **required** — Chapter title.
- `start_ms` *integer (int64)*, **required** — Start, in milliseconds from the start of the edition's timeline. At least 0.
- `end_ms` *integer (int64)*, **required** — End, in milliseconds: the next chapter's start, or the end of the timeline for the last chapter; never before `start_ms`. At least 0.
- `depth` *integer (int32)*, **required** — Nesting depth; 0 for a top-level chapter. A chapter at depth n + 1 is nested under the nearest earlier chapter at depth n. At least 0.

## ChoiceBody

Body of `PUT /api/me/notifications/choices/{type}`.

- `enabled` *boolean*, **required** — `false` keeps the type in the inbox only.
- `destination_ids` *array of string* — The caller's own destinations it goes to; at most 32. Left out or empty, the type goes to no destination outside the inbox, even with `enabled: true`; delete the choice to return to the defaults. At most 32 items.

## ClerkAuthorizeParams

The authorization request a clerk consent is for (the query
`/oidc/authorize` forwards to the library's `/clerk/approve` page).

- `client_id` *string*, **required** — Must be [`CLERK_CLIENT_ID`].
- `redirect_uri` *string*, **required** — Where the code goes: loopback or https.
- `response_type` *string*, **required** — Must be `code`.
- `scope` *string or null* — Must be [`CLERK_SCOPE`].
- `state` *string or null* — Echoed on the redirect.
- `code_challenge` *string*, **required** — PKCE S256 challenge.
- `code_challenge_method` *string or null* — Must be `S256`.
- `clerk_name` *string or null* — The name the clerk gives itself, shown on the consent page and in the user's session list.

## ClerkCallbackBody

`POST /api/config/clerk/callback` body.

- `code` *string* — The code from the library.
- `state` *string* — The `state` it came back with.
- `error` *string or null* — The library's `error` (`access_denied`), when it sent one.

## ClerkCards

JSON body for `GET /api/config/clerk`.

- `cards` *array of [`ClerkLinkView`](#clerklinkview)*, **required** — The libraries this install holds (or held) a card for, oldest sign-in first.
- `default_clerk_name` *string*, **required** — The name this install signs in as unless another is given.

## ClerkConsentBody

`POST /api/auth/clerk-consent` body.

Includes every field of [`ClerkAuthorizeParams`](#clerkauthorizeparams).

- `approve` *boolean*, **required** — True to let the clerk sign in; false to deny.

## ClerkConsentPage

JSON body for `GET /api/auth/clerk-consent`: what the consent page shows.

- `library_name` *string*, **required** — This library's name.
- `clerk_name` *string*, **required** — The name the clerk gives itself.
- `redirect_origin` *string*, **required** — Origin of the clerk's redirect URI, for the page to name.
- `user` *[`ConsentingUser`](#consentinguser)*, **required** — The signed-in user the clerk would sign in as.

## ClerkInspectBody

`POST /api/config/clerk/inspect` body.

- `address` *string*, **required** — The library's address (`https://books.example.com`, `books.lan:8443`).

## ClerkLinkView

A library this install holds (or held) a library card for, without the
card.

- `library_id` *string*, **required** — The library id.
- `origin` *string*, **required** — The library's canonical origin.
- `library_name` *string*, **required** — The library's name at sign-in.
- `pinned` *boolean*, **required** — Whether the card's connection pins the library's self-signed certificate.
- `display_name` *string or null* — The person's display name on the library, when it has one.
- `clerk_name` *string*, **required** — The name this install signed in as.
- `state` *string*, **required** — `active`, or `signed_out` once the library ended the card.
- `signed_in_at` *string*, **required** — RFC 3339 sign-in time.

## ClerkRequestBody

`POST /api/config/clerk/request` body.

- `library_id` *string*, **required** — The library to ask: one this install holds an active card for.
- `work_id` *string*, **required** — The library's own work id: 1 to 64 ASCII letters, digits and `-`.
- `from` *[`ClerkRequestFrom`](#clerkrequestfrom)*, **required** — Where the copy comes from: the edition's `request.from` in the window books.
- `node_id` *string or null* — The library's branch to ask, from the work's `available_through` there; absent asks the first branch that lists it. Only with `from` `branch`.

## ClerkRequestFrom

Where a clerk's request asks the library to get a copy from.

Type: one of `branch`, `library`

## ClerkRequestSent

JSON body for `POST /api/config/clerk/request`.

- `library_id` *string*, **required** — The library that took the request.

## ClerkSignedIn

JSON body for `POST /api/config/clerk/callback`.

- `card` *[`ClerkLinkView`](#clerklinkview)*, **required** — The library this install now holds a card for.
- `left_branch` *[`LeftBranch`](#leftbranch) or null* — Set when the card replaced this install's branch link to the library (a sign-in started `from_branch`): the branch was left.

## ClerkSignedOut

JSON body for `POST /api/config/clerk/sign-out`.

- `library_id` *string*, **required** — The library id.
- `library_signed_out` *boolean*, **required** — Whether the library ended the card's session too. False when it could not be reached (or the card had already ended): the session then stays in the person's session list there until it idles out or they revoke it.

## ClerkSignOutBody

`POST /api/config/clerk/sign-out` body.

- `library_id` *string*, **required** — The library id.

## ClerkStartBody

`POST /api/config/clerk/start` body.

- `address` *string* — The library's address. Left out with `from_branch`.
- `from_branch` *string or null* — The library id of a library this install is an active branch of: the card replaces the branch. The library's address and pinned certificate come from the branch link, so `address` and `fingerprint` are left out; once the library hands over the card, the callback leaves the branch (as Leave library does) and records the card. Only the operator may, as only the operator leaves a library.
- `fingerprint` *string or null* — The fingerprint the person confirmed (self-signed libraries).
- `clerk_name` *string or null* — The name to sign in as (default: this machine's name).

## ClerkStarted

JSON body for `POST /api/config/clerk/start`.

- `authorize_url` *string*, **required** — The library's sign-in URL to open in the browser.
- `library` *[`InspectedLibraryView`](#inspectedlibraryview)*, **required** — What the library said about itself.
- `clerk_name` *string*, **required** — The name this install signs in as.
- `expires_in_secs` *integer (int64)*, **required** — Seconds the pending sign-in stays valid. At least 0.

## ClerkTokenAnswer

`POST /api/clerk/v1/token` answer.

- `token` *string*, **required** — The session token, sent as `Authorization: Bookclerk-Clerk <token>`. Shown once; the library keeps only its hash.
- `library_id` *string*, **required** — The library id.
- `user` *[`ClerkUser`](#clerkuser)*, **required** — The signed-in person on that library.
- `idle_secs` *integer (int64)*, **required** — Seconds the session stays signed in unused; each use resets it.

## ClerkTokenRequest

`POST /api/clerk/v1/token` body: exactly these three strings, each
once, and nothing else.

- `code` *string*, **required** — The single-use code from the redirect.
- `code_verifier` *string*, **required** — The PKCE verifier whose S256 challenge the authorize request sent.
- `redirect_uri` *string*, **required** — The redirect URI the authorize request named.

## ClerkUser

The person a clerk session signs in as.

- `id` *integer (int64)*, **required** — The user's id on the library.
- `display_name` *string or null* — The user's display name on the library, when set.

## ConfigOptionValueInfo

One allowed value for a storefront config option.

- `id` *string*, **required** — Stored option value (for example `m4b`).
- `label` *string*, **required** — Operator-facing value label.

## ConflictJson

One conflict the resolver could not decide.

- `listing_id` *string*, **required** — The listing whose resolution conflicted.
- `kind` *string*, **required** — What conflicted: `strong_identifiers` (two editions share the listing's strong identifiers), `work_identifiers` (its work identifiers name two works), `ambiguous_work` (two works match its title and authors) or `work_kind_mismatch` (its work identifier names a work of another kind).
- `subject_ids` *array of string*, **required** — The editions (`strong_identifiers`) or works (the other kinds) involved.
- `plugin_key` *string*, **required** — The listing's storefront PluginKey.
- `product_id` *string*, **required** — The store's product id.
- `title` *string*, **required** — The listing's title as the store reported it.
- `edition_id` *string*, **required** — The edition the listing sells now.
- `updated_at` *string*, **required** — When the conflict was last seen (RFC 3339).

## ConflictPage

One page of conflicts.

- `conflicts` *array of [`ConflictJson`](#conflictjson)*, **required** — The conflicts, by listing.
- `next_cursor` *string or null* — Pass as `cursor` for the next page; absent on the last page.

## ConnectionChoice

What the consent page may offer: branch only, or a choice between branch and shared_branch for an Owner.

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
- `pace` *[`PaceView`](#paceview)*, **required** — How fast Bookclerk downloads from this account when it starts the downloads itself.

## ConnectionsAdded

JSON body after `POST /api/config/join/connections`.

- `added` *integer (int64)*, **required** — Accounts now connected in the library for the person. At least 0.
- `already_here` *integer (int64)*, **required** — Accounts the person had already connected there. At least 0.
- `taken` *integer (int64)*, **required** — Accounts connected to someone else there; left as they are. At least 0.
- `failed` *integer (int64)*, **required** — Accounts that could not be kept there. At least 0.
- `not_sent` *integer (int64)*, **required** — Accounts not sent: not the person's, not connected, or of a storefront whose sign-ins can't be shared. At least 0.
- `not_allowed` *boolean*, **required** — The library's operator does not let the person connect store accounts there; nothing was kept.

## ConnectionsResponse

JSON body for `GET /connections`.

- `connections` *array of [`ConnectionInfo`](#connectioninfo)*, **required** — Linked store accounts the SPA may show or revoke.

## ConnectionType

How a joining install connects to a library.

Type: one of `branch`, `shared_branch`

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
- `travels` *boolean*, **required** — True when the connection travels: a shareable storefront's, or a single-keeper one's that no other branch keeps yet (the branch then keeps its login fresh for this library). False when another branch already keeps it, or it does not fit one join's bundle.

## ConsentingUser

The user a branch consent is for.

- `id` *integer (int64)*, **required** — `users.id`.
- `display_name` *string or null* — Display name, when set.

## ConsentRedirect

JSON body after a branch consent was answered.

- `redirect` *string*, **required** — Where to send the browser: the branch's callback with a code, or with `error=access_denied`.

## ControlOverrides

One patron's overrides: a value for some controls; the rest (an absent
or `null` field) follow the library's default for a Member and are
allowed for an Administrator.

- `link_accounts` *boolean or null* — Overrides [`MemberControls::link_accounts`].
- `download` *boolean or null* — Overrides [`MemberControls::download`].
- `edit_metadata` *boolean or null* — Overrides [`MemberControls::edit_metadata`].
- `upload` *boolean or null* — Overrides [`MemberControls::upload`].
- `request_books` *boolean or null* — Overrides [`MemberControls::request_books`].
- `see_prices` *boolean or null* — Overrides [`MemberControls::see_prices`].

## ConvertRequestBody

Body for `POST /api/library/convert`.

- `copy` *[`OutputCopy`](#outputcopy)*, **required** — The copy to add: a format and a quality, as `output.copies` lists them. `original` is refused: a convert makes a new encoding.
- `titles` *array of string* — Title ids (uuid, ASIN, ISBN, product id) of stored books; each becomes its own job.
- `editions` *array of string* — Editions to convert (`id` of an edition in a work's detail); each becomes one job for the title that acquired it.
- `force` *boolean* — Make the copy again when the book already has it.

## CopyFormat

The format of one copy of a book.

Type: one of `m4b`, `mp3`, `mp3_chapters`, `original`, `opus`

## CopyQuality

A copy's quality: `original`, a bitrate (for MP3 a constant rate the encoder writes: `64k`, `96k`, `128k`, `160k`, `192k`, `256k` or `320k`; for Opus a target: `16k`, `24k`, `32k`, `48k`, `64k`, `96k` or `128k`), or a LAME VBR quality `v0` to `v9` (MP3 only); the last two may end in `-mono`.

Type: string

Constraints: matches `^(original|((16|24|32|48|64|96|128|160|192|256|320)k|v[0-9])(-mono)?)$`

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

## DefaultAdminControlsJson

The library's default Administrator switches as the switch routes
report them.

- `controls` *[`AdminControls`](#admincontrols)*, **required** — The switches every Administrator follows unless set for them alone.

## DefaultControlsJson

The library's default member controls as the control routes report
them.

- `controls` *[`MemberControls`](#membercontrols)*, **required** — The controls every Member follows unless set for them alone. Administrators do not follow it.

## DefaultScopeJson

The library's default scope as the scope routes report it.

- `scope` *[`PatronScope`](#patronscope)*, **required** — The scope patrons without their own follow.

## DefaultSwitchChanges

Changes to some of the library's default switches: a new value for
each named switch; an absent switch keeps its stored value.

- `manage_members` *boolean or null* — New default for [`AdminControl::ManageMembers`].
- `set_patron_limits` *boolean or null* — New default for [`AdminControl::SetPatronLimits`].
- `library_scan` *boolean or null* — New default for [`AdminControl::LibraryScan`].
- `run_jobs` *boolean or null* — New default for [`AdminControl::RunJobs`].
- `fix_catalog` *boolean or null* — New default for [`AdminControl::FixCatalog`].
- `invite_branches` *boolean or null* — New default for [`AdminControl::InviteBranches`].
- `approve_requests` *boolean or null* — New default for [`AdminControl::ApproveRequests`].

## DeliveryBacklog

Event delivery backlog across the library.

- `pending` *integer (int64)*, **required** — Deliveries waiting for a consumer.
- `running` *integer (int64)*, **required** — Deliveries a consumer is handling now.
- `dead_letter` *integer (int64)*, **required** — Deliveries that exhausted their retries.
- `oldest_pending_age_secs` *integer (int64) or null* — Age of the oldest pending delivery in seconds; `null` when none is pending.

## DeliveryJson

One delivery-log row.

- `id` *string*, **required** — Delivery id.
- `notification_id` *string*, **required** — The notification.
- `notification_type` *string*, **required** — Its type.
- `title` *string*, **required** — Its title.
- `destination_id` *string*, **required** — The destination.
- `plugin_instance_id` *string*, **required** — The notifier instance.
- `state` *string*, **required** — `pending`, `delivered`, `failed` or `dropped`.
- `attempts` *integer (int64)*, **required** — Send attempts made.
- `provider_id` *string or null* — The service's id for the delivered message.
- `reason` *string or null* — The last failure or retry reason.
- `created_at` *string*, **required** — When it was made (RFC 3339).
- `updated_at` *string*, **required** — When it last changed (RFC 3339).

## DeliveryPage

One page of the delivery log.

- `deliveries` *array of [`DeliveryJson`](#deliveryjson)*, **required** — Newest first.
- `next_cursor` *string or null* — Pass as `cursor` for the next page; `null` on the last page, where paging stops.

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

## DestinationFilter

Which books one destination takes.

- `rules` *array of [`FilterRule`](#filterrule)* — Filter rules, at most 64. Empty means the destination takes every book. At most 64 items.

## DestinationFilterResponse

`GET` and `PUT /api/config/plugin-instances/{id}/filter` body: which
books a storage destination takes.

- `plugin_instance_id` *string*, **required** — The destination's plugin instance id.
- `revision` *integer (int64)*, **required** — Revision of the destination's `core.destination` document; `0` when none was written, so the destination takes every book.
- `filter` *[`DestinationFilter`](#destinationfilter)*, **required** — The filter; no rules take every book.

## DestinationJson

A destination as administrators and its owner see it. The address is
never returned: it may hold secrets.

- `id` *string*, **required** — Destination id.
- `kind` *[`DestinationKind`](#destinationkind)*, **required** — `channel` (the library's) or `personal` (a patron's).
- `plugin_instance_id` *string*, **required** — The notifier instance that carries it.
- `label` *string*, **required** — Name shown in pickers.
- `failure_count` *integer (int64)*, **required** — Deliveries in a row that ended `failed`, whether refused for good or out of attempts; a delivery, or resuming the destination, resets it to 0. It pauses at 5.
- `paused_at` *string or null* — When repeated failures paused it (RFC 3339); `null` while active.
- `paused_reason` *string or null* — Why it was paused.
- `created_at` *string*, **required** — When it was made (RFC 3339).

## DestinationKind

Whether a destination is a library channel or one patron's address.

Type: one of `channel`, `personal`

## DeviceInfoBody

The device an app plays on, as it describes itself.

- `clientName` *string or null* — The app's name.
- `manufacturer` *string or null* — The device's maker.
- `model` *string or null* — The device's model.
- `deviceName` *string or null* — The name the patron gave the device.

## DiagnosticsReportDeleted

Body of `DELETE /api/diagnostics/reports/{id}`.

- `report_id` *string*, **required** — The report's ID.
- `deleted_at_unix_ms` *integer (int64)*, **required** — When its stored copy was deleted, Unix milliseconds. At least 0.

## DiagnosticsReportLookup

Body of `GET /api/diagnostics/reports/{id}`: the report as the collector
holds it, and what became of it.

- `report_id` *string*, **required** — The report's ID.
- `received_at_unix_ms` *integer (int64)*, **required** — When the collector received it, Unix milliseconds. At least 0.
- `deleted_at_unix_ms` *integer (int64) or null* — When its stored copy was deleted, Unix milliseconds, or null. At least 0.
- `report` *[`StoredReportView`](#storedreportview) or null* — The stored report, or null once deleted.
- `issues` *array of [`ReportIssueView`](#reportissueview)*, **required** — Issues tracking it, the tracking issue first.
- `fix` *[`ReportFixView`](#reportfixview) or null* — The tracking issue's fix, or null.

## DiagnosticsReportsResponse

Body of `GET /api/diagnostics/reports`.

- `reports` *array of [`DiagnosticsReportView`](#diagnosticsreportview)*, **required** — Sent reports, newest first (the last 200 are kept).

## DiagnosticsReportView

One diagnostics report this install sent.

- `report_id` *string*, **required** — The collector's ID for the report (a UUID). The GitHub issue filed for it mentions this ID; search the issues for it or give it to a maintainer.
- `trigger` *string*, **required** — Why it was sent: `crash`, `error_burst`, `slow_burst`, `job_failed`, `cli_error`, `cli_manual`, ….
- `version` *string*, **required** — The Bookclerk version that sent it.
- `sent_at_unix_ms` *integer (int64)*, **required** — When the collector accepted it, Unix milliseconds. At least 0.
- `access_code` *string or null* — The code that reads the report back or deletes it, in groups of four, or null when the collector issued none. With the ID it also opens the collector's public lookup page.
- `deleted_at_unix_ms` *integer (int64) or null* — When this install deleted the collector's stored copy, Unix milliseconds, or null while it is stored. At least 0.

## DisconnectBody

`POST /api/config/join/connections/disconnect` body.

- `plugin_key` *string*, **required** — Provenance-qualified plugin key of the storefront.
- `account_id` *string*, **required** — Store account id.
- `everywhere` *boolean* — Also disconnect it in every linked library it was shared with; absent means `false`.

## Disconnected

JSON body after `POST /api/config/join/connections/disconnect`.

- `here` *boolean*, **required** — The account was the person's here and is disconnected here now.
- `libraries` *array of string*, **required** — Linked libraries (parent library ids) that confirmed the disconnect.
- `unreached` *array of string*, **required** — Linked libraries that could not be reached or refused; the account stays connected there until the person disconnects it there.

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
- `scope_type` *string*, **required** — Scope type: `library` for a shared document, `node` for one node's.
- `scope_id` *string*, **required** — Scope id: `singleton` for the library, else the node id.
- `schema_version` *integer (int64)*, **required** — Version of the document's schema, starting at 1.
- `revision` *integer (int64)*, **required** — Compare-and-swap revision; send it back as `expected_revision`.
- `updated_at` *string*, **required** — RFC 3339 time of the last write.
- `updated_by` *string*, **required** — Audit identity of the last writer.

## DownloadRequestBody

`POST /api/download-requests` body: the work to download.

- `work_id` *string*, **required** — This library's work id, as its window books and search name it.

## DownloadRequestQueued

`POST /api/download-requests` answer.

- `job_id` *string*, **required** — The acquire job, as `/api/jobs` lists it.
- `edition_id` *string*, **required** — The edition being downloaded.
- `already_queued` *boolean*, **required** — `true` when a download of that edition was already waiting or running; `job_id` is that download.

## EditionChapter

One chapter of a checked list.

Named `EditionChapter` in the OpenAPI documents, apart from the player
apps' `Chapter`.

- `title` *string*, **required** — Chapter title; never empty, at most 512 bytes, no control characters.
- `start_ms` *integer (int64)*, **required** — Start, in milliseconds from the start of the edition's timeline. At least 0.
- `depth` *integer (int32)*, **required** — Nesting depth; 0 for a top-level chapter, at most 7. A chapter at depth n + 1 is nested under the nearest earlier chapter at depth n. At least 0.

## EditionChaptersJson

The chapters an edition plays.

- `edition_id` *string*, **required** — The edition.
- `chapters` *array of [`EditionChapter`](#editionchapter)*, **required** — The chapters in pre-order, starts never decreasing; empty when the edition has none or an Administrator cleared them.
- `source_kind` *[`ChapterSourceKind`](#chaptersourcekind) or null* — The kind of source the chapters came from; absent when the edition has no chapters yet.
- `source_id` *string or null* — Which source of that kind (a PluginKey; empty for an edit or for Bookclerk's own processing).
- `runtime_ms` *integer (int64) or null* — Length of the timeline the starts were measured on, in milliseconds. At least 0.

## EditionJson

One edition of a work.

- `id` *string*, **required** — Edition id.
- `work_id` *string*, **required** — The work it realises: this work, or one of a show's episodes.
- `expression_id` *string*, **required** — The expression it carries.
- `expression_kind` *[`ExpressionKindJson`](#expressionkindjson)*, **required** — How the expression realises the work.
- `language` *string or null* — The expression's language tag, when known.
- `abridgement` *[`AbridgementJson`](#abridgementjson)*, **required** — Whether the expression is abridged.
- `media_kind` *[`MediaKindJson`](#mediakindjson)*, **required** — Carrier of the edition.
- `title` *string*, **required** — The edition's title.
- `subtitle` *string or null* — The edition's subtitle.
- `published_at` *string or null* — Publication date as the store gave it.
- `duration_minutes` *integer (int64) or null* — Running time in minutes, for audio.
- `cover_path` *string or null* — Same-origin path of the edition's cover, when it has one.
- `acquire_status` *[`AcquireStatus`](#acquirestatus)*, **required** — The edition's acquire state.
- `listings` *array of [`ListingJson`](#listingjson)*, **required** — The edition's store listings.

## Elevated

JSON body after a successful Owner elevation.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `elevated` *boolean*, **required** — Always `true`: the cookie the response sets is an elevated operator session.
- `expires_in_secs` *integer (int64)*, **required** — Seconds until the elevated session expires. At least 0.

## ElevateRequest

JSON body for Owner password re-auth before issuing an elevated operator cookie.

- `password` *string*, **required** — Owner account password (re-authentication).

## EmptyPage

An empty page, for collections and playlists the library does not
have yet.

- `results` *array of object*, **required** — Always empty.
- `total` *integer (int64)*, **required** — Always 0. At least 0.
- `limit` *integer (int64)*, **required** — Items per page, echoed. At least 0.
- `page` *integer (int64)*, **required** — The page number, echoed. At least 0.

## EnrollBody

`POST /api/node/v1/enroll` body. Keys are base64url without padding.

- `link_id` *string*, **required** — Link id from the token.
- `secret` *string*, **required** — Link secret from the token.
- `sign_pk` *string*, **required** — Ed25519 public signing key.
- `box_pk` *string*, **required** — X25519 public box key.
- `software_version` *string* — Host's Bookclerk version.

## EnrolledNodeView

JSON body after an enrollment link or branch sign-in was redeemed.

- `library_id` *string*, **required** — The library the node joined.
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

## EntitlementJson

On what terms a holder has a listing.

Type: one of `purchase`, `plus`, `loan`, `free`, `own`

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
- `concurrency` *integer (int32)*, **required** — Local delivery workers and the library in-flight cap. At least 0.

## ExplicitContent

Whether a patron sees books marked as explicit (adult) content.

Type: one of `allow`, `hide`

## ExpressionKindJson

How an expression realises its work (`expressions.kind`).

Type: one of `text`, `narration`, `dramatisation`, `graphic`, `original_audio`

## FacetValueJson

One value of a facet.

- `value` *string*, **required** — The value, as `value` of `GET /api/library/works` takes it.
- `works` *integer (int64)*, **required** — How many held works have it. At least 0.

## FacetValuesJson

A page of `GET /api/library/works/facets/{facet}`.

- `values` *array of [`FacetValueJson`](#facetvaluejson)*, **required** — The values, in order of their lower-cased spelling.
- `next_cursor` *string or null* — Pass as `cursor` for the next page; absent on the last page.

## FeedState

How current a stored Discover feed is.

Type: one of `ready`, `refreshing`, `preparing`

## FieldJson

One labelled value.

- `name` *string*, **required** — Label.
- `value` *string*, **required** — Value.

## FileMetadata

A file's name, size and times.

- `filename` *string*, **required** — The file name.
- `ext` *string*, **required** — The extension with its dot (`.m4b`).
- `path` *string*, **required** — A display path: the file name.
- `relPath` *string*, **required** — A display path: the file name.
- `size` *integer (int64)*, **required** — The file's size in bytes. At least 0.
- `mtimeMs` *integer (int64)*, **required** — When the file was stored, in epoch milliseconds.
- `ctimeMs` *integer (int64)*, **required** — When the file was stored, in epoch milliseconds.
- `birthtimeMs` *integer (int64)*, **required** — When the file was stored, in epoch milliseconds.

## FileTimestampMode

Which timestamp to apply after acquire (`CreationTime` / `LastWriteTime`).

Type: one of `now`, `purchased`, `published`

## FilterData

The values an app offers as filters.

- `authors` *array of [`NamedRef`](#namedref)*, **required** — Authors with at least one item.
- `genres` *array of string*, **required** — Genres.
- `tags` *array of string*, **required** — Tags: none are shown.
- `series` *array of [`NamedRef`](#namedref)*, **required** — Series with at least one item.
- `narrators` *array of string*, **required** — Narrators.
- `languages` *array of string*, **required** — Languages.
- `publishers` *array of string*, **required** — Publishers.
- `publishedDecades` *array of string*, **required** — Decades of publication: none are offered.
- `bookCount` *integer (int64)*, **required** — Items in the library. At least 0.
- `authorCount` *integer (int64)*, **required** — Authors in the library. At least 0.
- `seriesCount` *integer (int64)*, **required** — Series in the library. At least 0.
- `podcastCount` *integer (int64)*, **required** — Always 0. At least 0.
- `numberEpisodes` *integer (int64)*, **required** — Always 0. At least 0.
- `loadedAt` *integer (int64)*, **required** — When the values were read, in epoch milliseconds.

## FilterEffect

Whether a [`FilterRule`] lets matching books in or keeps them out.

Type: one of `include`, `exclude`

## FilterField

The book field a [`FilterRule`] matches one value of.

Type: one of `patron`, `storefront`, `account`, `media_kind`, `format`, `quality`

## FilterRule

One rule of a destination filter: a field, one value of it, and its
effect.

- `field` *[`FilterField`](#filterfield)*, **required** — The field the rule matches.
- `value` *string*, **required** — The value: a user id for `patron`, `book` or `podcast` for `media_kind`, a copy format for `format`, a copy quality for `quality`, otherwise 1 to 128 bytes of text.
- `effect` *[`FilterEffect`](#filtereffect)*, **required** — Include or exclude.

## GuestReservation

CPU and memory one guest may use at most: its grant's ceilings after the
node's `[plugins.jail]` limits.

- `memory_bytes` *integer (int64)*, **required** — Memory ceiling in bytes.
- `cpu_millicores` *integer (int64)*, **required** — CPU ceiling in millicores.

## HealthResponse

Liveness payload for unauthenticated `GET /health`.

- `status` *string*, **required** — Always `ok` when the process can serve HTTP.

## HolderJson

Who holds a listing.

Type: object

## HoldingJson

One holding of a listing.

- `id` *string*, **required** — Holding id.
- `holder` *[`HolderJson`](#holderjson)*, **required** — Who holds the listing.
- `entitlement` *[`EntitlementJson`](#entitlementjson)*, **required** — On what terms.
- `purchased_at` *string or null* — RFC 3339 purchase time, when the store reported one.
- `added_at` *string*, **required** — RFC 3339 time the holding was first recorded.

## ImpersonateRequest

JSON body naming the first-party user to impersonate.

- `user_id` *integer (int64)*, **required** — Target `users.id` to impersonate.

## ImpersonationStarted

JSON body after an operator session started impersonating a user.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `impersonating` *[`AuthMeImpersonating`](#authmeimpersonating)*, **required** — The user the session now acts as.

## InboxEntryJson

One inbox entry.

- `id` *string*, **required** — Notification id.
- `notification_type` *string*, **required** — Type id.
- `severity` *string*, **required** — `info`, `warning` or `critical`.
- `title` *string*, **required** — Title.
- `text` *string*, **required** — Plain-text body.
- `link_path` *string or null* — Path inside the library it links to.
- `fields` *array of [`FieldJson`](#fieldjson)*, **required** — Labelled values.
- `created_at` *string*, **required** — When it arrived (RFC 3339).
- `read_at` *string or null* — When the patron read it (RFC 3339); `null` while unread.

## InboxPage

One page of the inbox.

- `entries` *array of [`InboxEntryJson`](#inboxentryjson)*, **required** — Newest first.
- `unread` *integer (int64)*, **required** — Unread entries in the whole inbox. At least 0.
- `next_cursor` *string or null* — Pass as `cursor` for the next page; `null` on the last page, where paging stops.

## IngressChange

`POST /api/config/ingress` body: each field is one change, applied in order
(off or on, then exposure, then accepting a reported origin).

- `activate` *string or null* — Makes this the library's active ingress: a plugin instance UUID, or a plugin key for that plugin's default instance.
- `deactivate` *boolean* — Turns ingress off; the adopted origin is kept.
- `exposure` *[`IngressExposure`](#ingressexposure) or null* — What public traffic may reach.
- `accept_reported_origin` *boolean* — Adopts the origin the active instance reported. Renaming the library breaks passkeys and sign-in redirects registered under the old name.

## IngressExposure

What public traffic from the ingress may reach.

Type: one of `branches`, `library`

## IngressStatus

`GET /api/config/ingress` and the answer to `POST /api/config/ingress`: the library's
ingress settings and what they mean now.

- `active_instance` *string or null* — The plugin instance serving the library's public address, as its UUID; `null` when ingress is off.
- `plugin_key` *string or null* — The active instance's plugin, when the instance exists.
- `exposure` *[`IngressExposure`](#ingressexposure)*, **required** — What public traffic may reach.
- `origin` *string or null* — The adopted public origin, kept while ingress is off.
- `reported_origin` *string or null* — A different origin the active instance reported, waiting for the operator to accept it.
- `public_origin` *string or null* — The origin the library serves under now: `integrations.public_origin` when set, else the ingress origin while the instance runs.
- `operator_override` *boolean*, **required** — True when `integrations.public_origin` in `config.toml` overrides the ingress origin.
- `blocked` *string or null* — Why ingress can't run now (no Owner yet, or operator auth off); `null` when it can.

## InspectBody

`POST /api/config/join/inspect` body.

- `address` *string*, **required** — The library's address (`https://owner.example`, `owner.lan:8443`).

## InspectedLibraryResponse

JSON body for `POST /api/config/join/inspect`.

- `library` *[`InspectedLibraryView`](#inspectedlibraryview)*, **required** — What the library said about itself.

## InspectedLibraryView

A library as `POST /api/config/join/inspect` saw it.

- `library_id` *string*, **required** — The library id.
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

## IntakeAttribution

Who holds the books a drop folder ingests.

Type: one of `owner`, `subfolder`

## IntakeResponse

`GET`, `PUT` and `DELETE /api/config/plugin-instances/{id}/intake` body:
whether a storage instance is a drop folder, and its settings.

- `plugin_instance_id` *string*, **required** — The storage instance's plugin instance id.
- `revision` *integer (int64)*, **required** — Revision of the instance's `core.intake` document; `0` when none was written, so the instance is a destination.
- `intake` *[`IntakeSettings`](#intakesettings) or null* — The drop folder's settings; `null` when the instance is a destination.

## IntakeSettings

The settings of one drop folder (`core.intake`).

Every field has a default, so `{}` is a valid body only when
`attribute_to` is `subfolder`; `owner` needs a `patron`.

- `settle_seconds` *integer (int32)*, **required** — Seconds an item's files must stay unchanged before it is ingested, 5 to 86,400 (default 60). At least 5; at most 86400.
- `scan_interval_seconds` *integer (int32)*, **required** — Seconds between two full listings of the folder, 30 to 86,400 (default 300). A change feed starts a listing sooner. At least 30; at most 86400.
- `after_ingest` *[`AfterIngest`](#afteringest)*, **required** — What happens to an item once it is ingested (default `move`).
- `attribute_to` *[`IntakeAttribution`](#intakeattribution)*, **required** — Who holds the ingested books (default `owner`).
- `patron` *integer (int64) or null* — The user id that holds every book in `owner` mode; required there and refused in `subfolder` mode. At least 1.
- `ignore` *array of string*, **required** — Extra file name patterns to ignore, at most 64, each 1 to 128 bytes with no `/`: `*` matches any run of characters and `?` one character, compared without ASCII case. The built-in list always applies too ([`BUILTIN_IGNORE_PATTERNS`]). At most 64 items.
- `prune_imported_days` *integer (int32) or null* — With `after_ingest = move`, delete `_imported` day folders older than this many days, 1 to 3,650; absent keeps them. At least 1; at most 3650.

## IntakeSettingsRequest

The drop folder settings as a request body may send them: every field
may be left out and takes the default [`IntakeSettings`] names. Only the
published API schema uses it; the body is read as [`IntakeSettings`].

- `settle_seconds` *integer (int32)* — Seconds an item's files must stay unchanged before it is ingested, 5 to 86,400 (default 60). At least 5; at most 86400.
- `scan_interval_seconds` *integer (int32)* — Seconds between two full listings of the folder, 30 to 86,400 (default 300). At least 30; at most 86400.
- `after_ingest` *[`AfterIngest`](#afteringest)* — What happens to an item once it is ingested (default `move`).
- `attribute_to` *[`IntakeAttribution`](#intakeattribution)* — Who holds the ingested books (default `owner`).
- `patron` *integer (int64) or null* — The user id that holds every book in `owner` mode; required there and refused in `subfolder` mode. It must name an active user. At least 1.
- `ignore` *array of string* — Extra file name patterns to ignore, at most 64, each 1 to 128 bytes with no `/` (default none). At most 64 items.
- `prune_imported_days` *integer (int32) or null* — With `after_ingest = move`, delete `_imported` day folders older than this many days, 1 to 3,650; absent keeps them. At least 1; at most 3650.

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

## ItemListening

Seconds listened to one item.

- `id` *string*, **required** — The item id.
- `timeListening` *number (double)*, **required** — Seconds listened.
- `mediaMetadata` *[`BookMetadata`](#bookmetadata)*, **required** — The item's metadata.

## ItemPage

A page of items: `GET /api/libraries/{id}/items` and the like.

- `results` *array of [`LibraryItem`](#libraryitem)*, **required** — The page's items.
- `total` *integer (int64)*, **required** — Items matching the query in all. At least 0.
- `limit` *integer (int64)*, **required** — Items per page (0: no paging). At least 0.
- `page` *integer (int64)*, **required** — The page number, from 0. At least 0.
- `sortBy` *string or null* — The sort key the app sent, echoed.
- `sortDesc` *boolean*, **required** — Whether the order is reversed.
- `filterBy` *string or null* — The filter the app sent, echoed.
- `mediaType` *string*, **required** — Always [`MEDIA_TYPE_BOOK`].
- `minified` *boolean*, **required** — Whether items are minified.
- `collapseseries` *boolean*, **required** — Whether series were collapsed: never.
- `include` *string*, **required** — The include list the app sent, echoed.
- `offset` *integer (int64)*, **required** — The offset of the first result. At least 0.

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

Type: one of `scan`, `acquire`, `listen_sync`, `integration_scan`, `plugin_copy`, `plugin_scheduled`, `discover_candidates`, `discover_feed`, `discover_request_queue`, `cache_sweep`, `search_index_rebuild`, `library_index_rebuild`, `branch_request`, `ingest_batch`, `notify_send`, `chapter_align`, `invalid`

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

JSON body once a join completed: as a branch, or as a shared branch.

Type: [`BranchJoined`](#branchjoined) or [`SharedBranchJoined`](#sharedbranchjoined)

## JoinConflict

A `409` from a join step: the handler's error, or an origin mismatch that names the address to join with; the route's examples name each.

Type: [`ErrorDetail`](#errordetail) or [`OriginMismatch`](#originmismatch)

## JoinStarted

JSON body for `POST /api/config/join/start`.

- `authorize_url` *string*, **required** — The library's sign-in URL to open in the browser.
- `library` *[`InspectedLibraryView`](#inspectedlibraryview)*, **required** — What the library said about itself.
- `branch_name` *string*, **required** — The name this install asked to join as.
- `shared_branch_offered` *boolean*, **required** — Whether joining as a shared branch was offered.
- `expires_in_secs` *integer (int64)*, **required** — Seconds the pending sign-in stays valid. At least 0.

## JoinStatus

JSON body for `GET /api/config/join`.

- `fresh` *boolean*, **required** — Whether this install has no users, no libraries and no library cards yet.
- `has_owner` *boolean*, **required** — Whether an Owner (or legacy administrator) exists here.
- `shared_branch_eligible` *boolean*, **required** — Whether this install could still join a library as a shared branch.
- `shared_branch_problem` *string or null* — Why it cannot join as a shared branch, when it cannot.
- `default_branch_name` *string*, **required** — The name a join is offered under by default (this machine's name).
- `links` *array of [`LibraryLinkView`](#librarylinkview)*, **required** — Every library this install is a branch of, with whether its client runs.

## LameConfig

LAME encoder tuning (classic `Lame*` settings).

- `target` *string* — `LameTarget`: `quality`, `bitrate`, `constant`, etc. Default `"quality"`.
- `vbr_quality` *integer (int32)* — `LameVBRQuality` (0–9, lower = better). At least 0; default `2`.
- `bitrate_kbps` *integer (int32)* — `LameBitrate` in kbps when target is bitrate/constant. At least 0; default `128`.
- `mode` *string* — `LameMode`: `default`, `mono`, `stereo`. Default `"default"`.
- `downsample_mono` *boolean* — `LameDownsampleMono`. Default `false`.
- `constant_bitrate` *boolean* — `LameConstantBitrate`. Default `false`.

## LeaseView

A library lease as the status page shows it.

- `name` *string*, **required** — Lease name.
- `holder` *string*, **required** — Holder (a node id).
- `token` *integer (int64)*, **required** — Fencing token; a new holder always gets a larger one.
- `held` *boolean*, **required** — True while the lease has not expired.
- `expires_at_ms` *integer (int64)*, **required** — Expiry in Unix milliseconds.

## LeaveBody

`POST /api/config/join/leave` body.

- `library_id` *string*, **required** — The library (parent library id) to leave.

## LeaveStarted

JSON body once leaving a library started.

- `library` *[`LibraryLinkView`](#librarylinkview)*, **required** — The link, now `left`.
- `owner_notified` *boolean*, **required** — Whether the library was told (otherwise it notices the missed heartbeats).

## LeftBranch

The branch a card sign-in left.

- `owner_notified` *boolean*, **required** — Whether the library accepted the leave notice. False when it could not be told: the branch then stays in its list until it is removed there.

## Libraries

`GET /api/libraries`.

- `libraries` *array of [`Library`](#library)*, **required** — The libraries the patron may open.

## Library

One library an app can open.

- `id` *string*, **required** — The library id ([`crate::ids::WHOLE_LIBRARY_ID`] for the whole library).
- `name` *string*, **required** — The library's name.
- `folders` *array of object*, **required** — Folders on disk: none are shown.
- `displayOrder` *integer (int32)*, **required** — The library's place in the list, from 1. At least 0.
- `icon` *string*, **required** — The icon apps show (`audiobookshelf` names a book stack).
- `mediaType` *string*, **required** — Always [`MEDIA_TYPE_BOOK`].
- `provider` *string*, **required** — The metadata provider apps show: none.
- `settings` *[`LibrarySettings`](#librarysettings)*, **required** — Display settings.
- `lastScan` *integer (int64) or null* — Always null.
- `lastScanVersion` *string or null* — Always null.
- `createdAt` *integer (int64)*, **required** — When the library was made, in epoch milliseconds.
- `lastUpdate` *integer (int64)*, **required** — When it last changed, in epoch milliseconds.

## LibraryAnswer

`GET /player/api/libraries/{id}` answer: the library, with its filter
values when asked for (`include=filterdata`).

Type: [`LibraryWithFilterData`](#librarywithfilterdata) or [`Library`](#library)

## LibraryCollection

One collection: a name and the storage key prefixes whose files belong
to it.

- `name` *string*, **required** — The name people see and scope rules match, 1 to 64 bytes, distinct ignoring case.
- `prefixes` *array of string*, **required** — Storage key prefixes, 1 to 8, each a folder relative to the destination's root (no leading `/`); a closing `/` is added when missing, so `kids` matches `kids/…` but not `kidsbooks/…`. At most 8 items.

## LibraryCollections

The library's collections: the `core.collections` document and the
body of `PUT /api/library/collections`.

- `collections` *array of [`LibraryCollection`](#librarycollection)*, **required** — The collections, at most 32, in the order shown. At most 32 items.

## LibraryConsentsResponse

JSON body for `GET /api/config/library-consents`.

- `libraries` *array of [`LibraryLinkView`](#librarylinkview)*, **required** — Every library this install is a branch of.
- `consents` *array of [`RemoteConsentView`](#remoteconsentview)*, **required** — Every plugin request from those libraries, pending or answered.

## LibraryFile

A file of an item, in the expanded item's `libraryFiles`.

- `ino` *string*, **required** — The file's `ino` ([`crate::ids::file_ino`]).
- `metadata` *[`FileMetadata`](#filemetadata)*, **required** — The file's name, size and times.
- `addedAt` *integer (int64)*, **required** — When the file was stored, in epoch milliseconds.
- `updatedAt` *integer (int64)*, **required** — When the file was stored, in epoch milliseconds.
- `fileType` *string*, **required** — Always `audio`.

## LibraryItem

An item: an edition with a stored copy.

- `id` *string*, **required** — The item id: the edition id.
- `ino` *string*, **required** — The item's inode as apps expect one: the edition id again.
- `oldLibraryItemId` *string or null* — Always null.
- `libraryId` *string*, **required** — The library the item is listed in.
- `folderId` *string*, **required** — The folder id: the library id.
- `path` *string*, **required** — A display path: the title.
- `relPath` *string*, **required** — A display path: the title.
- `isFile` *boolean*, **required** — Always false: an item is a folder of tracks.
- `mtimeMs` *integer (int64)*, **required** — When the copy last changed, in epoch milliseconds.
- `ctimeMs` *integer (int64)*, **required** — When the copy was stored, in epoch milliseconds.
- `birthtimeMs` *integer (int64)*, **required** — When the copy was stored, in epoch milliseconds.
- `addedAt` *integer (int64)*, **required** — When the item joined the library, in epoch milliseconds.
- `updatedAt` *integer (int64)*, **required** — When its record last changed, in epoch milliseconds.
- `isMissing` *boolean*, **required** — Always false.
- `isInvalid` *boolean*, **required** — Always false.
- `mediaType` *string*, **required** — Always [`MEDIA_TYPE_BOOK`].
- `media` *[`BookMedia`](#bookmedia)*, **required** — The book.
- `numFiles` *integer (int64)*, **required** — Files in the item. At least 0.
- `size` *integer (int64)*, **required** — Bytes in all its files. At least 0.
- `libraryFiles` *array of [`LibraryFile`](#libraryfile) or null* — The item's files, when the item is fetched expanded.
- `userMediaProgress` *[`MediaProgress`](#mediaprogress) or null* — The patron's progress, when the app asked for it.
- `seriesSequence` *string or null* — The item's place in a series, on a series' book list.

## LibraryItems

`{ "libraryItems": [...] }`: in-progress items and batch reads.

- `libraryItems` *array of [`LibraryItem`](#libraryitem)*, **required** — The items.

## LibraryJobsBody

`POST /api/config/libraries/{library_id}/jobs` body.

- `job_ids` *array of string*, **required** — Job ids on the library.

## LibraryJobsResponse

JSON body for `POST /api/config/libraries/{library_id}/jobs`.

- `jobs` *array of [`JobStatusEntry`](#jobstatusentry)*, **required** — The owner's state of each job this install may see, in request order.

## LibraryLinkView

A library this install is a branch of: ids, origin and state, never key material.

- `library_id` *string*, **required** — The library id.
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
- `keep_copies` *boolean*, **required** — "Keep copies on this computer": the branch takes the library's copies of its person's books into its own storage. The branch takes the library's work and shows its books whether or not this is on.
- `running` *boolean or null* — Whether this install's client for the link is running now; only the join routes report it.

## LibraryScanBody

`POST /api/config/libraries/{library_id}/scan` body.

- `account` *string or null* — Scan only this store account of the library.
- `plugin_instance_id` *string or null* — Pin every planned job to this owner plugin instance.

## LibraryScanPlanned

JSON body after the owner library queued a scan for this install.

- `library_id` *string*, **required** — The parent library's id.
- `created` *array of string*, **required** — Ids of the jobs the owner created.
- `duplicates` *array of string*, **required** — Ids of jobs that were already pending or running.
- `queue_full` *integer (int64)*, **required** — Planned jobs the owner's queue had no room for. At least 0.

## LibrarySettings

A library's display settings.

- `coverAspectRatio` *integer (int32)*, **required** — 1 for square covers, 2 for the 1.6:1 book shape: square. At least 0.
- `disableWatcher` *boolean*, **required** — Always true: the library is not a watched folder.
- `audiobooksOnly` *boolean*, **required** — Always true: audiobooks only.
- `hideSingleBookSeries` *boolean*, **required** — Whether one-book series are hidden from the series list: no.
- `onlyShowLaterBooksInContinueSeries` *boolean*, **required** — Whether continue series skips earlier books: no.
- `epubsAllowScriptedContent` *boolean*, **required** — Always false.
- `markAsFinishedPercentComplete` *number (double) or null* — An item counts as finished at this share listened, in percent.
- `markAsFinishedTimeRemaining` *number (double)*, **required** — An item counts as finished with this many seconds left.

## LibraryStats

`GET /api/libraries/{id}/stats`.

- `totalItems` *integer (int64)*, **required** — Items. At least 0.
- `totalAuthors` *integer (int64)*, **required** — Authors. At least 0.
- `totalGenres` *integer (int64)*, **required** — Genres. At least 0.
- `totalDuration` *number (double)*, **required** — Seconds of audio in all.
- `numAudioTracks` *integer (int64)*, **required** — Audio tracks in all. At least 0.
- `totalSize` *integer (int64)*, **required** — Bytes in all: not counted. At least 0.

## LibraryStatusResponse

`GET /api/library-status` body.

- `library` *[`LibrarySummary`](#librarysummary)*, **required** — Library-wide counters.
- `node_id` *string*, **required** — Id of the node that answered.
- `nodes` *array of [`NodeView`](#nodeview)*, **required** — The nodes the caller administers.
- `capabilities` *[`NodeCapabilities`](#nodecapabilities)*, **required** — What the caller may do with them.

## LibrarySummary

Library-wide counters every node administrator may see.

- `nodes` *integer*, **required** — Nodes in the library's node list. At least 0.
- `live` *integer*, **required** — Nodes whose last heartbeat is recent enough. At least 0.
- `drained` *integer*, **required** — Live nodes an operator drained. At least 0.
- `branch_nodes` *integer*, **required** — Enrolled hosts (nodes with their own database). At least 0.
- `cpu_budget_millicores` *integer (int64)*, **required** — CPU the scheduler may commit across live nodes, in millicores.
- `cpu_reserved_millicores` *integer (int64)*, **required** — CPU the running guests' ceilings add up to, in millicores.
- `memory_budget_bytes` *integer (int64)*, **required** — Memory the scheduler may commit across live nodes, in bytes.
- `memory_reserved_bytes` *integer (int64)*, **required** — Memory the running guests' ceilings add up to, in bytes.
- `running_guests` *integer (int64)*, **required** — Guests running across live nodes.
- `active_jobs` *integer (int64)*, **required** — Jobs queued or running.
- `deliveries` *[`DeliveryBacklog`](#deliverybacklog)*, **required** — Event delivery backlog.
- `leases` *array of [`LeaseView`](#leaseview)*, **required** — Every library lease, held or expired.
- `schema` *[`SchemaStateView`](#schemastateview)*, **required** — The database schema against this binary.

## LibraryWithFilterData

`GET /api/libraries/{id}?include=filterdata`.

- `filterdata` *[`FilterData`](#filterdata)*, **required** — The filter values.
- `issues` *integer (int64)*, **required** — Items with problems: none are reported. At least 0.
- `numUserPlaylists` *integer (int64)*, **required** — The patron's playlists in the library: none. At least 0.
- `library` *[`Library`](#library)*, **required** — The library.

## LinkedIdentitiesResponse

JSON body for `GET /api/auth/oidc/identities`.

- `identities` *array of [`LinkedIdentity`](#linkedidentity)*, **required** — Every sign-in identity linked to the caller's user, the local one included.

## LinkedIdentity

One sign-in identity linked to a user.

- `provider` *string*, **required** — Identity-broker id (`local`, `oidc:google`, …).
- `external_user_id` *string*, **required** — Provider-stable subject.
- `label` *string or null* — Operator-facing label, when the provider supplied one.

## ListeningStats

`GET /api/me/listening-stats`.

- `totalTime` *number (double)*, **required** — Seconds listened in all.
- `items` *map of string to [`ItemListening`](#itemlistening)*, **required** — Seconds listened per item, by item id.
- `days` *map of string to number (double)*, **required** — Seconds listened per day, by `YYYY-MM-DD`.
- `dayOfWeek` *map of string to number (double)*, **required** — Seconds listened per weekday, by English weekday name.
- `today` *number (double)*, **required** — Seconds listened today.
- `recentSessions` *array of [`PlaybackSession`](#playbacksession)*, **required** — The most recent sessions.

## ListingJson

One store listing of an edition.

- `id` *string*, **required** — Listing id.
- `plugin_key` *string*, **required** — The storefront plugin that lists it.
- `marketplace` *string*, **required** — Store marketplace; empty when the store has one.
- `product_id` *string*, **required** — The store's product id.
- `url` *string or null* — The product page, when known.
- `holdings` *array of [`HoldingJson`](#holdingjson)*, **required** — The listing's holdings.

## LocalSessionBody

`POST /player/api/session/local`: a stretch of listening the app made
offline.

- `id` *string*, **required** — The app's own id for the session.
- `libraryItemId` *string*, **required** — The item.
- `currentTime` *number (double)*, **required** — The position, in seconds.
- `timeListening` *number (double) or null* — Seconds listened in the session.
- `duration` *number (double) or null* — The runtime the app measured, in seconds.
- `updatedAt` *integer (int64) or null* — When the app last updated the session, in epoch milliseconds.

## LocalSessionsBody

`POST /player/api/session/local-all`: several offline sessions.

- `sessions` *array of [`LocalSessionBody`](#localsessionbody)*, **required** — The sessions; each is answered on its own.

## LoginBegin

Login-name or email used to start a discoverable-less passkey login.

- `login` *string*, **required** — Login name or email looked up before issuing an authentication challenge.

## LoginBody

`POST /player/login`: a sign-in with a player password.

- `username` *string*, **required** — The patron's sign-in name or email address.
- `password` *string*, **required** — A player password from Account settings; the account password is never accepted.

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

## ManifestFile

One file of a `POST /api/uploads` manifest, as sent.

- `path` *string*, **required** — Relative path, folders kept (`Book/01 Chapter.mp3`); `\` reads as `/`. Refused when absolute, when it climbs out with `..`, or when it is too long or too deep.
- `size` *integer (int64)*, **required** — Size in bytes. At least 0.
- `sha256` *string or null* — Lower-case hex SHA-256 of the file, checked before it is stored.

## MasterKeyStatus

`GET /api/config/master-key` and the `PUT` answer.

- `store` *[`MasterKeyStore`](#masterkeystore)*, **required** — Where the key lives.
- `recovery_copy` *[`RecoveryCopyAnswer`](#recoverycopyanswer)*, **required** — What the operator recorded about a recovery copy of this key. A record for an earlier key (since restored or replaced) reads as `unrecorded`.
- `recorded_at` *string or null* — When that was recorded (RFC 3339, UTC); absent when `unrecorded`.
- `needs_recovery_copy` *boolean*, **required** — Whether the web app should remind the operator to save a copy: the key is in the OS keychain or sealed with `systemd-creds` (so backups of the files directory don't include it) and no copy was recorded as saved.

## MasterKeyStore

Where this install keeps its key.

Type: one of `file`, `password_file`, `keychain`, `systemd_creds`

## MediaCapabilitiesJson

The codecs, containers and encodings this build's media worker handles.

- `decodes` *array of string*, **required** — Audio codecs the worker decodes (`aac`, `alac`, `flac`, `mp3`, `opus`, `vorbis`, and `eac3` in builds with the FFmpeg decoders).
- `reads` *array of string*, **required** — Media types of files the worker opens, compared without case or parameters. Builds with the FFmpeg decoders add WMA, raw AC-3 and E-AC-3, and AMR types.
- `encodes` *array of [`OutputEncodingJson`](#outputencodingjson)*, **required** — Encodings the worker writes.

## MediaKindJson

Carrier of an edition (`editions.media_kind`).

Type: one of `audio`, `ebook`, `print`

## MediaProgress

The patron's progress on one item.

- `id` *string*, **required** — The progress id: the item id (one record per patron and item).
- `userId` *string*, **required** — The patron's id.
- `libraryItemId` *string*, **required** — The item id.
- `episodeId` *string or null* — Always null.
- `mediaItemId` *string*, **required** — The media id: the item id.
- `mediaItemType` *string*, **required** — Always `book`.
- `duration` *number (double)*, **required** — The book's length in seconds, as last reported.
- `progress` *number (double)*, **required** — The share listened, from 0 to 1.
- `currentTime` *number (double)*, **required** — Where the patron is, in seconds.
- `isFinished` *boolean*, **required** — Whether the patron finished it.
- `hideFromContinueListening` *boolean*, **required** — Whether it is hidden from continue listening.
- `ebookLocation` *string or null* — Always null.
- `ebookProgress` *number (double) or null* — Always null.
- `lastUpdate` *integer (int64)*, **required** — When the progress last changed, in epoch milliseconds.
- `startedAt` *integer (int64)*, **required** — When the patron started, in epoch milliseconds.
- `finishedAt` *integer (int64) or null* — When the patron finished, in epoch milliseconds.

## MemberControls

A value for each control: the library's default, or what applies to
one user.

- `link_accounts` *boolean* — May connect store accounts ([`MemberControl::LinkAccounts`]).
- `download` *boolean* — May take copies of the library's files ([`MemberControl::Download`]).
- `edit_metadata` *boolean* — May change metadata ([`MemberControl::EditMetadata`]).
- `upload` *boolean* — May upload books ([`MemberControl::Upload`]); off unless set.
- `request_books` *boolean* — May ask branches for books ([`MemberControl::RequestBooks`]).
- `see_prices` *boolean* — May see prices and store links ([`MemberControl::SeePrices`]).

## MembersBooksBody

`PUT /api/config/branches/members-books` body: a branch, and whether
this library acquires all of its members' books.

- `node_id` *string*, **required** — The branch's node id, as Settings → Branches lists it.
- `acquire` *boolean*, **required** — "Acquire all of this branch's members' books".

## MeResponse

JSON body for `GET /me`.

- `provider` *string*, **required** — Identity provider id (`local` or an integration plugin id).
- `external_user_id` *string*, **required** — Provider-scoped user id stored on the portal identity row.
- `label` *string or null* — Optional display label from the identity row.

## MetaTags

The tags read from an audio file, as apps read them: always empty until
the stored media probe reads tags, so it serializes as `{}`.

Type: object

## MfaPolicy

The host-wide second-factor policy.

- `require_second_factor` *boolean*, **required** — Whether every password sign-in must be followed by a passkey or authenticator code.

## MfaPolicyPut

PUT body for the host-wide second-factor policy.

- `require_second_factor` *boolean*, **required** — When true, password login requires TOTP or the user must use a passkey.

## MissingFileAction

What happens to a stored copy whose file disappears from its destination
outside Bookclerk (noticed by the destination's change feed or listing).

Type: one of `mark_missing`, `restore`

## MultiDestinationMode

Behavior when some output destinations already have a title and others do not.

Type: one of `sync_missing`, `refetch_missing`, `refetch_all`

## NamedRef

An id and a name.

- `id` *string*, **required** — The entity's id.
- `name` *string*, **required** — Its display name.

## NamingProfile

Preset path-template profile.

Select via `output.naming_profile`. Individual `folder_template` /
`file_template` / `chapter_file_template` values override the profile when
set.

Type: one of `audiobookshelf`, `classic`

## Narrator

A narrator with how many items they read.

- `id` *string*, **required** — The name in URL-safe base64, as apps filter by it.
- `name` *string*, **required** — The narrator's name.
- `numBooks` *integer (int64)*, **required** — Items they read that the patron can see. At least 0.

## Narrators

`GET /api/libraries/{id}/narrators`.

- `narrators` *array of [`Narrator`](#narrator)*, **required** — The narrators.

## NewBookmarkBody

`POST /api/library/editions/{id}/bookmarks` body.

- `position_ms` *integer (int64)*, **required** — The place to mark, in milliseconds (at most sixty days). At least 0.
- `title` *string* — A note for it, at most 200 bytes, no control characters; empty when left out.

## NewChannelBody

Body of `POST /api/notifications/channels`.

- `plugin_instance_id` *string*, **required** — A notifier instance that sends to channels; its own settings say where (a webhook URL, a service key).
- `label` *string*, **required** — Name shown in the routing matrix, 1 to 128 bytes on one line.

## NewPersonalBody

Body of `POST /api/me/notifications/destinations`.

- `plugin_instance_id` *string*, **required** — A notifier instance that sends to patrons.
- `address` *map of string to string*, **required** — The address, as the notifier's `address_fields` describe it: a flat object of strings. A key written twice is refused (`422`) rather than one value kept.
- `label` *string or null* — Name shown in pickers; the notifier's own label for the address, or the notifier's name, when absent.

## NewPlayerPasswordBody

`POST /api/auth/player-passwords` body.

- `label` *string*, **required** — A name for it, such as the app or device (1 to 64 bytes once trimmed).

## NewPlayerPasswordJson

`POST /api/auth/player-passwords` answer.

- `password` *[`PlayerPasswordJson`](#playerpasswordjson)*, **required** — The stored password.
- `secret` *string*, **required** — The password itself, shown this once: the library keeps only its digest.

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

- `node_id` *string*, **required** — Branch node.
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

How a node joined the library.

Type: one of `shared`, `branch`

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

Desired state for one node, editable from any node in the library: an
operator label, and the budget the scheduler may commit on this node.
Budgets are shares of the capacity the node detects at heartbeat
(CPU cores and memory, the tighter of the machine and its cgroup).

- `label` *string*, **required** — Operator label shown for this node. Empty is allowed.
- `cpu_budget_percent` *integer (int32)* — Share of detected CPU cores the scheduler may commit to plugin guests on this node (1..=100). At least 1; at most 100.
- `memory_budget_percent` *integer (int32)* — Share of detected memory the scheduler may commit (1..=100). At least 1; at most 100.
- `schedulable` *boolean* — False drains the node: the scheduler places nothing new here and moves its replicas elsewhere. Hand-managed deployments still run.
- `cache_budget_bytes` *integer (int64) or null* — Disk space, in bytes, this node's image caches may use together, overriding the library's `core.cache` budget (16 MiB to 1 TiB). Absent means the library budget applies. At least 16777216; at most 1099511627776.

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
- `public_origin` *string or null* — The address other installs reach this branch at, as it last reported it; `null` for a local branch and for a shared node. A branch with one is a remote branch of this library.

## NodeView

One node with its latest heartbeat, budget and owner.

- `node_id` *string*, **required** — Node id.
- `library_id` *string*, **required** — Library the node belongs to.
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
- `kind` *[`NodeKind`](#nodekind)*, **required** — `shared` for a node sharing this library's database, `branch` for an enrolled branch.
- `relationship` *[`BranchRelationship`](#branchrelationship)*, **required** — How the node relates to this library: `shared_branch` (shares its database), `remote_branch` (a branch reporting a public address in its heartbeat, `status.public_origin`) or `local_branch` (a branch with none).
- `owner_user_id` *integer (int64) or null* — User id of the node's owner; `null` when unassigned.
- `share_books` *boolean or null* — A branch's sharing setting, as it last reported it; `null` for a shared node.
- `include_branches` *boolean or null* — A branch's include-branches setting, as it last reported it; `null` for a shared node.
- `owner` *[`NodeOwnerSummary`](#nodeownersummary) or null* — The owner, when the node has one and the user still exists.

## NotificationCatalog

Everything the routing screens pick from.

- `types` *array of [`NotificationTypeJson`](#notificationtypejson)*, **required** — Host types first, then plugin types by id.
- `notifiers` *array of [`NotifierJson`](#notifierjson)*, **required** — Notifier instances installed on this server.

## NotificationTypeJson

One notification type a routing rule or a patron choice can name.

- `id` *string*, **required** — Qualified id: a host type (`test`) or `<plugin id>:<type>`.
- `audience` *string*, **required** — Who it is for: `operational` (routed to channels) or `patron` (offered to patrons).
- `severity` *string*, **required** — Default severity: `info`, `warning` or `critical`.
- `title` *string*, **required** — Title shown in the routing matrix and on each notification.
- `source` *string*, **required** — The plugin that raises it; empty for a host type.

## NotifierJson

One notifier instance.

- `plugin_instance_id` *string*, **required** — Plugin instance id.
- `plugin_key` *string*, **required** — Canonical plugin key.
- `name` *string*, **required** — Name shown in pickers.
- `channel` *boolean*, **required** — Whether administrators can make library channels on it.
- `personal` *boolean*, **required** — Whether patrons can add their own address on it.
- `address_fields` *array of [`AddressFieldJson`](#addressfieldjson)*, **required** — What a patron enters for a personal destination.

## OAuthEnrollAnswer

`POST /api/node/v1/enroll/oauth` answer.

- `library_id` *string*, **required** — The library id.
- `node_id` *string*, **required** — The node id recorded for the joining install.
- `connection` *[`ConnectionType`](#connectiontype)* — The connection type recorded (`branch` when left out).
- `bootstrap` *string or null* — For a shared branch: the [`crate::SharedBranchBootstrap`] sealed to the node's box key ([`crate::seal_shared_branch_bootstrap`]), as base64url without padding.
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

## OfferJson

One notifier offered to patrons for a type.

- `plugin_instance_id` *string*, **required** — A notifier instance that sends to personal destinations.
- `default_on` *boolean*, **required** — Whether a patron who chose nothing gets the type there.

## OffersBody

Body of `PUT /api/notifications/offers/{type}`.

- `offers` *array of [`OfferJson`](#offerjson)*, **required** — The type's offers, replacing the old ones; at most 32. Empty keeps the type in patrons' inboxes only. At most 32 items.

## OfferTable

Every patron offer, by type.

- `offers` *map of string to array of [`OfferJson`](#offerjson)*, **required** — Type id to its offers.

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

## OidcConfigPut

Operator PUT body that replaces the broker table (secrets omitted or replaced).

- `enabled` *boolean*, **required** — Whether the broker should accept authorize starts after this write.
- `allowed_email_domains` *array of string* — Replacement broker-wide email domain allowlist.
- `providers` *array of [`OidcProviderPut`](#oidcproviderput)* — Full replacement list of IdPs (omitted secrets keep the previous generation).
- `public_origin` *string or null* — `integrations.public_origin`. Omit to leave unchanged; empty string clears.

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

## OpenSessionBody

`POST /api/playback/sessions` body.

- `edition_id` *string*, **required** — The edition to play (1 to 128 bytes).
- `player` *[`PlayerKindParam`](#playerkindparam)* — What kind of player asks; `web` when left out.
- `player_name` *string* — The player's own name, shown in listening history. Control characters are dropped and at most 64 bytes are kept.

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

## OutputCopy

One copy of every acquired book: a format at a quality.

- `format` *[`CopyFormat`](#copyformat)*, **required** — The copy's format.
- `quality` *[`CopyQuality`](#copyquality)*, **required** — The copy's quality.

## OutputDomainChange

Body of `PUT /api/config/domains/core.output`: the settings after the
change, or a replay note.

Type: [`OutputDomainResponse`](#outputdomainresponse) or [`DomainReplayed`](#domainreplayed)

## OutputDomainResponse

Body of `GET /api/config/domains/core.output` and of a `PUT` that was
applied: the revision metadata and the `[output]` document.

Includes every field of [`DomainRevision`](#domainrevision).

- `document` *[`OutputSettingsV1`](#outputsettingsv1)*, **required** — The settings at that revision (schema version 1).

## OutputEncodingJson

One encoding the media worker writes.

- `name` *string*, **required** — The output format id that selects it (`m4b`, `mp3` or `opus`).
- `advanced` *boolean*, **required** — True when settings offer it only under their advanced options, because fewer players open it than M4B and MP3.

## OutputFormat

Post-download output formatting. Default is enriched M4B (tags + chapters).
`split_mp3_by_size` parses but is not implemented yet: the acquire
pipeline rejects a job that selects it before downloading.

Type: one of `enriched_m4b`, `single_mp3`, `split_mp3_by_chapter`, `split_mp3_by_size`, `opus`, `none`

## OutputSettingsV1

Typed `core.output` document, schema version 1.

Field meanings match [`OutputConfig`]. Unknown fields are rejected, inside
the nested `lame` and `replacement_characters` objects too, so a document
written by a newer binary under the same schema version fails closed
instead of being silently truncated on the next write.

- `format` *[`OutputFormat`](#outputformat)*, **required** — Post-acquire packaging format.
- `copies` *array of [`OutputCopy`](#outputcopy)* — Further copies of every acquired book beside the main output, at most 8; empty keeps one copy. Each needs a quality that suits its format, and none may repeat another or the main output's copy. At most 8 items.
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
- `keep_codec_in_m4b` *boolean* — Keep Opus, FLAC, ALAC, E-AC-3 and AC-4 audio unchanged inside an M4B instead of encoding it to AAC (advanced; off by default).
- `fixup_metadata` *boolean*, **required** — Embed tags, cover, and chapters natively.
- `chapter_json` *[`ChapterJsonMode`](#chapterjsonmode)*, **required** — Chapter JSON sidecars.
- `save_metadata_json` *boolean*, **required** — Persist raw catalog API JSON.
- `cover_size` *string*, **required** — Cover image size (`500`, `1215`, another pixel size, or `native`).
- `chapter_layout` *string*, **required** — Preferred chapter API layout (`tree`, `flat`, or `both`).
- `overwrite_existing` *boolean*, **required** — Re-acquire when media already exists at the destination.
- `multi_destination` *[`MultiDestinationMode`](#multidestinationmode)*, **required** — Policy when only some enabled destinations have a title.
- `missing_files` *[`MissingFileAction`](#missingfileaction)* — What happens to a stored copy whose file disappears from its destination outside Bookclerk. Absent in documents written before the setting existed, which read as the default.
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

## OverrideSwitchChanges

Changes to some of one Administrator's overrides: `on` or `off` for
them alone, or `default` to follow the library's default again; an
absent switch keeps what it had.

- `manage_members` *[`AdminSwitchChoice`](#adminswitchchoice) or null* — Change to [`AdminControl::ManageMembers`].
- `set_patron_limits` *[`AdminSwitchChoice`](#adminswitchchoice) or null* — Change to [`AdminControl::SetPatronLimits`].
- `library_scan` *[`AdminSwitchChoice`](#adminswitchchoice) or null* — Change to [`AdminControl::LibraryScan`].
- `run_jobs` *[`AdminSwitchChoice`](#adminswitchchoice) or null* — Change to [`AdminControl::RunJobs`].
- `fix_catalog` *[`AdminSwitchChoice`](#adminswitchchoice) or null* — Change to [`AdminControl::FixCatalog`].
- `invite_branches` *[`AdminSwitchChoice`](#adminswitchchoice) or null* — Change to [`AdminControl::InviteBranches`].
- `approve_requests` *[`AdminSwitchChoice`](#adminswitchchoice) or null* — Change to [`AdminControl::ApproveRequests`].

## OwnerBootstrapped

JSON body after the first Owner was created.

- `ok` *boolean*, **required** — Always `true` on this success path.
- `user_id` *integer (int64)*, **required** — `users.id` of the new Owner.
- `claim_ticket` *string*, **required** — Single-use claim ticket that signs the Owner in once (valid for seven days).
- `invite_url` *string*, **required** — Invite link carrying the ticket, for the browser.
- `login_name` *string or null* — Login name the Owner signs in with, when one was given.
- `has_password` *boolean*, **required** — Whether a password was set, so password sign-in works right away.

## OwnerChoice

The consent page's connection block for an Owner, who may pick shared branch.

- `choice` *boolean*, **required** — Always `true`.
- `default` *string*, **required** — Always `branch`.
- `shared_branch_available` *boolean*, **required** — Whether shared branch may be chosen right now.
- `shared_branch_reason` *string or null* — Why shared branch is shown disabled, when it is.
- `step_up` *array of string*, **required** — The second factors the Owner can step up with (`totp`, `passkey`).

## OwnerInfo

`GET /api/node/v1/info` answer.

- `library_id` *string*, **required** — The library id.
- `library_name` *string*, **required** — A name to show for the library.
- `origin` *string*, **required** — The library's canonical origin: the one the node handshake signs and the authorize endpoint lives under.
- `transport` *string*, **required** — The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).
- `enrollment` *boolean*, **required** — Whether the library lets new branches join (`core.branch_enrollment`).
- `spki_fingerprint` *string or null* — In `self_signed` mode: base64url SHA-256 of the certificate's SubjectPublicKeyInfo, which the joining operator confirms against the owner's Settings → Branches page.
- `software_version` *string* — The library's Bookclerk version.
- `ancestry` *array of string* — The library's ancestry: its own library id first, then the library id of every library above it up to the root. A joining library refuses a parent whose ancestry names its own library id, so a tree of libraries never becomes a cycle.

## PaceModeSchema

The `mode` of a [`PaceRequestBody`], for the API reference.

Type: one of `default`, `custom`, `unlimited`

## PaceRequestBody

Request body of the pace route, for the API reference (the route reads
the raw bytes with `bookclerk_library::pacing::parse_pace_request`).

- `mode` *[`PaceModeSchema`](#pacemodeschema) or null* — `default`, `custom` or `unlimited`. Leave it out to only end the cool-down (then `clear_cooldown` must be `true`) and keep the stored pace.
- `downloads_per_day` *integer (int32) or null* — Custom downloads per rolling 24 hours; `custom` only. At least 1; at most 10000.
- `min_gap_seconds` *integer (int32) or null* — Custom seconds between downloads; `custom` only. At least 0; at most 86400.
- `clear_cooldown` *boolean or null* — Also end a cool-down the store asked for.

## PaceView

How fast Bookclerk downloads from one store account when it starts the
downloads itself, for the Accounts page.

- `mode` *string*, **required** — `default` (the store's pace), `custom` or `unlimited`.
- `downloads_per_day` *integer (int32) or null* — Downloads allowed in any rolling 24 hours now in force; `null` for no daily budget. At least 0.
- `min_gap_seconds` *integer (int64)*, **required** — Seconds between two downloads now in force. At least 0.
- `store_downloads_per_day` *integer (int32) or null* — The store's declared daily budget; `null` when it declares none or its plugin is not deployed here. At least 0.
- `store_min_gap_seconds` *integer (int64) or null* — The store's declared gap in seconds; `null` when its plugin is not deployed here. At least 0.
- `custom_downloads_per_day` *integer (int32) or null* — Custom daily budget the operator set, when `mode` is `custom`. At least 0.
- `custom_min_gap_seconds` *integer (int32) or null* — Custom gap the operator set, when `mode` is `custom`. At least 0.
- `downloads_last_day` *integer*, **required** — Downloads from this account in the last 24 hours, by hand or not. At least 0.
- `cooldown_until` *string or null* — End of the cool-down the store asked for (RFC 3339), when one is running.
- `next_automatic_at` *string or null* — When the next automatic download may start (RFC 3339); `null` when one may start now.

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

## PatchDefaultAdminControls

`PATCH /api/users/default-admin-controls` body.

- `changes` *[`DefaultSwitchChanges`](#defaultswitchchanges)*, **required** — The default switches to change and their new values. Switches it does not name keep their stored value, so two Owners editing different switches do not undo each other.

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

## PatchUserAdminControls

`PATCH /api/users/{id}/admin-controls` body.

- `changes` *[`OverrideSwitchChanges`](#overrideswitchchanges)*, **required** — The switches to change for this Administrator: `on` or `off` sets the switch for them alone, `default` follows the library's default again. Switches it does not name keep what they had, so two Owners editing different switches do not undo each other.

## PatchUserRequest

Partial profile, role, or status patch for a first-party user.

- `role` *string or null* — Replacement role (`owner` / `administrator` / `member`) when present.
- `status` *string or null* — Replacement status (`active` / `disabled`) when present.
- `display_name` *string or null* — Replacement display name when present.
- `login_name` *string or null* — Replacement local login name when present.
- `email` *string or null* — Replacement email when present.

## PathSanitizationMode

How to sanitize storage path segments when [`crate::OutputConfig::replacement_characters`]
is empty (explicit rules always win).

Type: one of `auto`, `windows`, `posix`, `s3`, `none`

## PatronOfferJson

One notifier a patron may pick for a type.

- `plugin_instance_id` *string*, **required** — The notifier instance.
- `name` *string*, **required** — Its name.
- `default_on` *boolean*, **required** — Whether the type goes there when the patron chose nothing.

## PatronScope

Which part of the library a patron sees.

- `rules` *array of [`ScopeRule`](#scoperule)* — Visibility rules, at most 16 and 1,024 bytes as JSON. Empty means every book is visible. At most 16 items.
- `explicit` *[`ExplicitContent`](#explicitcontent)* — Whether books marked explicit are visible; left out means `allow`.

## PatronSettings

A patron's notification settings.

- `types` *array of [`PatronTypeJson`](#patrontypejson)*, **required** — Patron types administrators offer a notifier for.
- `destinations` *array of [`DestinationJson`](#destinationjson)*, **required** — The patron's own destinations.
- `notifiers` *array of [`NotifierJson`](#notifierjson)*, **required** — Notifiers the patron can add a destination on.

## PatronTypeJson

One patron type and what the patron chose for it.

- `id` *string*, **required** — Type id.
- `title` *string*, **required** — Title.
- `offers` *array of [`PatronOfferJson`](#patronofferjson)*, **required** — Notifiers offered for it.
- `chosen` *boolean*, **required** — Whether the patron made a choice; without one the defaults apply.
- `enabled` *boolean*, **required** — The patron's preference, not a delivery result: `false` keeps the type in the inbox only. `true` when nothing is chosen, in which case the offers' defaults pick where it goes.
- `destination_ids` *array of string*, **required** — The destinations the patron saved for it, when chosen. These are saved choices, not live targets: routing skips one whose notifier is no longer offered for the type, so check `offers` before showing one as eligible or sending the list back.

## Permissions

What a patron may do, as apps read it.

- `download` *boolean*, **required** — Whether the patron may download items to the device.
- `update` *boolean*, **required** — Always false: apps edit nothing.
- `delete` *boolean*, **required** — Always false.
- `upload` *boolean*, **required** — Always false.
- `createEreader` *boolean*, **required** — Always false.
- `accessAllLibraries` *boolean*, **required** — Always true: the libraries listed are the ones the patron sees.
- `accessAllTags` *boolean*, **required** — Always true.
- `accessExplicitContent` *boolean*, **required** — Always true: scope, not an explicit flag, decides visibility.
- `selectedTagsNotAccessible` *boolean*, **required** — Always false.

## PinBody

Body of `PUT /api/library/listings/{id}/pin`.

- `edition_id` *string*, **required** — The edition the listing sells.

## Ping

`GET /ping`: `{ "success": true }`.

- `success` *boolean*, **required** — Always true.

## PinJson

A listing's pin.

- `listing_id` *string*, **required** — The listing.
- `edition_id` *string*, **required** — The edition it is pinned to, and now sells.
- `moved_from` *string or null* — The edition it sold before, when the pin moved it; absent when it already sold the pinned edition.

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

## PlaybackManifestJson

`GET /api/library/editions/{id}/playback` answer.

- `edition_id` *string*, **required** — The edition.
- `work_id` *string or null* — Its work.
- `title` *string*, **required** — The edition's display title.
- `playable` *boolean*, **required** — True when this node holds a copy the caller may play. False when the library has not acquired the edition, no destination of this node holds it, or the caller may not take it.
- `duration_ms` *integer (int64) or null* — Runtime in milliseconds: the last player's report when the caller has one, else the catalog's, to the minute; null when neither is known. At least 0.
- `tracks` *array of [`TrackJson`](#trackjson)*, **required** — The tracks in play order; empty when not playable. A session's answer adds the stream paths.
- `progress` *[`ProgressJson`](#progressjson) or null* — The caller's progress; null before they first play it.
- `bookmarks` *array of [`BookmarkJson`](#bookmarkjson)*, **required** — The caller's bookmarks, in timeline order.
- `chapters` *array of [`ChapterSpan`](#chapterspan)*, **required** — The chapters the edition plays, in pre-order, each ending where the next starts and the last at `duration_ms`; empty when it has none.

## PlaybackSession

A playback session, as `POST /api/items/{id}/play` answers.

- `id` *string*, **required** — The session id; apps send it back to sync and close.
- `userId` *string*, **required** — The patron's id.
- `libraryId` *string*, **required** — The library id.
- `libraryItemId` *string*, **required** — The item id.
- `bookId` *string*, **required** — The media id: the item id.
- `episodeId` *string or null* — Always null.
- `mediaType` *string*, **required** — Always [`MEDIA_TYPE_BOOK`].
- `mediaMetadata` *[`BookMetadata`](#bookmetadata)*, **required** — The book's metadata.
- `chapters` *array of [`Chapter`](#chapter)*, **required** — The book's chapters.
- `displayTitle` *string*, **required** — The title apps show.
- `displayAuthor` *string*, **required** — The author apps show.
- `coverPath` *string or null* — A cover path apps test for presence.
- `duration` *number (double)*, **required** — The book's length in seconds.
- `playMethod` *integer (int32)*, **required** — How the app plays: 0, direct play of the stored files. At least 0.
- `mediaPlayer` *string*, **required** — The player the app reported.
- `deviceInfo` *[`SessionDevice`](#sessiondevice)*, **required** — The device the session was opened from.
- `serverVersion` *string*, **required** — [`crate::COMPATIBLE_SERVER_VERSION`].
- `date` *string*, **required** — The day the session started, `YYYY-MM-DD` in UTC.
- `dayOfWeek` *string*, **required** — The weekday the session started, in English.
- `timeListening` *number (double)*, **required** — Seconds listened in this session.
- `startTime` *number (double)*, **required** — Where the session started, in seconds.
- `currentTime` *number (double)*, **required** — Where the patron is, in seconds.
- `startedAt` *integer (int64)*, **required** — When the session started, in epoch milliseconds.
- `updatedAt` *integer (int64)*, **required** — When it last synced, in epoch milliseconds.
- `audioTracks` *array of [`AudioTrack`](#audiotrack)*, **required** — The tracks to play, in order.
- `libraryItem` *[`LibraryItem`](#libraryitem) or null* — The item, expanded.

## PlaybackSyncBody

`POST /api/playback/sessions/{id}/sync` and `.../close` body: where
the listener is.

- `position_ms` *integer (int64)*, **required** — Position on the edition's timeline, in milliseconds from its start (at most sixty days). At least 0.
- `listened_ms` *integer (int64)*, **required** — Milliseconds listened since this session's last report, as the player counted them; left out reads as 0. At least 0.
- `duration_ms` *integer (int64) or null* — The edition's whole runtime in milliseconds, when the player knows it; left out or `null` keeps the stored one. At least 0.

## PlayBody

`POST /player/api/items/{id}/play`: what an app asks for when it starts
playing. Tracks always play directly, whatever it asks.

- `deviceInfo` *[`DeviceInfoBody`](#deviceinfobody) or null* — The device, named in the patron's session list.
- `forceDirectPlay` *boolean or null* — Whether the app asks for direct play; tracks always play directly.
- `forceTranscode` *boolean or null* — Whether the app asks for transcoding; it gets direct tracks too.
- `mediaPlayer` *string or null* — The app's player, shown in the session list.

## PlayerConsentBody

`POST /api/auth/player-consent` body.

Includes every field of [`PlayerConsentParams`](#playerconsentparams).

- `approve` *boolean*, **required** — True to let the app sign in; false to decline.

## PlayerConsentPage

`GET /api/auth/player-consent` answer: what the consent page shows.

- `app_name` *string*, **required** — The app asking, as the page names it.
- `display_name` *string or null* — The signed-in patron's display name.

## PlayerConsentParams

The consent page's parameters, as the app passed them on.

- `redirect_uri` *string*, **required** — The return address from `/player/auth/openid`: the library's mobile-redirect path with the app's address as `app`.
- `code_challenge` *string*, **required** — The PKCE S256 challenge.
- `state` *string or null* — The app's state.

## PlayerConsentRedirect

`POST /api/auth/player-consent` answer.

- `redirect` *string*, **required** — Where to send the browser: the library's own way back to the app, with a code or the refusal.

## PlayerKindParam

The kind of first-party player that opens a session. Outside player
apps open theirs through the player API.

Type: one of `web`, `desktop`

## PlayerLoginResponse

The answer to a sign-in, a token refresh, `POST /api/authorize` and
the browser sign-in's callback.

- `user` *[`User`](#user)*, **required** — The signed-in patron, with the card's tokens.
- `userDefaultLibraryId` *string*, **required** — The library an app opens first: the whole library.
- `serverSettings` *[`ServerSettings`](#serversettings)*, **required** — Settings apps read at sign-in.
- `ereaderDevices` *array of object*, **required** — E-reader devices: always empty, the API serves no ebooks.
- `Source` *string*, **required** — How the server was installed; apps only display it.

## PlayerPasswordJson

A player password as Account settings lists it.

- `id` *string*, **required** — Its id, for removing it.
- `label` *string*, **required** — The name the patron gave it.
- `created_at` *string*, **required** — When it was made (RFC 3339).
- `last_used_at` *string or null* — When it last signed an app in (RFC 3339), if ever.

## PlayerPasswordsJson

`GET /api/auth/player-passwords` answer.

- `passwords` *array of [`PlayerPasswordJson`](#playerpasswordjson)*, **required** — The patron's player passwords, newest first.

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
- `setting_destinations` *array of [`PluginSettingDestinationApproval`](#pluginsettingdestinationapproval)* — Setting destinations to approve, from the response's `setting_destinations`. An approved destination stays approved while a setting still names it; one left out stays pending, and one no setting names any more is withdrawn.

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
- `setting_destinations` *array of [`PluginSettingDestinationView`](#pluginsettingdestinationview)*, **required** — The destination each network setting (`network = "fetch"` or `"tcp"`) names on each instance, and whether it is approved. Empty when the plugin declares no network setting. A `pending` row is not reachable until it is approved; `covered` is false while one is.

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
- `owner_node_id` *string or null* — Node the instance is local to; `null` for a library-wide instance.

## PluginInstanceDeleted

`DELETE /api/config/plugin-instances/{id}` body.

- `plugin_instance_id` *string*, **required** — Id of the instance that was removed.
- `deleted` *boolean*, **required** — Always true: removing an instance that is already gone answers the same way.

## PluginInstancesResponse

`GET /api/config/plugin-instances` body: every instance as this node
sees it.

- `node_id` *string*, **required** — Id of the node that answered; its deployments carry its own observations.
- `instances` *array of [`PluginInstanceView`](#plugininstanceview)*, **required** — Every plugin instance in the library.

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
- `storage_sync` *[`StorageSyncView`](#storagesyncview) or null* — How outside changes at this destination are followed; absent until a storage instance's first sync pass.

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

## PluginSettingColumn

One column of a `table` setting in the Settings UI.

- `key` *string*, **required** — Record field the column fills in each row.
- `label` *string*, **required** — Column heading.
- `value_type` *string*, **required** — UI widget hint (`boolean`, `string`, or `number`).
- `choices` *array of [`PluginSettingChoice`](#pluginsettingchoice) or null* — Dropdown values of an `enum` column; omitted otherwise.
- `default` *string or null* — String form of the value a new row starts with; omitted when there is no default.
- `required` *boolean* — True when every row must have a value in this column.

## PluginSettingDestinationApproval

A setting destination the operator approves.

- `setting` *string*, **required** — The setting key.
- `destination` *string*, **required** — The destination exactly as the consent response shows it.

## PluginSettingDestinationView

One network setting's destination on one plugin instance.

- `instanceId` *string or null* — The instance whose settings name it; absent when the plugin has no instance yet and the row shows the setting's default.
- `setting` *string*, **required** — The setting key, the source the grant is stored under.
- `label` *string*, **required** — The setting's label (its key when it has none).
- `network` *string*, **required** — `fetch` (HTTP requests) or `tcp` (a raw socket).
- `destination` *string or null* — The destination: `scheme://host:port` for a fetch, `host:port` for TCP (IPv6 in brackets); absent when the setting is unset or invalid.
- `scope` *string or null* — What kind of host it is: `name` (may resolve to a local-network address, for this name only), `publicAddress`, `localAddress` (this one local-network address) or `restricted` (loopback, link-local or a name such as `localhost`, which also needs an address grant).
- `tls` *boolean or null* — True for an `https` fetch, whose certificate is verified; false for plain HTTP and raw TCP, which trust the network's DNS answer.
- `state` *string*, **required** — `approved`, `pending` (awaiting consent), `unset` or `invalid`.
- `error` *string or null* — Why the value names no destination, when `state` is `invalid`.

## PluginSettingOption

One editable setting shown in the Settings UI.

- `key` *string*, **required** — Dotted `config.toml` key for this option.
- `label` *string*, **required** — Operator-facing field caption.
- `value` *string*, **required** — Current string form of the setting.
- `value_type` *string*, **required** — UI widget hint (`boolean`, `string`, `number`, `secret`, or `table`).
- `choices` *array of [`PluginSettingChoice`](#pluginsettingchoice) or null* — Optional dropdown values; omitted for free-form fields.
- `secret` *boolean* — True for a `secret` setting: `value` is never the stored value.
- `secret_set` *boolean or null* — For a secret setting, whether the instance document names a sealed value.
- `columns` *array of [`PluginSettingColumn`](#pluginsettingcolumn) or null* — Columns of a `table` setting, in display order; omitted for other kinds. `value` is then the rows as JSON text.

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
- `storage_sync` *[`StorageSyncView`](#storagesyncview) or null* — How outside changes at this destination are followed; absent for other plugins and before a destination's first sync pass.
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
- `role` *string*, **required** — The user's role (`owner`, `administrator`, or `member`).

## PortalSessionKind

Where a user's session token is presented.

A token works only in its own place: a browser token only as the session
cookie, a library card only as `Authorization: Bookclerk-Clerk`, and a
player card's tokens only on the `/player` API.

Type: one of `browser`, `clerk`, `player`

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
- `channel` *[`PortalSessionKind`](#portalsessionkind)*, **required** — How the session is used: `browser` for a browser session, `clerk` for a library card, whose `client_label` is the name the clerk gave itself, `player` for a player app's card (`/player`), whose `client_label` is the player password's label or the app's name.

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

## ProgressBatchEntry

One entry of `PATCH /player/api/me/progress/batch/update`.

- `libraryItemId` *string*, **required** — The item.
- `currentTime` *number (double) or null* — The position, in seconds.
- `duration` *number (double) or null* — The runtime, in seconds.
- `isFinished` *boolean or null* — Whether the book is finished.

## ProgressBody

`PATCH /player/api/me/progress/{id}`: a change to the patron's progress.

- `currentTime` *number (double) or null* — The position, in seconds.
- `progress` *number (double) or null* — The share of `duration` listened, 0 to 1, read when `currentTime` is absent.
- `duration` *number (double) or null* — The runtime, in seconds.
- `isFinished` *boolean or null* — Whether the book is finished.
- `hideFromContinueListening` *boolean or null* — True to take the item off Continue listening.

## ProgressEditBody

`PUT /api/playback/progress/{edition_id}` body: what to set. A field
left out keeps its stored value.

- `position_ms` *integer (int64) or null* — Position in milliseconds (at most sixty days). At least 0.
- `duration_ms` *integer (int64) or null* — The edition's runtime in milliseconds, when the page knows it. At least 0.
- `is_finished` *boolean or null* — True marks the edition and its work finished; false clears the edition's flag (the work's stays, and is changed with the work's own state). Left out, the position decides.

## ProgressJson

One person's progress in one edition.

- `edition_id` *string*, **required** — The edition.
- `work_id` *string or null* — The edition's work, when known: what a Continue listening shelf reads the title and cover from.
- `position_ms` *integer (int64)*, **required** — Position on the edition's timeline, in milliseconds. At least 0.
- `duration_ms` *integer (int64) or null* — The edition's runtime in milliseconds as the last player reported it; null until one has. At least 0.
- `is_finished` *boolean*, **required** — True once the caller reached the end of this edition.
- `started_at` *string*, **required** — When the caller first played it (RFC 3339).
- `finished_at` *string or null* — When the caller last reached the end (RFC 3339).
- `updated_at` *string*, **required** — When the progress last changed (RFC 3339).

## ProgressPage

`GET /api/playback/progress` answer.

- `items` *array of [`ProgressJson`](#progressjson)*, **required** — The caller's progress rows, most recently played first. Editions their patron scope no longer shows are left out, so a page can be shorter than `limit` with more to come.
- `next_cursor` *string or null* — Cursor of the next page; absent on the last.

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
- `incomplete` *boolean* — True when the batch deadline cut this lookup short (the hints are the query's trusted seeds alone), or when the hints could not be checked against the card's recording in time: a client should not keep the answer and asks again later. Omitted (false) for a finished lookup.

## PutActiveVersion

`PUT /api/config/plugins/{key}/active` body.

- `version` *string*, **required** — Installed version to activate.

## PutBranchEnrollment

`PUT /api/config/branch-enrollment` body.

- `enabled` *boolean*, **required** — Whether branch nodes may enroll into this library.

## PutBranches

`PUT /api/config/branches` body.

- `enrollment` *boolean*, **required** — "Let new branches join" (`core.branch_enrollment.enabled`).

## PutEventsDomainRequest

JSON body for `PUT /api/config/domains/core.events`.

- `expected_revision` *integer (int64)*, **required** — Revision the caller read. A mismatch is a conflict.
- `retention_days` *integer (int64)*, **required** — Replacement `[events]` document. At least 0.
- `dead_letter_retention_days` *integer (int64)*, **required** — Replacement dead-letter retention. At least 0.
- `concurrency` *integer (int32)*, **required** — Replacement concurrency cap. At least 0.

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
- `cache_budget_bytes` *integer (int64) or null* — This node's image cache budget in bytes (16 MiB to 1 TiB), overriding the library's `cache.budget_bytes`. `null` removes the override; leaving the field out keeps it as it is. At least 0.

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

## PutUserControls

`PUT /api/users/{id}/controls` body.

- `overrides` *[`ControlOverrides`](#controloverrides)*, **required** — The controls set for this patron alone; an absent or `null` control follows the library's default for a Member and is allowed for an Administrator, so `{}` clears every override.

## PutUserScope

`PUT /api/users/{id}/scope` body.

- `scope` *[`PatronScope`](#patronscope) or null* — The patron's own scope, or `null` to return them to the library's default.

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

## ReadBody

Body of `POST /api/me/notifications/inbox/read`.

- `notification_id` *string or null* — One notification to mark read; absent or `null` marks the whole inbox read.

## ReadMarked

How many entries a read marked.

- `marked` *integer (int64)*, **required** — Entries that were unread and are now read. At least 0.

## ReceivedJson

Where a file's bytes stand after a `PATCH`.

- `memberNo` *integer (int32)*, **required** — The file's number. At least 0.
- `received` *integer (int64)*, **required** — Bytes received; send the next `PATCH` from here. At least 0.
- `complete` *boolean*, **required** — True once every byte has arrived.

## Recommendation

One ranked recommendation (typically an unowned storefront title).

- `work_id` *string or null* — Library `works.id` of the work the card's edition belongs to, when the library's data model places it (a wish, or a store listing from the shared listing pool). Cards for one work are one card.
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
- `is_explicit` *boolean or null* — When `Some(true)`, a storefront marks the edition explicit (adult) content; `None` when none said (a wishlist entry says nothing).
- `wishers` *array of [`QueueWisher`](#queuewisher)* — Distinct people who have this title on an open wishlist.
- `wish_count` *integer (int64)* — Number of distinct users who wishlisted this title (may exceed `wishers.len()`).
- `seed_title` *string or null* — Title of the book or wish the storefront was asked about to find this card, when it came from such an expansion.
- `starter_genre` *string or null* — Starter genre this card was browsed from while the taste was thin.
- `available_through` *array of [`BranchAvailability`](#branchavailability)* — Branches of this library that currently list this title (they answered a catalog lookup for its work). Filled when the feed is read, never stored with it.

## RecoveryCopyAnswer

What the operator recorded about a recovery copy of the key in use.

Type: one of `unrecorded`, `saved`, `skipped`

## RecoveryCopyChoiceBody

What the operator did about a recovery copy.

Type: one of `saved`, `skipped`

## RecoveryCopyRequest

`POST /api/config/master-key/recovery-copy` body.

- `password` *string*, **required** — The password the copy is wrapped with: at least 12 characters. Keep it apart from the copy; without it the copy can't be restored.

## RecoveryCopyStateRequest

`PUT /api/config/master-key/recovery-copy/state` body.

- `state` *[`RecoveryCopyChoiceBody`](#recoverycopychoicebody)*, **required** — What the operator did.

## RedeemBody

JSON body for `POST /redeem` (claim ticket plus retry nonce).

- `ticket` *string*, **required** — Raw claim-ticket token to redeem into a portal session.
- `nonce` *string*, **required** — Browser-generated one-time nonce persisted across HTTP retries.
- `password` *string or null* — Required when the linked local user has no password (invite / reset).

## RegistrationFinish

`POST /api/auth/passkeys/register/finish` body: the browser's attestation
plus the challenge id from `register/begin`.

The credential is read straight into WebAuthn's typed attestation, so the
handler never decodes caller JSON itself.

- `challenge_id` *string*, **required** — Server-issued challenge id (consumed once; 5-minute TTL).
- `credential` *any JSON*, **required** — W3C PublicKeyCredential as the browser serializes it: a registration attestation (`response.attestationObject`) or an authentication assertion (`response.authenticatorData`, `signature`, `userHandle`), binary fields base64url-encoded.
- `name` *string or null* — Label stored with the credential (`Passkey` when empty).

## RemoteConsentView

An owner library's request to run a plugin on this install, and what the operator answered.

- `library_id` *string*, **required** — The parent library's id.
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

## RenameBody

Body of `PATCH /api/notifications/channels/{id}`.

- `label` *string*, **required** — The new name, 1 to 128 bytes on one line.

## ReplacedCard

The library card a branch join replaced.

- `card_removed` *boolean*, **required** — Whether the card was dropped here. False when it could not be read or dropped: it is then still listed under Library cards, and the person signs it out there. The branch join stands either way.
- `library_signed_out` *boolean*, **required** — Whether the library ended the card's session too. False when it could not be reached, or the card was not dropped here.

## ReplacementRule

Character replacement rule (`ReplacementCharacters` in classic Settings.json).

- `find` *string*, **required** — Substring to search for in each path segment / tag value.
- `replace` *string*, **required** — Replacement text (may be empty to strip `find`).

## ReportEventField

One field recorded with an event.

- `name` *string*, **required** — The field's name.
- `value` *string*, **required** — Its value, redacted before the report was sent.

## ReportEventLevel

Severity of one event in a report.

Type: one of `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`

## ReportEventView

One logged event in a stored report.

- `ts_unix_ms` *integer (int64)*, **required** — When it happened, Unix milliseconds, or `0` when the report had none. At least 0.
- `level` *[`ReportEventLevel`](#reporteventlevel)*, **required** — Its severity.
- `target` *string*, **required** — Where it was logged (the tracing target).
- `message` *string*, **required** — The message, redacted before the report was sent.
- `fields` *array of [`ReportEventField`](#reporteventfield)*, **required** — Fields recorded with it.

## ReportFixView

The fix recorded for the report's tracking issue.

- `pr_number` *integer (int64) or null* — The pull request that fixed it, when known. At least 0.
- `pr_url` *string or null* — That pull request's page (`https://`), when known.
- `fixed_in` *string or null* — The first release with the fix, when one has shipped.

## ReportIssueState

Whether an issue is open.

Type: one of `open`, `closed`

## ReportIssueView

A GitHub issue tracking the report.

- `number` *integer (int64)*, **required** — The issue number. At least 0.
- `title` *string*, **required** — Its title.
- `state` *[`ReportIssueState`](#reportissuestate)*, **required** — Whether it is open.
- `url` *string*, **required** — Its page (always `https://`).

## RequestedArtifact

The archive an owner library deploys to this install.

- `url` *string*, **required** — Where the owner fetches it from.
- `archive_sha256` *string*, **required** — SHA-256 of the archive.
- `payload_root_sha256` *string*, **required** — Payload-root digest of the tree it unpacks to.

## RequestStatus

Wishlist row status (`open` while wishlisted; `cancelled` after un-wishlist).

Type: one of `open`, `cancelled`

## ReviewItemJson

One stored book to review.

- `upload_id` *string*, **required** — The upload.
- `member_no` *integer (int32)*, **required** — The file's number in it. At least 0.
- `path` *string*, **required** — The file's path in the upload.
- `edition_id` *string or null* — The local edition it was stored under.
- `user_id` *integer (int64)*, **required** — The patron who uploaded it.

## ReviewPage

One page of books to review.

- `items` *array of [`ReviewItemJson`](#reviewitemjson)*, **required** — The books, oldest upload first.
- `next_cursor` *string or null* — Pass as `cursor` for the next page; absent when this page is empty.

## RevisionConflict

Body of a `409 Conflict` from a compare-and-swap write whose
`expected_revision` lost to a newer revision.

- `error` *string*, **required** — Always `revision_conflict`.
- `current_revision` *integer (int64)*, **required** — The revision to read and retry against.

## RevokeBody

`POST /api/config/library-consents/revoke` body.

- `library_id` *string*, **required** — Library id.
- `plugin_key` *string*, **required** — Canonical plugin key.

## RevokeForm

Form body for `POST /oidc/revoke` (RFC 7009-style refresh-token revoke).

- `token` *string*, **required** — Token to revoke; treated as a refresh token (hashed before lookup).
- `token_type_hint` *string or null* — Optional hint ignored; this server only stores refresh tokens.

## RouteJson

One route: a channel, or the built-in "In app" target, and the least
severity it takes.

- `destination_id` *string*, **required** — The channel's id, or `in-app` for the built-in target that puts the notification in every active Owner's and Administrator's inbox (the bell in the web app, and browser or desktop notifications where they turned them on). `in-app` runs no plugin and is listed first.
- `min_severity` *string*, **required** — `info`, `warning` or `critical`: notifications less urgent than this skip the channel.

## RoutesBody

Body of `PUT /api/notifications/routes/{type}`.

- `routes` *array of [`RouteJson`](#routejson)*, **required** — The type's routes, replacing the old ones; at most 32. Empty sends the type nowhere. At most 32 items.

## RouteTable

Every routing rule, by type.

- `routes` *map of string to array of [`RouteJson`](#routejson)*, **required** — Type id to its routes.

## RuleEffect

Whether a [`ScopeRule`] lets matching books in or keeps them out.

Type: one of `include`, `exclude`

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

## ScopeField

The book field a [`ScopeRule`] matches one value of.

Type: one of `genre`, `series`, `author`, `narrator`, `tag`, `collection`

## ScopeRule

One rule of a scope: a field, one value of it, and its effect.

- `field` *[`ScopeField`](#scopefield)*, **required** — The field the rule matches.
- `value` *string*, **required** — The value, compared without case and surrounding space (1 to 256 bytes).
- `effect` *[`RuleEffect`](#ruleeffect)*, **required** — Include or exclude.

## SearchBook

An item that matched a search.

- `libraryItem` *[`LibraryItem`](#libraryitem)*, **required** — The item.

## SearchResults

`GET /api/libraries/{id}/search`.

- `book` *array of [`SearchBook`](#searchbook)*, **required** — Matching items.
- `podcast` *array of object*, **required** — Always empty.
- `narrators` *array of [`Narrator`](#narrator)*, **required** — Matching narrators.
- `tags` *array of object*, **required** — Always empty.
- `genres` *array of object*, **required** — Always empty.
- `series` *array of [`SearchSeries`](#searchseries)*, **required** — Matching series.
- `authors` *array of [`Author`](#author)*, **required** — Matching authors.

## SearchSeries

A series that matched a search.

- `series` *[`Series`](#series)*, **required** — The series.
- `books` *array of [`LibraryItem`](#libraryitem)*, **required** — Its items.

## Series

A series.

- `id` *string*, **required** — The series id.
- `name` *string*, **required** — Its name.
- `nameIgnorePrefix` *string*, **required** — The name for sorting.
- `description` *string or null* — Its description: none.
- `addedAt` *integer (int64)*, **required** — When the library first saw it, in epoch milliseconds.
- `updatedAt` *integer (int64)*, **required** — When it last changed, in epoch milliseconds.
- `libraryId` *string*, **required** — The library it is listed in.
- `books` *array of [`LibraryItem`](#libraryitem) or null* — Its items in series order, on lists.
- `progress` *[`SeriesProgress`](#seriesprogress) or null* — The patron's progress through it, when asked for.

## SeriesPage

A page of series.

- `results` *array of [`Series`](#series)*, **required** — The page's series.
- `total` *integer (int64)*, **required** — Series in all. At least 0.
- `limit` *integer (int64)*, **required** — Series per page. At least 0.
- `page` *integer (int64)*, **required** — The page number, from 0. At least 0.
- `sortBy` *string or null* — The sort key, echoed.
- `sortDesc` *boolean*, **required** — Whether the order is reversed.
- `filterBy` *string or null* — The filter, echoed.
- `minified` *boolean*, **required** — Whether items are minified.
- `include` *string*, **required** — The include list, echoed.

## SeriesProgress

The patron's progress through a series.

- `libraryItemIds` *array of string*, **required** — The series' items, by id.
- `libraryItemIdsFinished` *array of string*, **required** — Those the patron finished.
- `isFinished` *boolean*, **required** — Whether the patron finished every one.

## SeriesRef

A series an item belongs to, with the item's place in it.

- `id` *string*, **required** — The series id.
- `name` *string*, **required** — The series name.
- `sequence` *string or null* — The item's place in the series as the library writes it (`1`, `2.5`), if known.

## ServerSettings

The server settings apps read.

- `id` *string*, **required** — Always `server-settings`.
- `version` *string*, **required** — [`crate::COMPATIBLE_SERVER_VERSION`]: apps turn features on by it.
- `buildNumber` *integer (int32)*, **required** — Always 1. At least 0.
- `language` *string*, **required** — The interface language.
- `dateFormat` *string*, **required** — How apps format dates.
- `timeFormat` *string*, **required** — How apps format times.
- `sortingIgnorePrefix` *boolean*, **required** — Whether sorting skips leading articles: no.
- `sortingPrefixes` *array of string*, **required** — The articles sorting would skip.
- `chromecastEnabled` *boolean*, **required** — Whether apps offer casting: no.
- `homeBookshelfView` *integer (int32)*, **required** — The home shelf style (1: standard). At least 0.
- `bookshelfView` *integer (int32)*, **required** — The library shelf style (1: standard). At least 0.
- `authActiveAuthMethods` *array of string*, **required** — [`AUTH_METHODS`].
- `authOpenIDButtonText` *string*, **required** — The browser sign-in button's label.
- `authOpenIDAutoLaunch` *boolean*, **required** — Never true.
- `authLoginCustomMessage` *string or null* — The sign-in screen's note.
- `timeZone` *string*, **required** — The host's time zone name, or UTC.

## SessionDevice

The device a playback session was opened from.

- `deviceId` *string*, **required** — The device's id: the card's session id.
- `deviceName` *string or null* — The device name the app reported, if any.
- `clientName` *string or null* — The app's name, if reported.

## SessionJson

`POST /api/playback/sessions` answer.

- `session_id` *string*, **required** — The session's id, for its sync, close and track routes.
- `edition_id` *string*, **required** — The edition being played.
- `work_id` *string or null* — Its work.
- `position_ms` *integer (int64)*, **required** — Where to start, in milliseconds: the caller's stored position, or 0 for a first listen or an edition they had finished. At least 0.
- `duration_ms` *integer (int64) or null* — Runtime in milliseconds when known (see the manifest). At least 0.
- `tracks` *array of [`TrackJson`](#trackjson)*, **required** — The tracks in play order, each with the path to stream it from.
- `idle_secs` *integer (int64)*, **required** — Seconds without a report after which the session stops streaming and the player opens a new one.

## SessionPage

`GET /api/me/listening-sessions`.

- `total` *integer (int64)*, **required** — Sessions in all. At least 0.
- `numPages` *integer (int64)*, **required** — Pages in all. At least 0.
- `page` *integer (int64)*, **required** — The page number, from 0. At least 0.
- `itemsPerPage` *integer (int64)*, **required** — Sessions per page. At least 0.
- `sessions` *array of [`PlaybackSession`](#playbacksession)*, **required** — The page's sessions, newest first.

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

## SettingCell

One cell of a table setting row: the scalar kinds only.

Type: boolean or [`SettingNumber`](#settingnumber) or string

## SettingKind

Value type of a declared plugin setting.

Type: one of `string`, `bool`, `integer`, `enum`, `secret`, `table`

## SettingNumber

A finite JSON number; an integer for integer settings.

Type: number

## SettingSpec

One setting a plugin declares in its `plugin.toml` `[[settings]]`:
its key, value type, default and help text.

- `key` *string*, **required** — Setting key (`[a-z][a-z0-9_]*`, at most 64 bytes).
- `label` *string or null* — Operator-facing label; omitted when the key is shown.
- `type` *[`SettingKind`](#settingkind)*, **required** — Value type: `string`, `bool`, `integer`, `enum`, `secret` or `table`.
- `values` *array of [`SettingValueSpec`](#settingvaluespec) or null* — Allowed values of an `enum` setting; omitted for other kinds.
- `default` *any JSON* — Default applied when the instance document has no value; omitted when there is none. A string, boolean or integer, or for a `table` an array of row objects, as the setting's `type` says.
- `required` *boolean or null* — True when the instance cannot run without a value; omitted when false.
- `description` *string or null* — Help text shown beside the setting; omitted when there is none.
- `columns` *array of [`SettingSpec`](#settingspec) or null* — Columns of a `table` setting, in display order: each a scalar entry naming one field of every row. Omitted for other kinds.

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

One setting value: a scalar, or the rows of a table setting.

Type: boolean or [`SettingNumber`](#settingnumber) or string or array of map of string to [`SettingCell`](#settingcell)

## SettingValueSpec

One allowed value of an `enum` setting: the stored id and its label.

- `id` *string*, **required** — Stored value.
- `label` *string or null* — Operator-facing label; omitted when the id is shown.

## SharedBranchesResponse

`GET /api/config/shared-branches` body.

- `node_id` *string*, **required** — Id of the node that answered.
- `nodes` *array of [`NodeView`](#nodeview)*, **required** — This node and the shared branches the caller may see.

## SharedBranchJoined

JSON body once this install joined a library as a shared branch.

- `connection` *string*, **required** — Always `shared_branch`.
- `library_id` *string*, **required** — The library id.
- `node_id` *string*, **required** — The node id the library minted for this install.
- `library_name` *string*, **required** — The library's name.
- `database` *string*, **required** — The database plugin this install now runs (`postgres` or `d1`).
- `master_key_wrapped` *boolean*, **required** — Whether the shared master key was wrapped with this install's password.
- `restarting` *boolean*, **required** — Always `true`: the daemon restarts as a shared node right after answering.

## SharedNodeRemoved

`DELETE /api/config/nodes/{node_id}` body.

- `node_id` *string*, **required** — Id of the node that was removed, trimmed.
- `removed` *boolean*, **required** — Always true: a refused removal is an error status instead.

## SharingBody

`POST /api/config/join/sharing` body: one library and the settings to
change for it.

- `library_id` *string*, **required** — The library (parent library id).
- `share_books` *boolean*, **required** — "Share this branch's books with *library*".
- `include_branches` *boolean*, **required** — "Include books from my own branches".
- `list_members_books` *boolean or null* — "List my members' books to *library*"; absent or `null` keeps the stored setting.
- `work_for_members` *boolean or null* — "Download for other members of *library*": take the library's work for its other members too, not only this install owner's. Absent or `null` keeps the stored setting.
- `keep_member_copies` *boolean or null* — "Keep copies of other members' books": also write a book acquired here for another member to this install's own destinations. Absent or `null` keeps the stored setting.
- `keep_copies` *boolean or null* — "Keep copies on this computer": take the library's copies of this install's person's books into its own storage. Absent or `null` keeps the stored setting.

## SharingChanged

JSON body after `POST /api/config/join/sharing`.

- `library` *[`LibraryLinkView`](#librarylinkview)*, **required** — The link with its settings as stored now.

## SharingSettings

JSON body for `GET /api/config/join/sharing`.

- `libraries` *array of [`LibraryLinkView`](#librarylinkview)*, **required** — Every library this install is a branch of, with its sharing and work settings.

## Shelf

A personalized shelf on the home screen.

- `id` *string*, **required** — The shelf id, such as `continue-listening`.
- `label` *string*, **required** — The shelf's English label.
- `labelStringKey` *string*, **required** — The key apps translate the label by.
- `type` *string*, **required** — What the shelf holds: `book`, `series` or `authors`.
- `entities` *[`ShelfEntities`](#shelfentities)*, **required** — The shelf's entries.
- `total` *integer (int64)*, **required** — Entries in all. At least 0.

## ShelfEntities

A shelf's entries.

Type: array of [`LibraryItem`](#libraryitem) or array of [`Series`](#series) or array of [`Author`](#author)

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

## SplitJson

The result of a split.

- `work_id` *string*, **required** — The new work the expressions moved to.
- `split_from` *string*, **required** — The work that was split.

## StartBody

`POST /api/config/join/start` body.

- `address` *string* — The library's address. Left out with `from_card`.
- `from_card` *string or null* — The library id of a library this install holds an active library card for: the branch replaces the card. The library's address and pinned certificate come from the card, so `address` and `fingerprint` are left out, and the branch name defaults to the card's name. Not with `shared_branch`.
- `fingerprint` *string or null* — The fingerprint the operator confirmed (self-signed libraries).
- `branch_name` *string or null* — The name to join as (default: this machine's name).
- `shared_branch` *boolean* — Offer to join as a shared branch (a shared node that shares the library's database): only an empty install may, and its data is replaced.

## Status

`GET /status`: what an app asks before it signs in.

- `app` *string*, **required** — Always [`STATUS_APP`].
- `serverVersion` *string*, **required** — The server release whose API this one answers like ([`crate::COMPATIBLE_SERVER_VERSION`]).
- `isInit` *boolean*, **required** — Always true: the library is set up before an app can sign in.
- `language` *string*, **required** — The interface language apps show server text in.
- `authMethods` *array of string*, **required** — [`AUTH_METHODS`].
- `authFormData` *[`AuthFormData`](#authformdata)*, **required** — What the sign-in screen shows.

## StatusResponse

Library and daemon snapshot for `GET /api/status`.

- `accounts` *integer*, **required** — Count of configured store accounts. At least 0.
- `books` *integer*, **required** — Held works in the library (the works page index). At least 0.
- `acquired` *integer (int64)*, **required** — Held works whose furthest acquire state is `acquired`.
- `pending` *integer (int64)*, **required** — Held works still needing acquire (`not_acquired`).
- `error` *integer (int64)*, **required** — Held works stuck in `error` after a failed acquire.
- `in_progress` *integer (int64)*, **required** — Held works currently `queued` or `downloading`.
- `listen` *string*, **required** — Comma-joined `daemon.listen` addresses from live config.
- `storage_backend` *string*, **required** — Comma-joined enabled destination plugin ids, or `none`.
- `auth_enabled` *boolean*, **required** — Whether the live auth middleware requires operator credentials.
- `events` *[`EventQueueStatus`](#eventqueuestatus)*, **required** — Durable domain-event delivery-queue counters.

## StepUp

A fresh second factor for a shared branch consent. `Debug` prints no code.

Type: object

## StorageFeedState

The state of a destination's change feed, as the API reports it.

Type: one of `off`, `healthy`, `gap`, `failing`

## StorageSyncView

How a destination notices changes made outside Bookclerk: its change
feed's state and its listings (storage destinations ADR, decision 5).

- `feed_state` *[`StorageFeedState`](#storagefeedstate)*, **required** — The change feed's state: `off` (no feed; listings alone), `healthy`, `gap` (the feed may have missed changes; a listing runs at once) or `failing` (reading the feed failed; listings cover it).
- `feed_error` *string or null* — The last feed failure.
- `feed_checked_at` *string or null* — RFC 3339 time the feed was last read.
- `listed_at` *string or null* — RFC 3339 time the last listing finished.
- `next_listing_at` *string or null* — RFC 3339 time the next scheduled listing is due.
- `listing_in_progress` *boolean*, **required** — True while a listing is in progress (it continues on the next pass).
- `listing_error` *string or null* — Why the last listing failed or its removals were held back.
- `objects` *integer (int64)*, **required** — Files the destination held after the last listing or feed page. At least 0.

## StoredReportView

The stored report as the collector keeps it.

- `version` *string*, **required** — The Bookclerk version that sent it.
- `trigger` *string*, **required** — Why it was sent.
- `os` *string*, **required** — The sender's operating system.
- `arch` *string*, **required** — The sender's CPU architecture.
- `distro` *string*, **required** — The sender's Linux distribution, or empty.
- `events` *array of [`ReportEventView`](#reporteventview)*, **required** — Its events, oldest first.

## StoreEdition

One storefront edition of a work (for multi-store purchase links).

- `source` *string*, **required** — Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).
- `product_id` *string*, **required** — Storefront-native product id (ASIN, ISBN, UUID, …).

## SyncBody

`POST /player/api/session/{id}/sync` and `.../close`: where the patron
is.

- `currentTime` *number (double) or null* — The position, in seconds.
- `timeListened` *number (double) or null* — Seconds listened since the last sync.
- `duration` *number (double) or null* — The runtime the app measured, in seconds.

## SyncJson

`POST /api/playback/sessions/{id}/sync` and `.../close` answer.

- `progress` *[`ProgressJson`](#progressjson)*, **required** — The caller's progress after the report.
- `finished_now` *boolean*, **required** — True when this report is the one that reached the edition's end: the edition and its work are now finished for the caller.

## SyncResult

One entry's outcome in a batch sync.

- `id` *string*, **required** — The session or item id the entry named.
- `success` *boolean*, **required** — Whether it was applied.
- `error` *string or null* — Why not, when it was not.

## SyncResults

`POST /api/session/local-all`.

- `results` *array of [`SyncResult`](#syncresult)*, **required** — One outcome per entry, in the order sent.

## TcpGrantView

Operator-facing TCP grant (`host` + `ports`).

- `host` *string*, **required** — Hostname or `*.` wildcard.
- `ports` *array of integer (int32)*, **required** — Destination ports.

## TestSent

The answer to a test send.

- `notification_id` *string*, **required** — The stored test notification.
- `delivery_id` *string*, **required** — Its delivery; follow it in the delivery log.

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
- `incomplete` *array of boolean*, **required** — Per query, in order: `true` when its `null` is a failure or a query that ran past the batch deadline, not an answer, so the client should not keep it and may ask again later.

## TitleMetaQuery

Query for public title metadata (Audnexus / Audible catalog).

- `title` *string* — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `narrators` *string or null* — Comma-separated narrator names when known.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `region` *string* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `store_editions` *array of [`StoreEdition`](#storeedition)* — Store editions that listed this title (the search hit's or card's `store_editions`), at most 16. A sparse record is filled from each of these stores that declared `detailByProductId`, by the product id it gave the title, before the stores that declared `detailByIsbn` are asked by ISBN.

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
- `resolved_edition_id` *string or null* — Edition (`editions.id`) the library holds once the wishlist item is fulfilled.
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

## TitleSampleJson

`POST /api/discover/title-sample` answer.

- `sample_path` *string or null* — Same-origin path of a storefront's sample of the title, to play with `GET`; absent when no store that lists it offers one.

## TokenError

A token-endpoint refusal as RFC 6749 §5.2 writes it: JSON with an
`error` code and a human-readable `error_description`, sent with
`Cache-Control: no-store`.

- `error` *[`TokenErrorCode`](#tokenerrorcode)*, **required** — What went wrong, as a client library branches on it.
- `error_description` *string or null* — A sentence for the person debugging the client; never a secret or a token.

## TokenErrorCode

The `error` code of a token-endpoint refusal (RFC 6749 §5.2), which also
fixes its status.

Type: one of `invalid_request`, `invalid_client`, `invalid_grant`, `unsupported_grant_type`, `server_error`

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

## TrackJson

One audio file of an edition, as a player sees it.

- `index` *integer*, **required** — The track's place in play order, from 0. At least 0.
- `path` *string or null* — Path to stream it from, on this origin; only in a session's answer.
- `content_type` *string*, **required** — Media type the stream states, such as `audio/mp4`.
- `bitrate_kbps` *integer (int32) or null* — Average bitrate in kilobits per second, once probed. At least 0.
- `lossless` *boolean*, **required** — True for a lossless encoding.

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

- `status` *string*, **required** — Stable outcome: `received`, `partial` (a part that is not the file's last was taken), `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
- `reason` *string or null* — Why, for `refused`, and `resume` for an `unavailable` part that does not continue the open file at [`Self::offset`].
- `offset` *integer (int64) or null* — For `partial`, and an `unavailable` part with reason `resume`: the offset of the next byte of the file the owner needs. At least 0.
- `job_completed` *boolean* — For `committed`: the job is finished.

## UploadCommitRequest

Body of `POST <UPLOADS_PATH><upload_id>/commit`.

- `finish` *boolean* — Complete the job with this commit (the job's closing upload).
- `detail` *string or null* — Outcome text recorded as the job's progress when `finish` is set.

## UploadJson

One upload.

- `id` *string*, **required** — The upload id (32 hex digits).
- `state` *string*, **required** — `open` (taking bytes), `committed` (queued), `running`, `done`, `failed` or `cancelled`.
- `source` *string*, **required** — `upload`, or `intake` for a batch a drop folder made.
- `fileCount` *integer (int64)*, **required** — Files the manifest named. At least 0.
- `declaredBytes` *integer (int64)*, **required** — Bytes the manifest declared. At least 0.
- `jobId` *string or null* — The ingest job, once committed.
- `detail` *string or null* — Why the upload failed.
- `createdAt` *string*, **required** — When it was created (RFC 3339).
- `updatedAt` *string*, **required** — Its last activity (RFC 3339).
- `finishedAt` *string or null* — When it finished (RFC 3339).
- `files` *array of [`UploadMemberJson`](#uploadmemberjson) or null* — Its files; left out of lists.

## UploadList

The caller's uploads, newest first.

- `uploads` *array of [`UploadJson`](#uploadjson)*, **required** — At most 50 uploads, without their files.

## UploadManifest

`POST /api/uploads` body: the files of one batch.

- `files` *array of [`ManifestFile`](#manifestfile)*, **required** — Every file of the batch, in the order their numbers follow.

## UploadMemberJson

One file of an upload.

- `memberNo` *integer (int32)*, **required** — Its number in the upload; manifest files are numbered from 0 in order, an archive's entries after every earlier file. At least 0.
- `parentNo` *integer (int32) or null* — The archive it was expanded from. At least 0.
- `path` *string*, **required** — Its path in the upload (for a refused archive entry, its cleaned name).
- `size` *integer (int64)*, **required** — Its size in bytes. At least 0.
- `received` *integer (int64)*, **required** — Bytes received so far; resume a `PATCH` here. At least 0.
- `outcome` *string*, **required** — What became of it: `pending`, `expanded` (an archive), `matched`, `needs_review`, `duplicate`, `refused` or `failed`.
- `reason` *string or null* — Why it was refused or failed, as a stable code.
- `editionId` *string or null* — The edition it was stored under.

## User

A patron as apps see them.

- `id` *string*, **required** — The patron's portal identity id, as a string.
- `username` *string*, **required** — The name the patron signs in with.
- `email` *string or null* — The patron's email address, if the library has one.
- `type` *string*, **required** — Always `user`: no app is given administrator powers.
- `token` *string*, **required** — The access token, for apps that read it from here (older apps send this one as their bearer). Empty where the answer is not a sign-in.
- `accessToken` *string or null* — The card's access token, on sign-in and refresh answers only.
- `refreshToken` *string or null* — The card's refresh token, on sign-in and refresh answers only.
- `isOldToken` *boolean*, **required** — Always false: tokens are never the old non-expiring kind.
- `mediaProgress` *array of [`MediaProgress`](#mediaprogress)*, **required** — Progress on every item the patron has started.
- `seriesHideFromContinueListening` *array of string*, **required** — Series hidden from continue listening: none.
- `bookmarks` *array of [`Bookmark`](#bookmark)*, **required** — Every bookmark the patron has made.
- `isActive` *boolean*, **required** — Always true: a disabled patron cannot sign in.
- `isLocked` *boolean*, **required** — Always false.
- `lastSeen` *integer (int64) or null* — When the patron was last seen, in epoch milliseconds.
- `createdAt` *integer (int64)*, **required** — When the patron joined, in epoch milliseconds.
- `permissions` *[`Permissions`](#permissions)*, **required** — What the patron may do.
- `librariesAccessible` *array of string*, **required** — Libraries the patron may open, by id.
- `itemTagsSelected` *array of string*, **required** — Tags that limit what the patron sees: none (scope is applied by the host, not by tags).
- `hasOpenIDLink` *boolean*, **required** — Whether the patron signed in through the browser at least once.

## UserAdminControlsJson

An Administrator's switches as the switch routes report them.

- `user_id` *integer (int64)*, **required** — The user (`users.id`).
- `defaults` *[`AdminControls`](#admincontrols)*, **required** — The library's default for every Administrator.
- `overrides` *[`AdminControlOverrides`](#admincontroloverrides)*, **required** — The switches set for this Administrator alone; the others follow the default.
- `effective` *[`AdminControls`](#admincontrols)*, **required** — What applies to the user now: everything for an Owner, nothing for a disabled user or a Member, and the default with the overrides applied for an Administrator.
- `applies` *boolean*, **required** — Whether the switches limit the user's role: true only for an Administrator.

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

## UserControlsJson

A patron's member controls as the control routes report them.

- `user_id` *integer (int64)*, **required** — The user (`users.id`).
- `defaults` *[`MemberControls`](#membercontrols)*, **required** — The library's default, which Members follow.
- `overrides` *[`ControlOverrides`](#controloverrides)*, **required** — The controls set for this patron alone; the others follow the default for a Member and are allowed for an Administrator.
- `effective` *[`MemberControls`](#membercontrols)*, **required** — What applies to the user now: nothing for a disabled user, everything for an Owner, the overrides alone for an Administrator, and the default with the overrides applied for a Member.
- `applies` *boolean*, **required** — Whether controls can limit the user's role: false for an Owner, who may always do everything.
- `follows_default` *boolean*, **required** — Whether the user follows the library's default: true for a Member, false for an Administrator, whom only controls set for them alone limit (and only an Owner or the operator sets those).

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
- `controls` *[`MemberControls`](#membercontrols)*, **required** — What the user may do here (the member controls that apply to them; see `GET /api/users/{id}/controls` for the default and overrides).
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

First-party user role (`owner` | `administrator` | `member`).

Type: one of `owner`, `administrator`, `member`

## UserScopeJson

A patron's scope as the scope routes report it.

- `user_id` *integer (int64)*, **required** — The user (`users.id`).
- `scope` *[`PatronScope`](#patronscope)*, **required** — The scope that applies to them: their own, or the library's default.
- `own` *boolean*, **required** — True when an Administrator gave them this scope; false when they follow the library's default.
- `applies` *boolean*, **required** — Whether the scope applies to the user's role: false for an Owner or Administrator, who always see the whole library.

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

## WindowBooks

JSON body for `GET /api/config/join/window`.

- `libraries` *array of [`LibraryLinkView`](#librarylinkview)*, **required** — The libraries this install is an active branch of.
- `cards` *array of [`ClerkLinkView`](#clerklinkview)*, **required** — The libraries this install holds an active library card for.
- `books` *array of [`WindowEntryView`](#windowentryview)*, **required** — Their books, every one they list for a member, one entry per work, by title; each edition says whether it is this install's person's.

## WindowEditionView

One edition in [`WindowBooks`], and the library that holds it.

- `library_id` *string*, **required** — The library (parent library id) that holds it.
- `owner_origin` *string*, **required** — The library's origin.
- `store` *string*, **required** — The store's plain id.
- `marketplace` *string*, **required** — Lower-cased store marketplace; empty when the store has none.
- `product_id` *string*, **required** — The store's product id.
- `formats` *array of string*, **required** — Formats that library has (`audio`, `pdf`); empty when unknown.
- `title` *string*, **required** — The edition's title.
- `subtitle` *string or null* — Subtitle, when known.
- `authors` *string or null* — Authors as one display string, when known.
- `narrators` *string or null* — Narrators as one display string, when known.
- `series` *string or null* — Series name, when known.
- `series_index` *string or null* — Position in the series, when known.
- `length_minutes` *integer (int64) or null* — Length in minutes, when known.
- `yours` *boolean*, **required** — True when this install's person holds it there through a store account of their own; false for a book only other people there hold.
- `request` *[`WindowRequestView`](#windowrequestview) or null* — How to ask the library for a copy of this edition (`POST /api/config/clerk/request`): set when this install holds a card for the library and the library holds no copy, while one of its branches lists it (`from` is `branch`) or one of its own store accounts holds it (`from` is `library`). Absent otherwise.

## WindowEntryView

One work in [`WindowBooks`]: every edition its libraries hold for the person.

- `key` *string*, **required** — A key that stays the same while the entry's editions do.
- `work_id` *string or null* — The work of this install's own library the entry was placed on, by identifier; absent when none was.
- `editions` *array of [`WindowEditionView`](#windoweditionview)*, **required** — The editions, in library then store order; the first names the entry.

## WindowRequestView

How a clerk asks a library for a copy of one edition in [`WindowBooks`].

- `work_id` *string*, **required** — The library's own work id to ask for.
- `from` *[`ClerkRequestFrom`](#clerkrequestfrom)*, **required** — Where the copy would come from.

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

## WorkDetailJson

`GET /api/library/works/{id}` answer.

- `id` *string*, **required** — Work id.
- `kind` *[`WorkKindJson`](#workkindjson)*, **required** — Book or podcast.
- `title` *string*, **required** — Display title.
- `description` *string or null* — Blurb or synopsis (may contain HTML).
- `authors` *array of string*, **required** — Authors (hosts for a podcast), in billing order.
- `series` *string or null* — First series the work belongs to.
- `series_position` *string or null* — Position in `series`.
- `editions` *array of [`EditionJson`](#editionjson)*, **required** — Editions; a show's episodes' editions included.
- `state` *[`WorkStateJson`](#workstatejson) or null* — The caller's own state; absent for the operator or when the caller recorded nothing.
- `can_edit_state` *boolean*, **required** — Whether the caller can keep their own state for the work (`PUT /api/library/works/{id}/state`): false when the session has no library user, such as the operator token.

## WorkKindJson

Book or podcast (`works.kind`).

Type: one of `book`, `podcast`

## WorksPage

A page of `GET /api/library/works`.

- `works` *array of [`WorkSummaryJson`](#worksummaryjson)*, **required** — The works, in the requested order.
- `next_cursor` *string or null* — Pass as `cursor` for the next page; absent on the last page.

## WorkSplitBody

`POST /api/library/works/{id}/split` body: the expressions to move.

- `expression_ids` *array of string*, **required** — The expressions to move to the new work (1 to 12 distinct ids, each at most 64 bytes), from the work detail's editions (`expression_id`).

## WorkStateEdit

`PUT /api/library/works/{id}/state` body: the fields of the caller's
own state to change. A field left out keeps its stored value; `null`
clears a rating or the tags.

- `is_finished` *boolean or null* — Whether the caller finished the work.
- `rating_overall` *number (double) or null* — Overall rating, 0 to 5; `null` clears it.
- `rating_performance` *number (double) or null* — Narration rating, 0 to 5; `null` clears it.
- `rating_story` *number (double) or null* — Story rating, 0 to 5; `null` clears it.
- `tags` *string or null* — Tags separated by whitespace (at most 32, each at most 64 bytes, 1024 bytes in all); `null` or an empty string clears them.

## WorkStateJson

The caller's own state for a work.

- `is_finished` *boolean*, **required** — Whether the caller finished the work.
- `rating_overall` *number (double) or null* — Overall rating, 0 to 5.
- `rating_performance` *number (double) or null* — Narration rating, 0 to 5.
- `rating_story` *number (double) or null* — Story rating, 0 to 5.
- `tags` *string or null* — The caller's tags, as one serialized string.

## WorkSummaryJson

One work in a page of `GET /api/library/works`.

- `id` *string*, **required** — Work id.
- `kind` *[`WorkKindJson`](#workkindjson)*, **required** — Book or podcast.
- `title` *string*, **required** — Display title.
- `authors` *string or null* — Authors (hosts for a podcast), joined for display.
- `series` *string or null* — First series the work belongs to.
- `series_position` *string or null* — Position in `series`, as the store gave it.
- `cover_edition_id` *string or null* — The edition whose cover the list shows.
- `cover_path` *string or null* — Same-origin path of that cover, when it has one.
- `media_kinds` *array of [`MediaKindJson`](#mediakindjson)*, **required** — Carriers of the work's editions.
- `acquire_status` *[`AcquireStatus`](#acquirestatus)*, **required** — The furthest acquire state any edition reached.
- `edition_count` *integer (int64)*, **required** — How many editions the work has. At least 0.
- `holding_count` *integer (int64)*, **required** — How many holdings the work's listings have. At least 0.
- `added_at` *string*, **required** — RFC 3339 time of the newest holding.
- `length_minutes` *integer (int64) or null* — Runtime in minutes of the longest held edition, when known. At least 0.
- `purchased_at` *string or null* — RFC 3339 time of the newest purchase a store reported.

## WorkTag

One library tag on a work.

- `value` *string*, **required** — The tag as written.
- `source` *[`WorkTagSource`](#worktagsource)*, **required** — Where it came from.

## WorkTagsBody

`PUT /api/library/works/{id}/tags` body: the tags an Administrator sets.

- `tags` *array of string*, **required** — The tags, replacing the ones set in Bookclerk (sidecar tags stay). Blanks and repeats, ignoring case, are dropped; at most 32 tags of at most 256 bytes each. At most 32 items.

## WorkTagsJson

A work's library tags.

- `work_id` *string*, **required** — The work.
- `tags` *array of [`WorkTag`](#worktag)*, **required** — Every tag, by key, with where it came from.

## WorkTagSource

Where a library tag came from (`work_tags.source`).

Type: one of `admin`, `sidecar`
