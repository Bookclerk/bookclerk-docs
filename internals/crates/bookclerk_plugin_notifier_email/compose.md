# bookclerk_plugin_notifier_email::compose

Turns a rendered notification into an email, with no I/O of its own.

The host renders the message (title, plain text, optional minimal HTML,
fields and a link built from the library's public origin). This module
lays those out as a plain-text body and an HTML body, then hands them to
[`build_message`](smtp.md#build_message).

| Item | Kind | Summary |
| --- | --- | --- |
| [`Envelope`](#envelope) | struct | Who sends one email, and to whom. |
| [`plain_body`](#plain_body) | function | The plain-text body: the text, then each field on its own line, then the link when the text does not already hold it. |
| [`html_body`](#html_body) | function | The HTML body: the host's HTML (or the text, escaped, one paragraph per blank-line block), the fields as a table and the link as an anchor, in a minimal document. |
| [`compose`](#compose) | function | Builds the whole email for one destination. |

## Structs

### Envelope

```rust
pub struct Envelope<'a> {
    pub from: &'a str,
    pub from_name: &'a str,
    pub to: &'a [String],
    pub destination_id: &'a str,
    pub date: &'a str,
    pub boundary: &'a str,
}
```

Who sends one email, and to whom.

**Fields**

- `from`: Sender address, checked.
- `from_name`: Sender display name; may be empty.
- `to`: Recipient addresses, checked.
- `destination_id`: Host id of the destination, part of the stable `Message-ID`.
- `date`: When it is sent, as an RFC 5322 date.
- `boundary`: MIME boundary.

Implements: `Clone`, `Debug`

## Functions

### plain_body

```rust
pub fn plain_body(message: &NotificationMessage) -> String
```

The plain-text body: the text, then each field on its own line, then the
link when the text does not already hold it.

#### Arguments

* `message` - The rendered message.

### html_body

```rust
pub fn html_body(message: &NotificationMessage) -> String
```

The HTML body: the host's HTML (or the text, escaped, one paragraph per
blank-line block), the fields as a table and the link as an anchor, in a
minimal document.

#### Arguments

* `message` - The rendered message.

### compose

```rust
pub fn compose(message: &NotificationMessage, envelope: &Envelope<'_>) -> String
```

Builds the whole email for one destination.

#### Arguments

* `message` - The rendered message.
* `envelope` - Sender, recipients and the per-send values.

#### Returns

The message, ready for DATA (before dot-stuffing).
