# bookclerk_plugin_abi::notifications

The checks the host runs on notification requests and replies.

Three things cross a trust boundary here, and each is checked by sans-IO
code that the fuzz targets and the host share:

- **A notifier's reply** to `Notifier.send` ([`classify_send_reply`](#classify_send_reply)):
  delivered, retry later or failed for good. The plugin's error text is
  cut to a short single line ([`sanitize_reason`](#sanitize_reason)) before the delivery
  log stores it.
- **A destination address document** ([`parse_address_document`](#parse_address_document)): the
  JSON object a patron typed (an email address, a Telegram chat id, a
  push subscription) and that a notifier's `validateDestination`
  normalised. Only a flat object of short string values is accepted.
- **A plugin's raise** on the `NOTIFY` binding ([`check_raise_params`](#check_raise_params)):
  a declared type id, labelled fields, an optional patron and a path
  inside the library.

The host also checks every message it renders ([`check_message`](#check_message))
before a notifier sees it, so a bug in a renderer cannot hand a plugin
an oversized body or a link to somewhere other than the library.

| Item | Kind | Summary |
| --- | --- | --- |
| [`NotifyAddress`](#notifyaddress) | struct | A destination address document: a flat JSON object of string values, keyed by the `[[notifier.address_fields]]` keys. Values may be secrets (a push subscription's auth key); the type never prints them. |
| [`AddressFieldRule`](#addressfieldrule) | struct | One address field a notifier declares in `[[notifier.address_fields]]`. |
| [`NotificationError`](#notificationerror) | enum | Why a notification request, message or reply was refused. |
| [`SendOutcome`](#sendoutcome) | enum | What the host does with one destination after a `Notifier.send` reply. |
| [`is_notification_type_id`](#is_notification_type_id) | function | Whether `id` is a local notification type id: `a-z{0,63}`. |
| [`is_qualified_notification_type`](#is_qualified_notification_type) | function | Whether `id` is a routed notification type id: a host type (`job_failed`) or a plugin type qualified by its plugin id (`ntfy_relay:sync_failed`, plugin ids being `[a-z0-9_]{2,32}`). |
| [`is_notify_id`](#is_notify_id) | function | Whether `id` is a usable id: 1 to `MAX_NOTIFY_ID_BYTES` bytes of printable ASCII with no space. |
| [`is_layout_char`](#is_layout_char) | function | Whether `c` breaks a line or reorders text without being a control character: the Unicode line and paragraph separators and the bidirectional marks, embeddings, overrides and isolates. A one-line value refuses them so it shows as written. |
| [`check_link`](#check_link) | function | Checks an absolute link into the library: `https://host[:port]/...`, or `http://` only to `localhost`, `127.0.0.1` or `[::1]`; no credentials, no whitespace or control characters, at most `MAX_NOTIFICATION_FIELD_BYTES`. |
| [`check_link_path`](#check_link_path) | function | Checks a path inside the library that the host will prefix with its public origin: starts with one `/`, no `//` at the start, no backslash, whitespace or control characters, at most `MAX_NOTIFICATION_FIELD_BYTES`. |
| [`check_message`](#check_message) | function | Checks a message the host rendered before any notifier sees it. |
| [`parse_address_document`](#parse_address_document) | function | Parses a destination address document. |
| [`check_address_against_fields`](#check_address_against_fields) | function | Checks an address document against the fields the notifier declares: every required field present and non-blank, no undeclared field. |
| [`check_send_params`](#check_send_params) | function | Checks a `Notifier.send` request before it leaves the host. |
| [`check_validated`](#check_validated) | function | Checks a `Notifier.validateDestination` answer: a well-formed address document and an optional short single-line label. |
| [`check_raise_params`](#check_raise_params) | function | Checks a raise a plugin sent on its `NOTIFY` binding. Whether the type is one the plugin declared, and whether the patron is inside the invocation's account scope, are the host's checks, since only it knows the manifest and the scope. |
| [`classify_send_reply`](#classify_send_reply) | function | Turns a `Notifier.send` reply into what the host does next. |
| [`sanitize_reason`](#sanitize_reason) | function | Cuts a notifier's error text to one line of at most `MAX_NOTIFY_REASON_BYTES` bytes: control characters become spaces and the cut falls on a character boundary. |
| [`severity_rank`](#severity_rank) | function | Rank of a severity for comparing against a rule's minimum: info 0, warning 1, critical 2. |
| [`decode_notify_send_reply_bytes`](#decode_notify_send_reply_bytes) | function | Decodes a `NotifySendReply` from an unpacked Cap'n Proto message, the form a notifier's reply crosses the plugin boundary in. |
| [`encode_notify_send_reply_bytes`](#encode_notify_send_reply_bytes) | function | Encodes a `NotifySendReply`, the inverse of `decode_notify_send_reply_bytes`. |
| [`decode_notify_raise_params_bytes`](#decode_notify_raise_params_bytes) | function | Decodes `NotifyRaiseParams` from an unpacked Cap'n Proto message, the form a plugin's raise crosses the plugin boundary in. |
| [`encode_notify_raise_params_bytes`](#encode_notify_raise_params_bytes) | function | Encodes `NotifyRaiseParams`, the inverse of `decode_notify_raise_params_bytes`. |
| [`MAX_NOTIFY_ID_BYTES`](#max_notify_id_bytes) | constant | Longest notification id, destination id or patron id, in bytes. Ids are printable ASCII without spaces. |
| [`MAX_NOTIFICATION_TYPE_ID_BYTES`](#max_notification_type_id_bytes) | constant | Longest local notification type id (`job_failed`), in bytes; type ids follow the ABI identifier rule `a-z{0,63}`. |
| [`MAX_NOTIFY_ADDRESS_KEYS`](#max_notify_address_keys) | constant | Most keys in one destination address document. |
| [`MAX_NOTIFY_REASON_BYTES`](#max_notify_reason_bytes) | constant | Longest reason the delivery log keeps from a notifier's error, in bytes. |
| [`MAX_NOTIFY_RETRY_AFTER`](#max_notify_retry_after) | constant | Longest wait the host honours from a notifier's `retryAfterMs`. A service asking for longer is retried after this long instead. |

## Structs

### NotifyAddress

```rust
pub struct NotifyAddress {
    /* private fields */
}
```

A destination address document: a flat JSON object of string values,
keyed by the `[[notifier.address_fields]]` keys. Values may be secrets
(a push subscription's auth key); the type never prints them.

#### NotifyAddress::get

```rust
pub fn get(&self, key: &str) -> Option<&str>
```

The value of field `key`, if present.

#### NotifyAddress::keys

```rust
pub fn keys(&self) -> impl Iterator<Item = &str>
```

Field keys, sorted.

#### NotifyAddress::len

```rust
pub fn len(&self) -> usize
```

Number of fields.

#### NotifyAddress::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Whether the document has no fields.

#### NotifyAddress::to_json

```rust
pub fn to_json(&self) -> String
```

The canonical JSON text of the document (keys sorted, no spaces),
the form the host stores and sends.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### AddressFieldRule

```rust
pub struct AddressFieldRule {
    pub key: String,
    pub required: bool,
}
```

One address field a notifier declares in `[[notifier.address_fields]]`.

**Fields**

- `key`: Field key (`email`, `chat_id`).
- `required`: Whether a personal destination must carry it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### NotificationError

```rust
pub enum NotificationError {
    BadTypeId,
    BadId(&'static str),
    BadText(&'static str),
    TooManyFields,
    BadLink,
    BadLinkPath,
    BadAlert,
    BadAddress(&'static str),
    MissingAddressField(String),
    UnknownAddressField(String),
    AddressKindMismatch,
}
```

Why a notification request, message or reply was refused.

**Variants**

- `BadTypeId`: A notification type id is not `[a-z][a-z0-9_]{0,63}`, optionally
  prefixed by a plugin id and `:`.
- `BadId`: An id (notification, destination, patron) is empty, too long, or
  holds a space or a non-printable byte.
- `BadText`: A text value is empty where it may not be, over its cap, or holds
  a control character it may not.
- `TooManyFields`: More than [`MAX_NOTIFICATION_FIELDS`](README.md#max_notification_fields) fields.
- `BadLink`: A link is not an absolute `https` URL (or `http` to a loopback
  host), carries credentials, or is too long.
- `BadLinkPath`: A link path is not an absolute path inside the library.
- `BadAlert`: An alert action other than `none` came without an alert key, or an
  alert key came without an action.
- `BadAddress`: An address document is not a flat JSON object of short strings, or
  is over [`MAX_NOTIFIER_ADDRESS_BYTES`](README.md#max_notifier_address_bytes).
- `MissingAddressField`: An address document lacks a field the notifier requires.
- `UnknownAddressField`: An address document carries a field the notifier does not declare.
- `AddressKindMismatch`: A channel destination carried an address, or a personal one none.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<NotificationError>`, `PartialEq`

### SendOutcome

```rust
pub enum SendOutcome {
    Delivered { provider_id: Option<String> },
    Retry { after: Option<Duration>, reason: String },
    Failed { reason: String },
}
```

What the host does with one destination after a `Notifier.send` reply.

**Variants**

- `Delivered`: Delivered; the service's id for the message, when it gave a usable
  one.
  - `provider_id`: Service message id, at most [`MAX_NOTIFICATION_FIELD_BYTES`](README.md#max_notification_field_bytes) of
    printable text; dropped when malformed.
- `Retry`: Try again later.
  - `after`: How long the service asked for, capped at
    [`MAX_NOTIFY_RETRY_AFTER`](#max_notify_retry_after); `None` uses the job's backoff.
  - `reason`: Short single-line reason for the delivery log.
- `Failed`: Give up on this destination.
  - `reason`: Short single-line reason for the delivery log.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### is_notification_type_id

```rust
pub fn is_notification_type_id(id: &str) -> bool
```

Whether `id` is a local notification type id: `[a-z][a-z0-9_]{0,63}`.

#### Arguments

* `id` - The type id as a plugin declares it, without its plugin prefix.

#### Returns

True when the id is well formed.

### is_qualified_notification_type

```rust
pub fn is_qualified_notification_type(id: &str) -> bool
```

Whether `id` is a routed notification type id: a host type
(`job_failed`) or a plugin type qualified by its plugin id
(`ntfy_relay:sync_failed`, plugin ids being `[a-z0-9_]{2,32}`).

#### Arguments

* `id` - The type id as the router and the routing rules name it.

#### Returns

True when the id is well formed.

### is_notify_id

```rust
pub fn is_notify_id(id: &str) -> bool
```

Whether `id` is a usable id: 1 to [`MAX_NOTIFY_ID_BYTES`](#max_notify_id_bytes) bytes of
printable ASCII with no space.

#### Arguments

* `id` - The id to check.

#### Returns

True when the id is well formed.

### is_layout_char

```rust
pub fn is_layout_char(c: char) -> bool
```

Whether `c` breaks a line or reorders text without being a control
character: the Unicode line and paragraph separators and the
bidirectional marks, embeddings, overrides and isolates. A one-line
value refuses them so it shows as written.

#### Arguments

* `c` - The character to test.

#### Returns

True for U+2028, U+2029, U+061C, U+200E, U+200F, U+202A to U+202E and
U+2066 to U+2069.

### check_link

```rust
pub fn check_link(link: &str) -> Result<(), NotificationError>
```

Checks an absolute link into the library: `https://host[:port]/...`, or
`http://` only to `localhost`, `127.0.0.1` or `[::1]`; no credentials,
no whitespace or control characters, at most
[`MAX_NOTIFICATION_FIELD_BYTES`](README.md#max_notification_field_bytes).

#### Arguments

* `link` - The link to check.

#### Errors

[`NotificationError::BadLink`](#notificationerror).

### check_link_path

```rust
pub fn check_link_path(path: &str) -> Result<(), NotificationError>
```

Checks a path inside the library that the host will prefix with its
public origin: starts with one `/`, no `//` at the start, no
backslash, whitespace or control characters, at most
[`MAX_NOTIFICATION_FIELD_BYTES`](README.md#max_notification_field_bytes).

#### Arguments

* `path` - The path a raise names (`/books/123`).

#### Errors

[`NotificationError::BadLinkPath`](#notificationerror).

### check_message

```rust
pub fn check_message(message: &NotificationMessage) -> Result<(), NotificationError>
```

Checks a message the host rendered before any notifier sees it.

#### Arguments

* `message` - The rendered message.

#### Errors

The first [`NotificationError`](#notificationerror) the message breaks: a bad id or type,
a blank or oversized title, an oversized body, a link outside the
rules, too many fields, or a mismatched alert.

### parse_address_document

```rust
pub fn parse_address_document(text: &str) -> Result<NotifyAddress, NotificationError>
```

Parses a destination address document.

The document must be a JSON object of at most
[`MAX_NOTIFY_ADDRESS_KEYS`](#max_notify_address_keys) keys, each key an identifier
(`[a-z][a-z0-9_]{0,63}`) appearing once, each value a string of at most
[`MAX_NOTIFICATION_FIELD_BYTES`](README.md#max_notification_field_bytes) bytes with no control characters; the
whole text at most [`MAX_NOTIFIER_ADDRESS_BYTES`](README.md#max_notifier_address_bytes).

#### Arguments

* `text` - The document as a patron or a notifier gave it.

#### Returns

The parsed document.

#### Errors

[`NotificationError::BadAddress`](#notificationerror) saying what is wrong.

### check_address_against_fields

```rust
pub fn check_address_against_fields(
    address: &NotifyAddress,
    rules: &[AddressFieldRule],
) -> Result<(), NotificationError>
```

Checks an address document against the fields the notifier declares:
every required field present and non-blank, no undeclared field.

#### Arguments

* `address` - The parsed document.
* `rules` - The notifier's declared address fields.

#### Errors

[`NotificationError::MissingAddressField`](#notificationerror) or
[`NotificationError::UnknownAddressField`](#notificationerror).

### check_send_params

```rust
pub fn check_send_params(params: &NotifySendParams) -> Result<(), NotificationError>
```

Checks a `Notifier.send` request before it leaves the host.

#### Arguments

* `params` - The message and destination.

#### Errors

The message's [`check_message`](#check_message) error, a bad destination id, or an
address that does not fit the destination kind.

### check_validated

```rust
pub fn check_validated(
    validated: &NotifyValidated,
) -> Result<NotifyAddress, NotificationError>
```

Checks a `Notifier.validateDestination` answer: a well-formed address
document and an optional short single-line label.

#### Arguments

* `validated` - The notifier's answer.

#### Returns

The parsed, normalised document.

#### Errors

[`NotificationError::BadAddress`](#notificationerror) or [`NotificationError::BadText`](#notificationerror).

### check_raise_params

```rust
pub fn check_raise_params(params: &NotifyRaiseParams) -> Result<(), NotificationError>
```

Checks a raise a plugin sent on its `NOTIFY` binding. Whether the type
is one the plugin declared, and whether the patron is inside the
invocation's account scope, are the host's checks, since only it knows
the manifest and the scope.

#### Arguments

* `params` - The raise.

#### Errors

The first [`NotificationError`](#notificationerror) the raise breaks.

### classify_send_reply

```rust
pub fn classify_send_reply(reply: &Result<NotifySent, PluginError>) -> SendOutcome
```

Turns a `Notifier.send` reply into what the host does next.

`rate_limited`, `unavailable`, `deadline_exceeded` and `cancelled` are
retried, `rate_limited` after the
service's `retryAfterMs` when it gave one; every other error is a
permanent failure for this destination. A delivered reply with a
malformed provider id is still delivered, without the id.

#### Arguments

* `reply` - The decoded reply.

#### Returns

The outcome, with any reason cut by [`sanitize_reason`](#sanitize_reason).

### sanitize_reason

```rust
pub fn sanitize_reason(reason: &str) -> String
```

Cuts a notifier's error text to one line of at most
[`MAX_NOTIFY_REASON_BYTES`](#max_notify_reason_bytes) bytes: control characters become spaces and
the cut falls on a character boundary.

#### Arguments

* `reason` - The text to cut.

#### Returns

The cut text.

### severity_rank

```rust
pub fn severity_rank(severity: NotificationSeverity) -> u8
```

Rank of a severity for comparing against a rule's minimum: info 0,
warning 1, critical 2.

#### Arguments

* `severity` - The severity to rank.

#### Returns

Its rank.

### decode_notify_send_reply_bytes

```rust
pub fn decode_notify_send_reply_bytes(
    bytes: &[u8],
) -> Result<Result<NotifySent, PluginError>>
```

Decodes a `NotifySendReply` from an unpacked Cap'n Proto message, the
form a notifier's reply crosses the plugin boundary in.

#### Arguments

* `bytes` - The message bytes.

#### Returns

The reply: delivered, or the notifier's typed error.

#### Errors

Returns the Cap'n Proto error the reader reports.

### encode_notify_send_reply_bytes

```rust
pub fn encode_notify_send_reply_bytes(
    reply: &Result<NotifySent, PluginError>,
) -> Result<Vec<u8>>
```

Encodes a `NotifySendReply`, the inverse of
[`decode_notify_send_reply_bytes`](#decode_notify_send_reply_bytes).

#### Arguments

* `reply` - The reply to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

### decode_notify_raise_params_bytes

```rust
pub fn decode_notify_raise_params_bytes(bytes: &[u8]) -> Result<NotifyRaiseParams>
```

Decodes `NotifyRaiseParams` from an unpacked Cap'n Proto message, the
form a plugin's raise crosses the plugin boundary in.

#### Arguments

* `bytes` - The message bytes.

#### Returns

The raise.

#### Errors

Returns the Cap'n Proto error the reader reports.

### encode_notify_raise_params_bytes

```rust
pub fn encode_notify_raise_params_bytes(params: &NotifyRaiseParams) -> Result<Vec<u8>>
```

Encodes `NotifyRaiseParams`, the inverse of
[`decode_notify_raise_params_bytes`](#decode_notify_raise_params_bytes).

#### Arguments

* `params` - The raise to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

## Constants

### MAX_NOTIFY_ID_BYTES

```rust
pub const MAX_NOTIFY_ID_BYTES: usize = 128;
```

Longest notification id, destination id or patron id, in bytes. Ids are
printable ASCII without spaces.

### MAX_NOTIFICATION_TYPE_ID_BYTES

```rust
pub const MAX_NOTIFICATION_TYPE_ID_BYTES: usize = 64;
```

Longest local notification type id (`job_failed`), in bytes; type ids
follow the ABI identifier rule `[a-z][a-z0-9_]{0,63}`.

### MAX_NOTIFY_ADDRESS_KEYS

```rust
pub const MAX_NOTIFY_ADDRESS_KEYS: usize = 16;
```

Most keys in one destination address document.

### MAX_NOTIFY_REASON_BYTES

```rust
pub const MAX_NOTIFY_REASON_BYTES: usize = 512;
```

Longest reason the delivery log keeps from a notifier's error, in bytes.

### MAX_NOTIFY_RETRY_AFTER

```rust
pub const MAX_NOTIFY_RETRY_AFTER: Duration = _;
```

Longest wait the host honours from a notifier's `retryAfterMs`. A
service asking for longer is retried after this long instead.
