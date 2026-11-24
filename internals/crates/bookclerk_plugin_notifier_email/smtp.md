# bookclerk_plugin_notifier_email::smtp

SMTP submission, with no I/O of its own.

Everything the notifier reads from outside is parsed here: the server's
replies (RFC 5321 section 4.2), its EHLO capabilities, a patron's address
document and the operator's addresses. The message is built here too
(RFC 5322 headers, MIME `multipart/alternative` with quoted-printable
text, `multipart/related` for inline images, RFC 2047 encoded words), and [`Session`](#session) is the client's state machine:
it is fed one reply at a time and answers with the next thing to write, a
TLS upgrade, or the end. [`crate::client`](client.md) only moves bytes, so the unit
tests and the fuzzer drive the same code the guest runs.

No error text carries an address, a credential, the message or the
server's own reply text: those may be personal or secret. Errors name the
step and the reply code only.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Reply`](#reply) | struct | One complete server reply. |
| [`Capabilities`](#capabilities) | struct | What an EHLO reply says the server supports. |
| [`Address`](#address) | struct | A patron's address, checked. |
| [`HostPort`](#hostport) | struct | A TCP destination. |
| [`MailContent`](#mailcontent) | struct | What goes into one email. |
| [`InlineImage`](#inlineimage) | struct | An image sent inside the message and shown by the HTML as `cid:<content_id>`, so it appears even when the library is not reachable from the recipient's network and no remote image is fetched. |
| [`Credentials`](#credentials) | struct | Credentials for `AUTH`. |
| [`SessionOptions`](#sessionoptions) | struct | What one session sends. |
| [`Session`](#session) | struct | The client side of one SMTP submission: greeting, EHLO, STARTTLS and EHLO again, AUTH, MAIL, one RCPT per recipient, DATA, the message, QUIT. |
| [`Security`](#security) | enum | How the connection is protected. |
| [`Step`](#step) | enum | What the session asks the caller to do next. |
| [`read_reply`](#read_reply) | function | Takes the first complete reply off the front of what the server has sent. |
| [`parse_ehlo`](#parse_ehlo) | function | Reads the extensions an EHLO reply lists (RFC 5321 section 4.1.1.1). The first line is the server's greeting; each later line is a keyword and its parameters. Unknown keywords are ignored. |
| [`parse_email`](#parse_email) | function | Parses one email address: an ASCII dot-atom local part, `@`, and a host name of two or more labels whose last is not all digits. Quoted local parts, address literals and non-ASCII addresses are refused, so the address is safe inside `<...>` on a command line and in a header. |
| [`parse_address`](#parse_address) | function | Parses a patron's address document: a JSON object whose only field is `email`, a string. |
| [`parse_recipients`](#parse_recipients) | function | Parses the `to` setting: addresses separated by commas, semicolons or white space. |
| [`split_host_port`](#split_host_port) | function | Splits a granted destination, `host:port` or `[v6]:port`, as the host states it in the network grant. |
| [`one_line`](#one_line) | function | Drops control characters (line breaks included) and trims, so text from a message can never start a new header. |
| [`encode_header_text`](#encode_header_text) | function | Encodes header text: printable ASCII as is, other text as RFC 2047 `B` encoded words of at most 72 characters, folded onto continuation lines. |
| [`mailbox`](#mailbox) | function | Formats a `From` mailbox: the address alone, or a display name (quoted, or as encoded words when not ASCII) and the address in angle brackets. |
| [`rfc5322_date`](#rfc5322_date) | function | Formats a Unix time as an RFC 5322 date, in UTC. |
| [`encode_quoted_printable`](#encode_quoted_printable) | function | Encodes text as quoted-printable (RFC 2045 section 6.7): printable ASCII stays readable, so filters that dock base64-encoded text don't, and every other byte becomes `=XX`. Lines end in CRLF and run at most 76 characters, breaking with a soft `=` where they would run longer; a space or tab at a line's end is encoded, and so is the `F` of a line that starts `From `, so no relay rewrites it. |
| [`build_message`](#build_message) | function | Builds the message: RFC 5322 headers, then the text alone or a `multipart/alternative` of the text and the HTML; with inline images, the HTML and the images form a `multipart/related` inside it. Every line is ASCII, ends in CRLF and runs at most 76 characters in a body. |
| [`message_id`](#message_id) | function | Makes a stable `Message-ID`: the notification and destination ids (only their safe characters) at the sender's domain, so every retry of one send carries the same id. |
| [`escape_html`](#escape_html) | function | Escapes text for HTML. |
| [`dot_stuff`](#dot_stuff) | function | Applies SMTP's transparency rule (RFC 5321 section 4.5.2) and the end of data: a line starting with `.` gets another, and the data ends in `CRLF.CRLF`. |
| [`MAX_REPLY_LINE_BYTES`](#max_reply_line_bytes) | constant | Longest reply line accepted, in bytes (RFC 5321 allows 512; servers run longer). |
| [`MAX_REPLY_LINES`](#max_reply_lines) | constant | Most lines one reply may have. |
| [`MAX_EMAIL_CHARS`](#max_email_chars) | constant | Longest address accepted, in characters (RFC 5321's path limit less the brackets). |
| [`MAX_LOCAL_PART_CHARS`](#max_local_part_chars) | constant | Longest local part accepted, in characters (RFC 5321 section 4.5.3.1.1). |
| [`MAX_ADDRESS_BYTES`](#max_address_bytes) | constant | Longest address document accepted, in bytes (the ABI's `maxNotifierAddressBytes`). |
| [`MAX_CHANNEL_RECIPIENTS`](#max_channel_recipients) | constant | Most recipients the `to` setting may list. |
| [`MAX_FROM_NAME_CHARS`](#max_from_name_chars) | constant | Longest display name kept in `From`, in characters. |

## Structs

### Reply

```rust
pub struct Reply {
    pub code: u16,
    pub lines: Vec<String>,
}
```

One complete server reply.

**Fields**

- `code`: Three-digit reply code, 200 to 599.
- `lines`: Each line's text after the code and separator, in order (invalid
  UTF-8 replaced); at least one.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Capabilities

```rust
pub struct Capabilities {
    pub starttls: bool,
    pub auth: BTreeSet<String>,
    pub size: Option<u64>,
}
```

What an EHLO reply says the server supports.

**Fields**

- `starttls`: `STARTTLS` is offered.
- `auth`: SASL mechanisms from `AUTH`, upper-case.
- `size`: `SIZE` limit in bytes, when the server states one above zero.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### Address

```rust
pub struct Address {
    pub address_json: String,
    pub email: String,
}
```

A patron's address, checked.

**Fields**

- `address_json`: The document the host saves: `{"email": ...}` with the address
  normalised.
- `email`: The address.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HostPort

```rust
pub struct HostPort {
    pub hostname: String,
    pub port: u16,
}
```

A TCP destination.

**Fields**

- `hostname`: Host name or address, without brackets.
- `port`: Port, 1 to 65535.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### MailContent

```rust
pub struct MailContent<'a> {
    pub from: &'a str,
    pub from_name: &'a str,
    pub to: &'a [String],
    pub subject: &'a str,
    pub text: &'a str,
    pub html: Option<&'a str>,
    pub images: &'a [InlineImage<'a>],
    pub message_id: &'a str,
    pub date: &'a str,
    pub boundary: &'a str,
}
```

What goes into one email.

**Fields**

- `from`: Sender address, already checked.
- `from_name`: Sender display name; may be empty.
- `to`: Recipient addresses, already checked; shown in `To`.
- `subject`: Subject line.
- `text`: Plain-text body.
- `html`: HTML body, when there is one.
- `images`: Images the HTML shows by `cid:` reference; ignored without HTML.
- `message_id`: Stable id across retries, so receivers can drop a repeat; without
  angle brackets.
- `date`: When it is sent, as an RFC 5322 date.
- `boundary`: MIME boundary; random in the guest, fixed in tests.

Implements: `Clone`, `Debug`

### InlineImage

```rust
pub struct InlineImage<'a> {
    pub content_id: &'a str,
    pub content_type: &'a str,
    pub filename: &'a str,
    pub bytes: &'a [u8],
}
```

An image sent inside the message and shown by the HTML as
`cid:<content_id>`, so it appears even when the library is not reachable
from the recipient's network and no remote image is fetched.

**Fields**

- `content_id`: The `Content-ID`, without angle brackets; ASCII letters, digits, `.`,
  `-`, `_` and one `@`.
- `content_type`: Media type, such as `image/png`.
- `filename`: File name offered when the recipient saves it; ASCII.
- `bytes`: The image file's bytes.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Credentials

```rust
pub struct Credentials {
    pub username: String,
    pub password: String,
}
```

Credentials for `AUTH`.

**Fields**

- `username`: User name.
- `password`: Password; never printed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SessionOptions

```rust
pub struct SessionOptions {
    pub security: Security,
    pub helo_name: String,
    pub credentials: Option<Credentials>,
    pub from: String,
    pub recipients: Vec<String>,
    pub message: String,
}
```

What one session sends.

**Fields**

- `security`: How the connection is protected.
- `helo_name`: Name the client gives in EHLO.
- `credentials`: Credentials, when the server needs them.
- `from`: Envelope sender, checked.
- `recipients`: Envelope recipients, checked; at least one.
- `message`: The message, built by [`build_message`](#build_message).

Implements: `Clone`, `Debug`

### Session

```rust
pub struct Session {
    /* private fields */
}
```

The client side of one SMTP submission: greeting, EHLO, STARTTLS and
EHLO again, AUTH, MAIL, one RCPT per recipient, DATA, the message, QUIT.

Credentials are sent only once the connection is TLS, so a server that
does not offer STARTTLS (or a meddler that strips it) gets none;
[`Security::None`](#security) with credentials is refused up front.

#### Session::new

```rust
pub fn new(opts: SessionOptions) -> Result<Self, PluginError>
```

Starts a session; the first reply fed in is the server's greeting.

##### Arguments

* `opts` - What to send and how.

##### Errors

`invalid_params` for credentials without TLS, or no recipient.

#### Session::finished

```rust
pub fn finished(&self) -> bool
```

Whether the server accepted the message.

#### Session::is_tls

```rust
pub fn is_tls(&self) -> bool
```

Whether the connection is TLS now. Credentials are written only when
it is.

#### Session::on_reply

```rust
pub fn on_reply(&mut self, reply: &Reply) -> Result<Step, PluginError>
```

Takes the next reply.

##### Arguments

* `reply` - A complete reply.

##### Returns

What to do next.

##### Errors

When the server refuses a step, offers no way to proceed, or answers
out of turn.

#### Session::tls_started

```rust
pub fn tls_started(&mut self) -> Result<Step, PluginError>
```

Continues once the connection is TLS: EHLO again, as RFC 3207
requires, since what the server said before the upgrade can't be
trusted.

##### Errors

`unavailable` when no upgrade was asked for.

Implements: `Debug`

## Enums

### Security

```rust
pub enum Security {
    StartTls,
    Tls,
    None,
}
```

How the connection is protected.

**Variants**

- `StartTls`: Plain at first, upgraded by `STARTTLS` before anything else is sent
  (usually port 587). TLS is required: a server that does not offer
  it gets nothing.
- `Tls`: TLS from the first byte (usually port 465).
- `None`: No TLS, for a relay on a network the operator trusts; never with
  credentials.

#### Security::parse

```rust
pub fn parse(value: &str) -> Result<Self, PluginError>
```

Parses the `security` setting.

##### Errors

`invalid_params` for anything but `starttls`, `tls` or `none`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Step

```rust
pub enum Step {
    Write(String),
    StartTls,
    Done(String),
}
```

What the session asks the caller to do next.

**Variants**

- `Write`: Write these ASCII bytes and wait for the next reply.
- `StartTls`: Upgrade the connection to TLS, then call [`Session::tls_started`](#sessiontls_started).
- `Done`: Write these bytes (`QUIT`) and close: the message was accepted.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### read_reply

```rust
pub fn read_reply(buffer: &[u8]) -> Result<Option<(Reply, usize)>, PluginError>
```

Takes the first complete reply off the front of what the server has sent.

Lines end in CRLF (a bare LF is accepted too). Every line of a reply has
the same code; all but the last put a hyphen after it. A line is a code
of `2`-`5` and two digits, then a space, a hyphen or nothing.

#### Arguments

* `buffer` - Unread bytes from the server.

#### Returns

The reply and how many bytes it took, or `None` while it is incomplete.

#### Errors

`unavailable` when the bytes are not a reply, a line is longer than
[`MAX_REPLY_LINE_BYTES`](#max_reply_line_bytes), a reply mixes codes or has more than
[`MAX_REPLY_LINES`](#max_reply_lines) lines.

### parse_ehlo

```rust
pub fn parse_ehlo(reply: &Reply) -> Capabilities
```

Reads the extensions an EHLO reply lists (RFC 5321 section 4.1.1.1). The
first line is the server's greeting; each later line is a keyword and its
parameters. Unknown keywords are ignored.

#### Arguments

* `reply` - A 250 reply to EHLO.

### parse_email

```rust
pub fn parse_email(raw: &str, what: &str) -> Result<String, PluginError>
```

Parses one email address: an ASCII dot-atom local part, `@`, and a host
name of two or more labels whose last is not all digits. Quoted local
parts, address literals and non-ASCII addresses are refused, so the
address is safe inside `<...>` on a command line and in a header.

#### Arguments

* `raw` - The address as written; surrounding white space is dropped.
* `what` - What the value is, for the error ("the from_address setting").

#### Returns

The address with its domain in lower case.

#### Errors

`invalid_params` when it is not such an address.

### parse_address

```rust
pub fn parse_address(address_json: &str) -> Result<Address, PluginError>
```

Parses a patron's address document: a JSON object whose only field is
`email`, a string.

#### Arguments

* `address_json` - The document as entered.

#### Errors

`invalid_params` when the document or the address is unusable.

### parse_recipients

```rust
pub fn parse_recipients(raw: &str) -> Result<Vec<String>, PluginError>
```

Parses the `to` setting: addresses separated by commas, semicolons or
white space.

#### Arguments

* `raw` - The setting.

#### Returns

The addresses, in order, without repeats.

#### Errors

`invalid_params` when it names none, more than
[`MAX_CHANNEL_RECIPIENTS`](#max_channel_recipients), or a bad address.

### split_host_port

```rust
pub fn split_host_port(destination: &str) -> Result<HostPort, PluginError>
```

Splits a granted destination, `host:port` or `[v6]:port`, as the host
states it in the network grant.

#### Arguments

* `destination` - The destination.

#### Errors

`invalid_params` when it has no usable host or port.

### one_line

```rust
pub fn one_line(text: &str) -> String
```

Drops control characters (line breaks included) and trims, so text from
a message can never start a new header.

#### Arguments

* `text` - Any text.

#### Returns

The text on one line, each run of control characters as one space.

### encode_header_text

```rust
pub fn encode_header_text(text: &str) -> String
```

Encodes header text: printable ASCII as is, other text as RFC 2047 `B`
encoded words of at most 72 characters, folded onto continuation lines.

#### Arguments

* `text` - Text on one line.

#### Returns

The header value.

### mailbox

```rust
pub fn mailbox(name: &str, address: &str) -> String
```

Formats a `From` mailbox: the address alone, or a display name (quoted,
or as encoded words when not ASCII) and the address in angle brackets.

#### Arguments

* `name` - Display name; may be empty.
* `address` - The checked address.

### rfc5322_date

```rust
pub fn rfc5322_date(unix_seconds: i64) -> String
```

Formats a Unix time as an RFC 5322 date, in UTC.

#### Arguments

* `unix_seconds` - Seconds since 1970-01-01T00:00:00Z.

#### Returns

`Mon, 23 Nov 2026 05:00:00 +0000`.

### encode_quoted_printable

```rust
pub fn encode_quoted_printable(body: &str) -> String
```

Encodes text as quoted-printable (RFC 2045 section 6.7): printable ASCII
stays readable, so filters that dock base64-encoded text don't, and
every other byte becomes `=XX`. Lines end in CRLF and run at most 76
characters, breaking with a soft `=` where they would run longer; a
space or tab at a line's end is encoded, and so is the `F` of a line
that starts `From `, so no relay rewrites it.

#### Arguments

* `body` - The text; any line ending becomes CRLF.

#### Returns

The encoded body, ASCII only.

### build_message

```rust
pub fn build_message(mail: &MailContent<'_>) -> String
```

Builds the message: RFC 5322 headers, then the text alone or a
`multipart/alternative` of the text and the HTML; with inline images,
the HTML and the images form a `multipart/related` inside it. Every
line is ASCII, ends in CRLF and runs at most 76 characters in a body.

#### Arguments

* `mail` - What goes in it.

#### Returns

The message, without SMTP's dot-stuffing.

### message_id

```rust
pub fn message_id(notification_id: &str, destination_id: &str, from: &str) -> String
```

Makes a stable `Message-ID`: the notification and destination ids (only
their safe characters) at the sender's domain, so every retry of one send
carries the same id.

#### Arguments

* `notification_id` - Host id of the notification.
* `destination_id` - Host id of the destination.
* `from` - The checked sender address.

#### Returns

The id, without angle brackets.

### escape_html

```rust
pub fn escape_html(text: &str) -> String
```

Escapes text for HTML.

#### Arguments

* `text` - Any text.

#### Returns

The text with `& < > " '` as entities.

### dot_stuff

```rust
pub fn dot_stuff(message: &str) -> String
```

Applies SMTP's transparency rule (RFC 5321 section 4.5.2) and the end of
data: a line starting with `.` gets another, and the data ends in
`CRLF.CRLF`.

#### Arguments

* `message` - The message, every line ending in CRLF.

#### Returns

What to write after the 354 reply to DATA.

## Constants

### MAX_REPLY_LINE_BYTES

```rust
pub const MAX_REPLY_LINE_BYTES: usize = 2048;
```

Longest reply line accepted, in bytes (RFC 5321 allows 512; servers run
longer).

### MAX_REPLY_LINES

```rust
pub const MAX_REPLY_LINES: usize = 128;
```

Most lines one reply may have.

### MAX_EMAIL_CHARS

```rust
pub const MAX_EMAIL_CHARS: usize = 254;
```

Longest address accepted, in characters (RFC 5321's path limit less the
brackets).

### MAX_LOCAL_PART_CHARS

```rust
pub const MAX_LOCAL_PART_CHARS: usize = 64;
```

Longest local part accepted, in characters (RFC 5321 section 4.5.3.1.1).

### MAX_ADDRESS_BYTES

```rust
pub const MAX_ADDRESS_BYTES: usize = _;
```

Longest address document accepted, in bytes (the ABI's
`maxNotifierAddressBytes`).

### MAX_CHANNEL_RECIPIENTS

```rust
pub const MAX_CHANNEL_RECIPIENTS: usize = 10;
```

Most recipients the `to` setting may list.

### MAX_FROM_NAME_CHARS

```rust
pub const MAX_FROM_NAME_CHARS: usize = 128;
```

Longest display name kept in `From`, in characters.
