**bookclerk_node_client > clerk**

# Module: clerk

## Contents

**Structs**

- [`AppliedClerk`](#appliedclerk) - What [`apply_clerk_bootstrap`](clerk.md#bookclerk_node_clientclerkapply_clerk_bootstrap) wrote.
- [`ClerkJoined`](#clerkjoined) - A redeemed clerk sign-in. `Debug` prints no secret.
- [`PendingClerkJoin`](#pendingclerkjoin) - A clerk join whose files are all in place but whose first start has not
- [`RevertFailed`](#revertfailed) - Why `PendingClerkJoin::revert` did not finish.

**Functions**

- [`apply_clerk_bootstrap`](#apply_clerk_bootstrap) - Writes `bootstrap` as this install's own bootstrap under `files_dir`:
- [`bootstrap_database_kind`](#bootstrap_database_kind) - The `[database].plugin` a clerk's bootstrap configures.
- [`checked_d1_api_base`](#checked_d1_api_base) - Checks the D1 API origin a library's bootstrap names, and returns it
- [`database_guest_installed`](#database_guest_installed) - True when the database guest for `kind` is installed for `config`, so
- [`install_problem`](#install_problem) - Why this install may not join as a clerk now, or `None`.
- [`interrupted_clerk_join`](#interrupted_clerk_join) - Names the copies an interrupted clerk join left beside this install's
- [`pending_clerk_join`](#pending_clerk_join) - The clerk join this install applied but has not yet started on (or
- [`redeem_clerk_code`](#redeem_clerk_code) - Redeems an approved clerk sign-in and opens the bootstrap with the

**Constants**

- [`CLERK_POSTGRES_URL_FILE`](#clerk_postgres_url_file) - File under the files directory that holds a clerk's Postgres URL, mode
- [`DATABASE_ENV_OVERRIDES`](#database_env_overrides) - Environment variables that would outrank a clerk's `[database]`

---

## bookclerk_node_client::clerk::AppliedClerk

*Struct*

What [`apply_clerk_bootstrap`](clerk.md#bookclerk_node_clientclerkapply_clerk_bootstrap) wrote.

**Fields:**
- `database: bookclerk_config::DatabasePluginKind` - The `[database].plugin` now configured.
- `wrapped: bool` - True when `master.key` is wrapped with this install's password.
- `config_file: std::path::PathBuf` - The config file written.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AppliedClerk`
- **PartialEq**
  - `fn eq(self: &Self, other: &AppliedClerk) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_client::clerk::CLERK_POSTGRES_URL_FILE

*Constant*: `&str`

File under the files directory that holds a clerk's Postgres URL, mode
`0600`. `[database.postgres].url_file` names it by its absolute path
(a relative `url_file` would be read from the daemon's working
directory).



## bookclerk_node_client::clerk::ClerkJoined

*Struct*

A redeemed clerk sign-in. `Debug` prints no secret.

**Fields:**
- `cluster_id: String` - The library's cluster id.
- `node_id: String` - The node id the library recorded (`kind = owner`).
- `bootstrap: bookclerk_node_proto::ClerkBootstrap` - The opened bootstrap.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_client::clerk::DATABASE_ENV_OVERRIDES

*Constant*: `&[&str]`

Environment variables that would outrank a clerk's `[database]`
bootstrap; a clerk join is refused while one is set.



## bookclerk_node_client::clerk::PendingClerkJoin

*Struct*

A clerk join whose files are all in place but whose first start has not
opened the library's database yet, or whose undoing was cut short. The
daemon `confirm`s it once the database opens, or
`revert`s it when it cannot, so a join to an unreachable
database does not leave the server unable to start.

**Methods:**

- `fn reverting(self: &Self) -> bool` - Whether an earlier start began putting the originals back and did not
- `fn confirm(self: Self)` - The library's database opened: drops the record, then the copies of
- `fn revert(self: Self) -> Result<(), RevertFailed>` - The library's database did not open: puts every original back,

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_node_client::clerk::RevertFailed

*Struct*

Why `PendingClerkJoin::revert` did not finish.

**Fields:**
- `started: bool` - Whether the undo had begun: when true the record is marked
- `detail: String` - What went wrong, naming the files involved.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_node_client::clerk::apply_clerk_bootstrap

*Function*

Writes `bootstrap` as this install's own bootstrap under `files_dir`:

- `database-postgres.url` (Postgres only): the URL, mode `0600`;
- `node-identity.json`: the node id the library recorded, bound to its
  cluster;
- `master.key`: the library's DEK, wrapped with `password` (this
  install's `BOOKCLERK_AUTH_PASSWORD` or `[auth].password`) when set;
- `[database]` in `config_file` (other settings kept): `postgres` with
  `url_file` naming the URL file (never the URL itself in the TOML), or
  `d1` with its ids (the D1 API token stays this node's
  environment-only bootstrap).

The files go in as one recoverable step. All the new contents are first
written and synced beside their targets (hidden `.<name>.clerk-<hex>`
files), and each existing target is copied to `<name>.pre-clerk`; a
failure up to here changes no target. The targets are then replaced
atomically one by one, in the order above. If one fails, the ones
already replaced are put back from their copies (a target that did not
exist before is removed), so the install keeps its own bootstrap; the
error names any file that could not be put back. On success the copies
stay, and `clerk-join.json` in `files_dir` records the join: the next start drops
them once the library's database opens (`PendingClerkJoin::confirm`)
and puts the originals back if it does not (`PendingClerkJoin::revert`).
The caller restarts the daemon afterwards.

A process that dies while replacing leaves the files before that point
in the list new and the rest old, with every original still beside its
file as `<name>.pre-clerk` and no record; a later apply refuses while a
copy is there, and so does the daemon ([`interrupted_clerk_join`](clerk.md#bookclerk_node_clientclerkinterrupted_clerk_join)), with
instructions to finish the join by hand (when `config_file` is already
new) or undo it. Staged `.<name>.clerk-<hex>` files left by such a crash
are unused and can be deleted.

# Errors

Returns `JoinError::Record` when a file cannot be written or the
existing `config.toml` does not parse.

```rust
fn apply_clerk_bootstrap(files_dir: &std::path::Path, config_file: &std::path::Path, bootstrap: &bookclerk_node_proto::ClerkBootstrap, password: Option<&str>) -> Result<AppliedClerk, crate::enroll::JoinError>
```



## bookclerk_node_client::clerk::bootstrap_database_kind

*Function*

The `[database].plugin` a clerk's bootstrap configures.

```rust
fn bootstrap_database_kind(database: &bookclerk_node_proto::ClerkDatabase) -> bookclerk_config::DatabasePluginKind
```



## bookclerk_node_client::clerk::checked_d1_api_base

*Function*

Checks the D1 API origin a library's bootstrap names, and returns it
normalized. Every D1 query carries the joining server's own API token
to that origin, so a clerk may only be pointed at Cloudflare's API over
HTTPS. The library checks its own origin the same way before it offers
a clerk join, so a library on another D1 endpoint cannot take clerks.

# Errors

Returns `JoinError::Record` when `api_base` is not an `https` URL on
`api.cloudflare.com` (no other port, user name or password).

```rust
fn checked_d1_api_base(api_base: &str) -> Result<String, crate::enroll::JoinError>
```



## bookclerk_node_client::clerk::database_guest_installed

*Function*

True when the database guest for `kind` is installed for `config`, so
the daemon can open the library's database once the bootstrap is
applied. A plugin directory that cannot be read counts as not
installed.

```rust
fn database_guest_installed(config: &bookclerk_config::Config, kind: bookclerk_config::DatabasePluginKind) -> bool
```



## bookclerk_node_client::clerk::install_problem

*Function*

Why this install may not join as a clerk now, or `None`.

# Errors

Returns the read error.

```rust
fn install_problem(store: &bookclerk_library::LibraryStore) -> Result<Option<String>, crate::enroll::JoinError>
```



## bookclerk_node_client::clerk::interrupted_clerk_join

*Function*

Names the copies an interrupted clerk join left beside this install's
bootstrap files, or `None` when there are none. The daemon refuses to
start while any is there: a process that died while replacing the files
leaves some new and some old, and starting on that mix would pair a
database with the wrong node identity or key. The message says how to
finish or undo the join by hand (see [`apply_clerk_bootstrap`](clerk.md#bookclerk_node_clientclerkapply_clerk_bootstrap)).

```rust
fn interrupted_clerk_join(files_dir: &std::path::Path, config_file: &std::path::Path) -> Option<String>
```



## bookclerk_node_client::clerk::pending_clerk_join

*Function*

The clerk join this install applied but has not yet started on (or
began undoing), if any. A record that cannot be read, or that names any
file other than this install's own bootstrap files and their copies, is
not used: copies left beside those files then stop the start (see
[`interrupted_clerk_join`](clerk.md#bookclerk_node_clientclerkinterrupted_clerk_join)).

```rust
fn pending_clerk_join(files_dir: &std::path::Path, config_file: &std::path::Path) -> Option<PendingClerkJoin>
```



## bookclerk_node_client::clerk::redeem_clerk_code

*Function*

Redeems an approved clerk sign-in and opens the bootstrap with the
node's box secret. The code and verifier go once in the request body,
and are registered for redaction (the caller unregisters them).

# Errors

Returns [`JoinError`](enroll.md#bookclerk_node_clientenrolljoinerror) when the library is unreachable or refuses (the
code was approved for a branch, the user is no longer an Owner, or its
database cannot be shared). Once the library has enrolled the node, an
answer for another cluster or type, without a bootstrap, or with one
that does not open for this node gives `JoinError::Unsaved` naming
it. `JoinError::is_retryable` tells which failures a repeat of the
same exchange can fix (the library answers it with the same node and a
freshly sealed bootstrap).

```rust
fn redeem_clerk_code(redeem: &crate::branch::BranchRedeem, keys: &bookclerk_library::control_plane::ClusterLinkKeys) -> Result<ClerkJoined, crate::enroll::JoinError>
```



