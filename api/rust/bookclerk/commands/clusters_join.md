**bookclerk > commands > clusters_join**

# Module: commands::clusters_join

## Contents

**Structs**

- [`Callback`](#callback) - What the browser brought back.

**Enums**

- [`Joined`](#joined) - How `clusters join --url` ended.

**Functions**

- [`join_with_sign_in`](#join_with_sign_in) - Runs `clusters join --url`, as a branch, or offering a clerk when
- [`wait_for_callback`](#wait_for_callback) - Waits on `listener` for `GET /join/callback?code=...&state=<state>`

**Constants**

- [`CALLBACK_TIMEOUT`](#callback_timeout) - How long the CLI waits for the browser to come back.

---

## bookclerk::commands::clusters_join::CALLBACK_TIMEOUT

*Constant*: `std::time::Duration`

How long the CLI waits for the browser to come back.



## bookclerk::commands::clusters_join::Callback

*Struct*

What the browser brought back.

**Fields:**
- `code: zeroize::Zeroizing<String>` - The authorization code.
- `error: Option<String>` - The library's `error`, when it denied.
- `connection: Option<String>` - The connection type the library's user chose (`clerk`), when not a

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &Callback) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk::commands::clusters_join::Joined

*Enum*

How `clusters join --url` ended.

**Variants:**
- `Branch(bookclerk_library::control_plane::ClusterLink)` - A branch: the link is recorded and a running daemon dials it.
- `Clerk{ cluster_id: String, node_id: String, applied: bookclerk_node_client::AppliedClerk }` - A clerk: the library's bootstrap is written; `bookclerkd` starts on

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk::commands::clusters_join::join_with_sign_in

*Function*

Runs `clusters join --url`, as a branch, or offering a clerk when
`offer_clerk` (the library's Owner then chooses).

# Errors

Returns an error when the library cannot be read or refuses, the
fingerprint is not confirmed, the sign-in does not come back, a clerk
join is not possible here, or the link or bootstrap cannot be written.

```rust
fn join_with_sign_in(store: &bookclerk_library::LibraryStore, config: &bookclerk_config::Config, address: &str, branch_name: Option<&str>, fingerprint: Option<&str>, offer_clerk: bool) -> anyhow::Result<Joined>
```



## bookclerk::commands::clusters_join::wait_for_callback

*Function*

Waits on `listener` for `GET /join/callback?code=...&state=<state>`
until `deadline`. Requests for other paths, or with another `state`,
are answered and ignored.

# Errors

Returns an error when the deadline passes or the listener fails.

```rust
fn wait_for_callback(listener: &tokio::net::TcpListener, state: &str, deadline: std::time::Instant) -> anyhow::Result<Callback>
```



