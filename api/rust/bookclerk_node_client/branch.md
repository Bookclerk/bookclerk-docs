**bookclerk_node_client > branch**

# Module: branch

## Contents

**Structs**

- [`BranchAuthorization`](#branchauthorization) - One sign-in about to start. `Debug` prints no secret.
- [`BranchRedeem`](#branchredeem) - What [`redeem_branch_code`](branch.md#bookclerk_node_clientbranchredeem_branch_code) needs besides the keys.
- [`InspectedLibrary`](#inspectedlibrary) - A library as [`inspect_library`](branch.md#bookclerk_node_clientbranchinspect_library) found it.

**Functions**

- [`announce_leaving`](#announce_leaving) - Tells the library of `link` that this install is leaving: connects with
- [`authorize_branch`](#authorize_branch) - Starts a sign-in at `library` for this install as `branch_name`, coming
- [`authorize_join`](#authorize_join) - [`authorize_branch`](branch.md#bookclerk_node_clientbranchauthorize_branch), optionally offering to join as a clerk
- [`inspect_library`](#inspect_library) - Reads what the library at `address` says about itself, under the
- [`library_origin`](#library_origin) - Normalizes what an operator typed (`owner.example:8443`,
- [`redeem_branch_code`](#redeem_branch_code) - Redeems an approved sign-in: the library runs the same redeem as for a

---

## bookclerk_node_client::branch::BranchAuthorization

*Struct*

One sign-in about to start. `Debug` prints no secret.

**Fields:**
- `authorize_url: String` - Where to send the browser.
- `state: zeroize::Zeroizing<String>` - The OAuth `state` (also in the URL).
- `verifier: zeroize::Zeroizing<String>` - The PKCE verifier; never leaves this install.
- `keys: bookclerk_library::control_plane::ClusterLinkKeys` - The node keys the branch will enroll.
- `redirect_uri: String` - The redirect URI in the URL.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerk_node_client::branch::BranchRedeem

*Struct*

What [`redeem_branch_code`](branch.md#bookclerk_node_clientbranchredeem_branch_code) needs besides the keys.

**Generic Parameters:**
- 'a

**Fields:**
- `origin: &'a str` - The library's canonical origin.
- `spki_pin: Option<&'a str>` - The pin confirmed for a self-signed library.
- `cluster_id: &'a str` - The cluster id the library reported before sign-in.
- `code: &'a str` - The code from the callback.
- `verifier: &'a str` - The PKCE verifier.
- `redirect_uri: &'a str` - The redirect URI of the authorization.
- `branch_name: &'a str` - The branch name.
- `software_version: &'a str` - This install's version.
- `enrolled_node_id: Option<&'a str>` - The node the library enrolled on an earlier attempt of this same



## bookclerk_node_client::branch::InspectedLibrary

*Struct*

A library as [`inspect_library`](branch.md#bookclerk_node_clientbranchinspect_library) found it.

**Fields:**
- `info: bookclerk_node_proto::OwnerInfo` - What the library said about itself.
- `origin: String` - Its canonical origin.
- `spki_pin: Option<String>` - The SPKI pin to trust from now on: set for a self-signed library

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &InspectedLibrary) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> InspectedLibrary`



## bookclerk_node_client::branch::announce_leaving

*Function*

Tells the library of `link` that this install is leaving: connects with
the link's keys (claiming nothing; `sampler` answers the connection's
first heartbeat), sends `node.leaving`, and waits for the answer for up
to `timeout`. Answers true when the library accepted it. Call it before
the keys are deleted.

A running client for the same link is superseded by this connection;
the library then revokes the node, so that client stops too.

```rust
fn announce_leaving(link: bookclerk_library::control_plane::ClusterLink, keys: bookclerk_library::control_plane::ClusterLinkKeys, software_version: &str, sampler: Box<dyn HeartbeatSource>, timeout: std::time::Duration) -> bool
```



## bookclerk_node_client::branch::authorize_branch

*Function*

Starts a sign-in at `library` for this install as `branch_name`, coming
back to `redirect_uri` (a loopback or https URL).

# Errors

Returns `JoinError::Link` for a redirect URI the library would refuse
or a malformed branch name.

```rust
fn authorize_branch(library: &InspectedLibrary, redirect_uri: &str, branch_name: &str) -> Result<BranchAuthorization, crate::enroll::JoinError>
```



## bookclerk_node_client::branch::authorize_join

*Function*

[`authorize_branch`](branch.md#bookclerk_node_clientbranchauthorize_branch), optionally offering to join as a clerk
(`offer_clerk`, [`CLERK_OFFER_PARAM`](../bookclerk_node_proto/clerk.md#bookclerk_node_protoclerkclerk_offer_param)): the consent page then lets an
Owner of the library choose a clerk instead of a branch. Offer it only
for an install that holds no library data of its own.

# Errors

As [`authorize_branch`](branch.md#bookclerk_node_clientbranchauthorize_branch).

```rust
fn authorize_join(library: &InspectedLibrary, redirect_uri: &str, branch_name: &str, offer_clerk: bool) -> Result<BranchAuthorization, crate::enroll::JoinError>
```



## bookclerk_node_client::branch::inspect_library

*Function*

Reads what the library at `address` says about itself, under the
transport policy (plain HTTP only on loopback).

- A `self_signed` library must present the certificate whose pin it
  reports; the pin is answered for the operator to confirm.
- Any other TLS library must present a certificate that chains to the
  public roots.
- The library must name the origin it was reached at as its own (the
  origin its node handshake signs), so the link is recorded with an
  origin that works.

# Errors

Returns [`JoinError`](enroll.md#bookclerk_node_clientenrolljoinerror) when the address is malformed, the library is
unreachable or untrusted, or its answer is malformed or names another
origin.

```rust
fn inspect_library(address: &str) -> Result<InspectedLibrary, crate::enroll::JoinError>
```



## bookclerk_node_client::branch::library_origin

*Function*

Normalizes what an operator typed (`owner.example:8443`,
`https://owner.example/`) to a canonical origin. A bare `host[:port]`
means `https`.

# Errors

Returns `JoinError::Link` when it is not an http(s) origin.

```rust
fn library_origin(address: &str) -> Result<String, crate::enroll::JoinError>
```



## bookclerk_node_client::branch::redeem_branch_code

*Function*

Redeems an approved sign-in: the library runs the same redeem as for a
link (one node, a receipt, idempotent retries), then this install seals
`keys` and records the link. The code and verifier go once in the
request body and are never logged.

# Errors

Returns [`JoinError`](enroll.md#bookclerk_node_clientenrolljoinerror) when the library is unreachable or refuses; once
the library has enrolled a node, a malformed answer or one for another
cluster gives `JoinError::Unsaved`, which names that node so it can be
revoked there, as does a link this install refuses to record. A failed
database or disk write gives `JoinError::SaveFailed` (redeeming the
same code again with the same verifier and keys gets the same node, so
that one can be retried, naming that node in
`BranchRedeem::enrolled_node_id`).

```rust
fn redeem_branch_code(store: &bookclerk_library::LibraryStore, redeem: &BranchRedeem, keys: &bookclerk_library::control_plane::ClusterLinkKeys) -> Result<bookclerk_library::control_plane::ClusterLink, crate::enroll::JoinError>
```



