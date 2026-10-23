**bookclerk_node_proto > clerk**

# Module: clerk

## Contents

**Structs**

- [`ClerkBootstrap`](#clerkbootstrap) - What a clerk receives at redeem: the `[database]` bootstrap and the DEK,

**Enums**

- [`ClerkDatabase`](#clerkdatabase) - The library's database, as a clerk needs it to connect.
- [`ConnectionType`](#connectiontype) - How a joining install connects to a library.

**Functions**

- [`clerk_bootstrap_aad`](#clerk_bootstrap_aad) - Associated data of a clerk bootstrap for `node_id` of `cluster_id`: the
- [`open_clerk_bootstrap`](#open_clerk_bootstrap) - Opens a bootstrap made by [`seal_clerk_bootstrap`](clerk.md#bookclerk_node_protoclerkseal_clerk_bootstrap) with the joining
- [`seal_clerk_bootstrap`](#seal_clerk_bootstrap) - Seals `bootstrap` to the joining node's box key `recipient`, as

**Constants**

- [`CLERK_BOOTSTRAP_AAD_DOMAIN`](#clerk_bootstrap_aad_domain) - Domain tag that starts the associated data of a clerk bootstrap.
- [`CLERK_BOOTSTRAP_VERSION`](#clerk_bootstrap_version) - Version of [`ClerkBootstrap`](clerk.md#bookclerk_node_protoclerkclerkbootstrap).
- [`CLERK_GRANTED_SCOPE`](#clerk_granted_scope) - The scope a clerk code is granted (stored with the code at consent).
- [`CLERK_OFFER_PARAM`](#clerk_offer_param) - Authorize-request parameter by which a joining install offers to join as
- [`DEK_LEN`](#dek_len) - Length of the data-encryption key.

---

## bookclerk_node_proto::clerk::CLERK_BOOTSTRAP_AAD_DOMAIN

*Constant*: `&[u8]`

Domain tag that starts the associated data of a clerk bootstrap.



## bookclerk_node_proto::clerk::CLERK_BOOTSTRAP_VERSION

*Constant*: `u32`

Version of [`ClerkBootstrap`](clerk.md#bookclerk_node_protoclerkclerkbootstrap).



## bookclerk_node_proto::clerk::CLERK_GRANTED_SCOPE

*Constant*: `&str`

The scope a clerk code is granted (stored with the code at consent).



## bookclerk_node_proto::clerk::CLERK_OFFER_PARAM

*Constant*: `&str`

Authorize-request parameter by which a joining install offers to join as
a clerk (`clerk_offer=1`): it is empty, so its own data may be replaced.
Without it the consent page offers a branch only.



## bookclerk_node_proto::clerk::ClerkBootstrap

*Struct*

What a clerk receives at redeem: the `[database]` bootstrap and the DEK,
for the node id the library recorded (`kind = owner`). `Debug` prints no
secret.

**Fields:**
- `v: u32` - [`CLERK_BOOTSTRAP_VERSION`](clerk.md#bookclerk_node_protoclerkclerk_bootstrap_version).
- `cluster_id: String` - The library's cluster id.
- `node_id: String` - The node id recorded for this clerk.
- `database: ClerkDatabase` - How to reach the library's database.
- `dek: crate::sealed::SecretValue` - The library's 32-byte data-encryption key.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ClerkBootstrap`
- **PartialEq**
  - `fn eq(self: &Self, other: &ClerkBootstrap) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_node_proto::clerk::ClerkDatabase

*Enum*

The library's database, as a clerk needs it to connect.

**Variants:**
- `Postgres{ url: crate::sealed::SecretValue }` - PostgreSQL: the connection URL (it carries credentials).
- `D1{ account_id: String, database_id: String, api_base: String }` - Cloudflare D1: the database's ids. The API token stays

**Methods:**

- `fn plugin(self: &Self) -> &'static str` - The `[database].plugin` id (`postgres`, `d1`).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ClerkDatabase) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ClerkDatabase`



## bookclerk_node_proto::clerk::ConnectionType

*Enum*

How a joining install connects to a library.

**Variants:**
- `Branch` - A host node: its own database, work over the node socket (default).
- `Clerk` - An owner node: shares the library's database and DEK.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Stable text (`branch`, `clerk`).
- `fn parse(raw: &str) -> Option<Self>` - Parses `branch` or `clerk` (case-insensitive); anything else is
- `fn is_branch(self: &Self) -> bool` - True for `Self::Branch` (serde: the field is left out).
- `fn granted_scope(self: Self) -> &'static str` - The scope a code for this connection type is granted.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ConnectionType) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> ConnectionType`
- **Clone**
  - `fn clone(self: &Self) -> ConnectionType`



## bookclerk_node_proto::clerk::DEK_LEN

*Constant*: `usize`

Length of the data-encryption key.



## bookclerk_node_proto::clerk::clerk_bootstrap_aad

*Function*

Associated data of a clerk bootstrap for `node_id` of `cluster_id`: the
domain tag, then each field as a 4-byte big-endian length and its bytes.

```rust
fn clerk_bootstrap_aad(cluster_id: &str, node_id: &str) -> Vec<u8>
```



## bookclerk_node_proto::clerk::open_clerk_bootstrap

*Function*

Opens a bootstrap made by [`seal_clerk_bootstrap`](clerk.md#bookclerk_node_protoclerkseal_clerk_bootstrap) with the joining
node's box secret, for the cluster and node ids the redeem answered.

# Errors

Returns `SealError::Open` when the box was sealed to another key, for
another cluster or node, or was altered, and `SealError::Bundle` when
its content is malformed or names other ids.

```rust
fn open_clerk_bootstrap(recipient_secret: &[u8; 32], cluster_id: &str, node_id: &str, sealed: &str) -> Result<ClerkBootstrap, crate::sealed::SealError>
```



## bookclerk_node_proto::clerk::seal_clerk_bootstrap

*Function*

Seals `bootstrap` to the joining node's box key `recipient`, as
base64url text for the redeem answer.

# Errors

Returns `SealError::Key` for an invalid recipient key and
`SealError::Bundle` for a bootstrap that is not version 1 or whose DEK
is not 32 bytes.

```rust
fn seal_clerk_bootstrap(recipient: &[u8], ephemeral_secret: &[u8; 32], bootstrap: &ClerkBootstrap) -> Result<String, crate::sealed::SealError>
```



