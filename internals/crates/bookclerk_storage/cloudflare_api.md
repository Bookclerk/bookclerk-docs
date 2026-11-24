# bookclerk_storage::cloudflare_api

Cloudflare API replies the guided setups of the `r2` destination and the
`d1` database read, and the request parts they build.

The R2 setup calls the Cloudflare v4 API with the operator's token: it
finds the account, creates (or reuses) the bucket, looks up the
bucket-scoped permission group, mints a token for that one bucket, and
deletes the token it minted on an earlier run. The D1 setup finds (or
creates) the database, looks up the D1 write permission group, and mints
an account token that can use D1 and nothing else. Every reply comes from
outside Bookclerk, so this module is the boundary that reads them: pure
functions over bytes, with no I/O, fuzzed by the `cloudflare_api`
target. The caller does the HTTP.

Every reply is the v4 envelope:

```json
{"success": true, "errors": [], "messages": [], "result": ...}
```

A failed call keeps only its error codes and short, single-line
messages; nothing else from a failed reply reaches the operator.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ApiFault`](#apifault) | struct | One error a failed reply reported. |
| [`AccountSummary`](#accountsummary) | struct | One account the token can see. |
| [`MintedToken`](#mintedtoken) | struct | A token the create call minted. |
| [`ApiReplyError`](#apireplyerror) | enum | Why a reply could not be used. |
| [`BucketCreated`](#bucketcreated) | enum | What a bucket create call did. |
| [`parse_envelope`](#parse_envelope) | function | Reads the envelope and returns its result. |
| [`parse_accounts`](#parse_accounts) | function | Reads a `GET /accounts` reply. |
| [`parse_bucket_created`](#parse_bucket_created) | function | Reads a `POST /accounts/{id}/r2/buckets` reply. |
| [`find_permission_group`](#find_permission_group) | function | Reads a `GET /accounts/{id}/tokens/permission_groups` reply and returns the id of the group named `name`. |
| [`parse_minted_token`](#parse_minted_token) | function | Reads a `POST /accounts/{id}/tokens` reply. |
| [`parse_done`](#parse_done) | function | Reads a reply whose result the setup does not need (a token delete). |
| [`find_d1_database`](#find_d1_database) | function | Reads a `GET /accounts/{id}/d1/database?name=` reply and returns the id of the database named exactly `name` (the filter also matches names that only contain it). |
| [`parse_d1_created`](#parse_d1_created) | function | Reads a `POST /accounts/{id}/d1/database` reply and returns the new database's UUID, lowercased. |
| [`check_d1_database_name`](#check_d1_database_name) | function | Checks a D1 database name: 1 to `MAX_D1_DATABASE_NAME_CHARS` lowercase letters, digits, hyphens and underscores, starting with a letter or digit. |
| [`account_resource`](#account_resource) | function | The token policy resource that names a whole account. |
| [`check_bucket_name`](#check_bucket_name) | function | Checks an R2 bucket name: 3 to 63 lowercase letters, digits and hyphens, not starting or ending with a hyphen. |
| [`bucket_resource`](#bucket_resource) | function | The token policy resource that names one bucket. |
| [`token_request`](#token_request) | function | The body of the token create call: one policy allowing `group` on one bucket. |
| [`MAX_API_REPLY_BYTES`](#max_api_reply_bytes) | constant | Largest reply read, in bytes. The longest reply the setup reads is a filtered permission group list, a few kilobytes. |
| [`MAX_API_LIST_ENTRIES`](#max_api_list_entries) | constant | Most entries a list reply may hold. |
| [`MAX_API_ERRORS`](#max_api_errors) | constant | Most errors kept from a failed reply. |
| [`MAX_API_ERROR_MESSAGE_CHARS`](#max_api_error_message_chars) | constant | Longest error message kept, in characters. |
| [`MAX_TOKEN_VALUE_BYTES`](#max_token_value_bytes) | constant | Longest token value accepted from a create reply, in bytes. |
| [`BUCKET_ALREADY_OWNED`](#bucket_already_owned) | constant | The error code the bucket create call answers when the caller already owns a bucket of that name. |
| [`BUCKET_ITEM_WRITE`](#bucket_item_write) | constant | The permission group that lets a token read and write the objects of the buckets it names, and nothing else. |
| [`D1_WRITE`](#d1_write) | constant | The permission group that lets a token read and write the account's D1 databases (shown as "D1 Edit" in the dashboard). |
| [`D1_EDIT`](#d1_edit) | constant | The dashboard's name for `D1_WRITE`, which some accounts' permission group lists use. |
| [`MAX_D1_DATABASE_NAME_CHARS`](#max_d1_database_name_chars) | constant | Longest D1 database name accepted, in characters. |

## Structs

### ApiFault

```rust
pub struct ApiFault {
    pub code: Option<i64>,
    pub message: String,
}
```

One error a failed reply reported.

**Fields**

- `code`: Cloudflare's error code; `None` when the reply gave none.
- `message`: The message, cut to [`MAX_API_ERROR_MESSAGE_CHARS`](#max_api_error_message_chars) with control
  characters removed.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `PartialEq`

### AccountSummary

```rust
pub struct AccountSummary {
    pub id: String,
    pub name: String,
}
```

One account the token can see.

**Fields**

- `id`: The account id, 32 lowercase hex digits.
- `name`: The account's name, cleaned like an error message.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### MintedToken

```rust
pub struct MintedToken {
    pub id: String,
    pub value: String,
}
```

A token the create call minted.

**Fields**

- `id`: The token's id, which is the bucket's S3 access key id.
- `value`: The token's value, shown once; its SHA-256 is the S3 secret.

#### MintedToken::secret_access_key

```rust
pub fn secret_access_key(&self) -> String
```

The S3 secret access key: the SHA-256 of the value, in lowercase hex.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### ApiReplyError

```rust
pub enum ApiReplyError {
    TooLarge,
    Malformed(String),
    Refused(Vec<ApiFault>),
}
```

Why a reply could not be used.

**Variants**

- `TooLarge`: The reply is over [`MAX_API_REPLY_BYTES`](#max_api_reply_bytes).
- `Malformed`: The reply is not the envelope, or its result has the wrong shape.
- `Refused`: The API reported failure.

#### ApiReplyError::has_code

```rust
pub fn has_code(&self, code: i64) -> bool
```

Whether the API refused with `code`.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### BucketCreated

```rust
pub enum BucketCreated {
    Created,
    AlreadyOwned,
}
```

What a bucket create call did.

**Variants**

- `Created`: The bucket was created.
- `AlreadyOwned`: The caller already owned a bucket of that name; it is reused.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_envelope

```rust
pub fn parse_envelope(reply: &[u8]) -> Result<Value, ApiReplyError>
```

Reads the envelope and returns its result.

The reply is parsed as a JSON tree first, so a repeated member counts
once, as its last value.

#### Errors

[`ApiReplyError::TooLarge`](#apireplyerror) past the cap, [`ApiReplyError::Malformed`](#apireplyerror)
for anything but an envelope object, and [`ApiReplyError::Refused`](#apireplyerror) when
`success` is false.

### parse_accounts

```rust
pub fn parse_accounts(reply: &[u8]) -> Result<Vec<AccountSummary>, ApiReplyError>
```

Reads a `GET /accounts` reply.

#### Errors

As [`parse_envelope`](#parse_envelope), and [`ApiReplyError::Malformed`](#apireplyerror) when an entry's
id is not 32 hex digits.

### parse_bucket_created

```rust
pub fn parse_bucket_created(reply: &[u8]) -> Result<BucketCreated, ApiReplyError>
```

Reads a `POST /accounts/{id}/r2/buckets` reply.

#### Errors

As [`parse_envelope`](#parse_envelope), except that [`BUCKET_ALREADY_OWNED`](#bucket_already_owned) is
[`BucketCreated::AlreadyOwned`](#bucketcreated).

### find_permission_group

```rust
pub fn find_permission_group(
    reply: &[u8],
    name: &str,
) -> Result<Option<String>, ApiReplyError>
```

Reads a `GET /accounts/{id}/tokens/permission_groups` reply and returns
the id of the group named `name`.

#### Returns

The group's id, or `None` when the reply lists no group of that name.

#### Errors

As [`parse_envelope`](#parse_envelope), and [`ApiReplyError::Malformed`](#apireplyerror) when the
matching group's id is not 32 hex digits.

### parse_minted_token

```rust
pub fn parse_minted_token(reply: &[u8]) -> Result<MintedToken, ApiReplyError>
```

Reads a `POST /accounts/{id}/tokens` reply.

#### Errors

As [`parse_envelope`](#parse_envelope), and [`ApiReplyError::Malformed`](#apireplyerror) when the id is
not 32 hex digits or the value is empty, over
[`MAX_TOKEN_VALUE_BYTES`](#max_token_value_bytes), or not printable ASCII.

### parse_done

```rust
pub fn parse_done(reply: &[u8]) -> Result<(), ApiReplyError>
```

Reads a reply whose result the setup does not need (a token delete).

#### Errors

As [`parse_envelope`](#parse_envelope).

### find_d1_database

```rust
pub fn find_d1_database(reply: &[u8], name: &str) -> Result<Option<String>, ApiReplyError>
```

Reads a `GET /accounts/{id}/d1/database?name=` reply and returns the id
of the database named exactly `name` (the filter also matches names
that only contain it).

#### Returns

The database's UUID, lowercased, or `None` when none has that name.

#### Errors

As [`parse_envelope`](#parse_envelope), and [`ApiReplyError::Malformed`](#apireplyerror) when the
matching entry's `uuid` is not a UUID.

### parse_d1_created

```rust
pub fn parse_d1_created(reply: &[u8]) -> Result<String, ApiReplyError>
```

Reads a `POST /accounts/{id}/d1/database` reply and returns the new
database's UUID, lowercased.

#### Errors

As [`parse_envelope`](#parse_envelope), and [`ApiReplyError::Malformed`](#apireplyerror) when the result
has no UUID.

### check_d1_database_name

```rust
pub fn check_d1_database_name(name: &str) -> Result<(), String>
```

Checks a D1 database name: 1 to [`MAX_D1_DATABASE_NAME_CHARS`](#max_d1_database_name_chars)
lowercase letters, digits, hyphens and underscores, starting with a
letter or digit.

#### Errors

A sentence saying what is wrong.

### account_resource

```rust
pub fn account_resource(account: &str) -> String
```

The token policy resource that names a whole account.

#### Arguments

* `account` - the account id.

### check_bucket_name

```rust
pub fn check_bucket_name(name: &str) -> Result<(), String>
```

Checks an R2 bucket name: 3 to 63 lowercase letters, digits and hyphens,
not starting or ending with a hyphen.

#### Errors

A sentence saying what is wrong.

### bucket_resource

```rust
pub fn bucket_resource(account: &str, jurisdiction: &str, bucket: &str) -> String
```

The token policy resource that names one bucket.

#### Arguments

* `account` - the account id.
* `jurisdiction` - `default`, `eu` or `fedramp`.
* `bucket` - a name [`check_bucket_name`](#check_bucket_name) accepts.

### token_request

```rust
pub fn token_request(name: &str, group: &str, resource: &str) -> Value
```

The body of the token create call: one policy allowing `group` on one
bucket.

## Constants

### MAX_API_REPLY_BYTES

```rust
pub const MAX_API_REPLY_BYTES: usize = _;
```

Largest reply read, in bytes. The longest reply the setup reads is a
filtered permission group list, a few kilobytes.

### MAX_API_LIST_ENTRIES

```rust
pub const MAX_API_LIST_ENTRIES: usize = 1000;
```

Most entries a list reply may hold.

### MAX_API_ERRORS

```rust
pub const MAX_API_ERRORS: usize = 4;
```

Most errors kept from a failed reply.

### MAX_API_ERROR_MESSAGE_CHARS

```rust
pub const MAX_API_ERROR_MESSAGE_CHARS: usize = 160;
```

Longest error message kept, in characters.

### MAX_TOKEN_VALUE_BYTES

```rust
pub const MAX_TOKEN_VALUE_BYTES: usize = 1024;
```

Longest token value accepted from a create reply, in bytes.

### BUCKET_ALREADY_OWNED

```rust
pub const BUCKET_ALREADY_OWNED: i64 = 10004;
```

The error code the bucket create call answers when the caller already
owns a bucket of that name.

### BUCKET_ITEM_WRITE

```rust
pub const BUCKET_ITEM_WRITE: &str = "Workers R2 Storage Bucket Item Write";
```

The permission group that lets a token read and write the objects of
the buckets it names, and nothing else.

### D1_WRITE

```rust
pub const D1_WRITE: &str = "D1 Write";
```

The permission group that lets a token read and write the account's
D1 databases (shown as "D1 Edit" in the dashboard).

### D1_EDIT

```rust
pub const D1_EDIT: &str = "D1 Edit";
```

The dashboard's name for [`D1_WRITE`](#d1_write), which some accounts' permission
group lists use.

### MAX_D1_DATABASE_NAME_CHARS

```rust
pub const MAX_D1_DATABASE_NAME_CHARS: usize = 64;
```

Longest D1 database name accepted, in characters.
