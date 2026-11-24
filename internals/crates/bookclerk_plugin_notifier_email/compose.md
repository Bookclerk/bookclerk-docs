# bookclerk_plugin_notifier_email::compose

Turns a rendered notification into an email, with no I/O of its own.

The host renders the message (title, plain text, optional minimal HTML,
fields and a link built from the library's public origin). This module
lays those out as a plain-text body and a branded HTML body, compiled
from MJML with mrml so it renders alike in Outlook, Gmail and Apple Mail,
with the logo sent inside the message rather than fetched; then hands
them to [`build_message`](smtp.md#build_message).

| Item | Kind | Summary |
| --- | --- | --- |
| [`Envelope`](#envelope) | struct | Who sends one email, and to whom. |
| [`plain_body`](#plain_body) | function | The plain-text body: the text, then each field on its own line, then the link under its short label when the text does not already hold it, then the footer after a signature separator. |
| [`worded_for`](#worded_for) | function | The message as this sender words it. An invite's subject names the library by the From name (`Alex has invited you to Riverside Library`) instead of the host's generic `a Bookclerk library`, and its `Invited by` field is left out because the subject already names them. Every other message is returned as it is. |
| [`plain_footer`](#plain_footer) | function | The footer under every message as plain text: who sent it (the library's home page when the host gave one) and Bookclerk's site. The HTML footer says the same words, which spam filters compare. |
| [`link_host`](#link_host) | function | The host part of a link (`library.example.com`, with a port when one is given), for the footer's link to the library's home page; `None` when the link has no `scheme://host`. |
| [`link_label`](#link_label) | function | The label of the link button: short, naming the library it opens (`Open Riverside Library`), or `Open your library` when the From name is just the product's. The plain-text part keeps the full URL beside it. |
| [`mjml_source`](#mjml_source) | function | The MJML source of the branded message: the Bookclerk mark and name as the header; a card with the sender (the From name), the title, the body, the fields as a table and the link as a button; and a footer linking the library's home page and Bookclerk's site. |
| [`render_mjml`](#render_mjml) | function | Compiles MJML to HTML, with no web fonts (nothing is fetched to show the message). |
| [`simple_html`](#simple_html) | function | The plain HTML body, used when the branded template cannot be compiled (for example host HTML that MJML's parser refuses): the content, the fields as a table and the link under its short label, in a minimal document with a language and a title. |
| [`html_body`](#html_body) | function | The HTML body and whether it shows the logo: the branded template, or `simple_html` when it cannot be compiled. |
| [`compose`](#compose) | function | Builds the whole email for one destination. |
| [`LOGO_PNG`](#logo_png) | constant | The Bookclerk mark the branded HTML shows in the header: a 64-pixel PNG drawn at 32, so it stays sharp on dense screens. |
| [`LOGO_CONTENT_ID`](#logo_content_id) | constant | The `Content-ID` the HTML names the logo by (`cid:` URL). |
| [`INVITE_TYPE`](#invite_type) | constant | The notification type the host gives invite mail: its recipient may not know Bookclerk yet, so it carries the introduction. |
| [`INVITED_BY_FIELD`](#invited_by_field) | constant | The field an invite names the Administrator who sent it in. |
| [`INTRO_TITLE`](#intro_title) | constant | The introduction's heading, for recipients who may not know Bookclerk. |
| [`INTRO_TEXT`](#intro_text) | constant | The introduction, in both parts. |
| [`BOOKCLERK_SITE`](#bookclerk_site) | constant | Bookclerk's own site, linked from every footer. |

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
pub fn plain_body(message: &NotificationMessage, sender: &str) -> String
```

The plain-text body: the text, then each field on its own line, then the
link under its short label when the text does not already hold it, then
the footer after a signature separator.

#### Arguments

* `message` - The rendered message.
* `sender` - The From name, the library's name in the link's label.

### worded_for

```rust
pub fn worded_for(message: &NotificationMessage, sender: &str) -> NotificationMessage
```

The message as this sender words it. An invite's subject names the
library by the From name (`Alex has invited you to Riverside Library`)
instead of the host's generic `a Bookclerk library`, and its
`Invited by` field is left out because the subject already names them.
Every other message is returned as it is.

#### Arguments

* `message` - The rendered message.
* `sender` - The From name, which names the library.

### plain_footer

```rust
pub fn plain_footer(message: &NotificationMessage) -> String
```

The footer under every message as plain text: who sent it (the library's
home page when the host gave one) and Bookclerk's site. The HTML footer
says the same words, which spam filters compare.

#### Arguments

* `message` - The rendered message.

### link_host

```rust
pub fn link_host(link: &str) -> Option<&str>
```

The host part of a link (`library.example.com`, with a port when one is
given), for the footer's link to the library's home page; `None` when the
link has no `scheme://host`.

#### Arguments

* `link` - An absolute URL.

### link_label

```rust
pub fn link_label(sender: &str) -> String
```

The label of the link button: short, naming the library it opens
(`Open Riverside Library`), or `Open your library` when the From name
is just the product's. The plain-text part keeps the full URL beside it.

#### Arguments

* `sender` - The From name, which names the library.

### mjml_source

```rust
pub fn mjml_source(message: &NotificationMessage, sender: &str) -> String
```

The MJML source of the branded message: the Bookclerk mark and name as
the header; a card with the sender (the From name), the title, the body,
the fields as a table and the link as a button; and a footer linking the
library's home page and Bookclerk's site.

#### Arguments

* `message` - The rendered message.
* `sender` - The From name, shown at the top of the card.

### render_mjml

```rust
pub fn render_mjml(source: &str) -> Result<String, String>
```

Compiles MJML to HTML, with no web fonts (nothing is fetched to show the
message).

#### Arguments

* `source` - MJML.

#### Errors

The parser's or renderer's message when the source is not valid MJML.

### simple_html

```rust
pub fn simple_html(message: &NotificationMessage, sender: &str) -> String
```

The plain HTML body, used when the branded template cannot be compiled
(for example host HTML that MJML's parser refuses): the content, the
fields as a table and the link under its short label, in a minimal
document with a language and a title.

#### Arguments

* `message` - The rendered message.
* `sender` - The From name, the library's name in the link's label.

### html_body

```rust
pub fn html_body(message: &NotificationMessage, sender: &str) -> (String, bool)
```

The HTML body and whether it shows the logo: the branded template, or
[`simple_html`](#simple_html) when it cannot be compiled.

#### Arguments

* `message` - The rendered message.
* `sender` - The From name, shown at the top of the card.

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

## Constants

### LOGO_PNG

```rust
pub const LOGO_PNG: &[u8] = b"\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR\x00\x00\x00@\x00\x00\x00@\x08\x06\x00\x00\x00\xaaiq\xde\x00\x00\x00\tpHYs\x00\x00\x01\xd8\x00\x00\x01\xd8\x01\xfa\\\xa6r\x00\x00\x00\x19tEXtSoftware\x00www.inkscape.org\x9b\xee<\x1a\x00\x00\x08\xe0IDATx\x9c\xed\x9b{l\xd3\xd7\x15\xc7?\xf7\xe7G\xe2\x9f\xed\xbc\xdf\x86\x18\x08\x8f\xa6\xa3\xa4@IR\n\x01\x1a:\x18\x05\x95jY\xb6\xfeQ\x8dV\xa2\xdb\xbaj\x9b\xd4\xb5\x9b4i[\'\xadRWUZW\xd41\xad\x03m\xd3\xd8\xa00\xca\xa0\x81R\x82\xb4\n\x1a\xca\xa3*\xa5\xd0vP\x12\x08\xe4]\x88\x13\xc7v\xfc\xb8w\x7f\x84B\xc8\xc3\x8e\x1dcC\xe9W\xf2?\xbfs~\xe7\x9c\xfb\xf5\xbd\xe7\x9e\xfb\xf8\t\xa5\x14\xb73\xb4d\x07\x90l\x18\x93\xe9<of\xad\xad\xcf\x1aZ\xa4)*\xa4f\xd8\xd1wh\xcb\xb1D\xc7\x90P\x02\xc4\xccZ\xb3M\x0fU\x82\xa8\x96P-t*\x84\x12F\x05 \xe5\xfd\xc0\x82D\xc6\x03\t \xc0R^3E\x83\xa5\x02\x96Zu\x96)D\x1a\x80\xf8Bn62{z!\r\'/\xd8ot,#!\xee\x04\x0cn0p\xbf\x01\xb2\x07\xcb\r\x9a`\xe6\xe4<\x16\x96\x15SU\xe6\xa4\xfc\xce\t\xa4\x9a\x0c\xacy~\x87-\xde\xb1\x8c\x05\xe3&\xc0V\xf9P\xbeP\xe6*\x94Z\xaa\x04\xcb\x0c\xe0\x1c\xaa\xe3,H\xa7\xaa\xccIU\x99\x93\x85\xb3\x8a\xc9\xb0\xa5\x0e\xb3\xb3\xe6\x1be\xd9\xc3\x1e&\x00\"\xdai0of\xad\xcd\xa3\xcbJ\x06\xfe\xe1\xa5\xc0\x1c\xae\xf5\xe8\x01\x9dL+\x15\xa5\x0e\xaa\xca\x9c,\x993\x99\t\xb9\x91{wH*e\x16\x86b\x8b\xa3\xe2BT\x01\x8d\x13\x11\t\x10\xf3k-v?s\xa5A\xdd\x87RK\x81E\x80i\xb0N\x86-\x95\xf9wM\xa4jV1\x0bg9)qdF\x1d\xc8\x91OZ\x82\xeb\xb6\x1d^\xff\xd6\x91\xb3A\xa9\xc9\x97<\x87\xb6\r#\xa2a\xd5|\x87A\x06j\r~\xb5a\xee\xdbG]Q;\x19\x01a\x87\x80}\xde7k\xacBl\x90\x1av\x06\xf1d1\x1b\x99W\xea\xa0\xbc\xb4\x88\xf2R\x07\xf7\xce\x9c\x88\xc9\x10]I\xd1\xd4\xda\xcd\xa6\xfa\x13\xec;z\x963\x17/\xe3\x0f\x84\x8c\xc0S\x00\x9a\x12\x01\xe0\xd9\xa1\xef\x18T\xf0\x17\n~\x102\t\x13\xf0\xbb\xa8\x1c\x8e\x82\xb0\x04(\xa1\xbd\x0c\xca>Z\xe2\x8a\x06]./\x9b\xebOPw\xe84\x1f\x9f\xeb\xc2\xd3\x1f\xbc*\x13\x02lY:\xf6\x1c\x9d\xde\xcf=\xf4~\xdeg\x1d1\x1e%- \x90\x02=*\xe7a\x10!\t\xaa\xa2\xe5\x15%\xac\xfb\xf1\nl\xba9*\xc3n_\x80\xcd\xfbN\xb0\xeb\xdd\xd3|\xd4\xd8A\xaf\xd7\x7f\x9d<E7a\xcf\xb1\x92\x96\xa3c\xcf\xb1b0\x0e\xf4\xa0\xfe\xbekz\r\xab\xe6;4\x19|\x16\xa5\xfd\xa9\xa2\xee\xd0\xa9aN\x9e\x13\xda\x81#w\xa5/\xd8\xf5\xe1\xe5\xa8\x82\x1b\x84\x88\xb3\xc0\xea\x05\xa5cj|0$y\xb3\xe14[\xff{\x8a\x0f\xfe\xd7F\xa7\xcb\x0b\x83\xc6\x8d\xd1l\xc4\x96e!-G\'-\xcf\x8a9\xd54\xba\xb1+\x10*\xb0\x06\xf8\x11Z\xd0\x02<1T~\xf8\xc8\xbcu&\xa5\xd6\x1e^UQU\xbe\xf3\xbd\x86\x88\x06G@D\x02\x8e\x9fic\xf5\xc2\x19#\xca\x1aN^`\xd3\xbe\x13\xbc{\xa2\x99\x96\xcf\xdd\x0cN\xa8\x06\x83@\xcf\xbc\xf6\x0f\xeb\xf6\x94!s\xc5\x18\xa0\xae$[\xc5\x88l)\xa9J\x11\x18\xa5T3\x80\x1bC\xc0\x86\xba\x0fXu\xdft\xe6L/\xe4dc\'\x7f\xdf{\x9cw\x8e\x9f\xe7\\\xbb\x8bPH^\xd5\x13\x80\x9e\x96r\xb5[\xdb\xb2t\x84\x16m\x8b\x13\x8f\x88\x04\xf4\x07\x82\xac\xfc\xf9?\xd1\x84 8\xa8\xc10\xfa8\xbe\x95\x10\x96\x00M\x13H\xa9\x06~(L\xa9F\xd2\xb2\x07\x1ak\xcf\xb1bJ\x89n&\xb8\x19\x11\x96\x80\xec\xe2\x0c:\x9b\xae%\xd8I\xb3\n\xb0\xe7\x8c8C\xdd\xb2\x08\xdbg\xf3\x9c\x99d\xe4\'e\x8d\x02\x80vu\x1a\x11#\x97\xab\x82\x81bB\x11\x18\x87\x8f\xd1\xe1\xeap3ivQ\xd2H\x90\xa8=\xc0\x87J\x89\xed#\xc9\x95\xe2\xd7\x08\xf1\x92@\xdb\x19\xab\x8f\xb0C\xc0\xd5\xd6KZ\xaeN^I6\xdd\xed\xeeX}\xc4\x8c\xca7\x8f\xbe\x07\x94\x8d*\xaf;r\x1088\x1e\x1fa{@\xe1\x1d\xb94}\xd0J\x7fo\xffx|\xc4\x0fBH\x00\xa1T(^&\xc3\xaf\x05B\x92\x893\x0b8s\xb89^\xfe\xc6\x87\x90\xda\x88&\xb2\xa5\x91\xd7\xe3e2,\x01-\x9fv1\xed\xdeb&\xcfq\xdc\x14$T\xec>z\x008\x10O\x9ba\x87@\x9f\xcb\xc7\xf9\x13mh\x86\x9b\xbf\xa2\x8b\x15\x11+\xc1K\x17{\xf0\xb8|Q\x1b\x9eU\x98\xcd\xb4\xdct\xb4(\xcb\xe1K\xf9y\xe4\xea\xa9\xb3{[\xde\xfd\xd9h:>\xb7\xe7`\xee\xf4\xa5q\xe9\tc\xda\x13\xf4\xb9\xfd\x91\x95\x06a\xcd\xbc;\xa8\x9d=5\xa6\x80\xae\xe0\xde+\xbf\x11\xa1\x82\xee`\xe7\xe9\xbd\xd5\xb9\xd3\xbe\xfe\xcex\x9c\xc0\r8\x19\xca\xd4S\xa9\xb9\xbb\xe4\xbag\xde\xa6sq\xf5!e\xc8\xa8\x02\x9e\xfa\xce\xd3{\xab\xc6k+\xee\x048\xd2u4q}\xb7\xef\xfdp\xf8^\xc6x\x11/\x12\xe2N\x80\x16\xf5\xa2?v\x0c\x90\xe0\xad\xefnz{Q\xac6n\xbd\xf5\xeb\x10H\x194\xfa=\x9e}\xb1\x92pK\x12 \x86\x0c\xb1\x01\x12\xfa\xf6\xc52\x1cnI\x02\x8cf\xcb\xb0g\xb1\xe6\x84\xa4\x1e\x8f\xc7\n\xb35\x1da0\xa2dp\xa8\xc8\xa8\t\xde\xb8|\xf9\xd8\xe4\xcc\xcc\xb9c:8I\x18\x01\x81\x90\xa4\xf6W\xaf\xe3\xf1\xc6\xbct\x07\xc0h\xd0\xf8\xe3\xd3\x0fR\x9c\x9f>\x9aJf\xb0?\x90\t\xdc\\\x04\x98\x0c\x1a\xcf\xaf\xad\xc6\x1f\x1c\xdfBN\x00\x8e\xdc\xb4\xf8\x04E\x82\x87@\xa93\'\x91\xee\xc6\x84[2\t\xc6\x13\xb7=\x01\tM\x82\xcf\xbc\xba\x97\xbe\x18\x93\xa0\xa6i<\xf9\xf0=\x94\x95\xe4\xc75\xae\x84\x11`\xd4\x04\xcb\xcb\xa7\x11\x08\xc5\x9e\x04\xf3M\x12\xf7\xc9OPR\x92RT\x809;k\xfcq\x8d\xdb\xc2\x18!\x84`yEId\xc5\x11\xe0k\xbeH\xdb\x96\xedx\xbc>\xa4\xd3\x01\xc2@\xe7\xae=\x08!\xc8\xafy\x08}\xea\x94\x98\xe3\xba\xe9\x0b!\xf7\xc9\x8fi\xdb\xbc\x9d\xa2G\xbf\x8d>\xad\x84`O/JJL\x19\xe9x\xcf7\xd3\xb2q\x139+\x1e }\xde\x9c\x98\xec\xdf\xd4\x04\x04]=\xb4n\xda\xca\xa4\xa7\x9f\"\xd8\xd3\xc3g\xbfy\x01!4\xd04\xa4\xbf\x9f\xc2\xef\xd40\xe9\xa7O\xd1\xf8\xc2\xcbX&N\xc0\\\x90\x17\xb5\x8f\x84\x11\x10\x92\x8a\xd7v\xbdO (#\xeaj\x9a\xe0\xb1\x15w\xd3\xb3w?9\xcb\xaaQ\xa1\x10\x17\xfe\xfc7&<\xf1],\xceb\x00\xfa\xdb\xdai~\xf5/L\xfc\xfe\xe3\xe4?\xbc\x92\xce\xba\xb7p<\xfeh\xd4q\x85%\xc0h2\x10\x0c\xc4m\x0b\x1e\xaf?\x80\xdb\x13y{-\xd5<p\x1d\xa0\xf7\xa3SLY\xb9\x8c\xd6-o\x90\xb7\xfa\xc1\xab\x8d\x07H)\xc8\xc7\xf9\xf4\x0f1\xa5\xd91\xe7\xe7\xd2\xb6\xf9\xdf1\xc5\x14\x96\x80\xc2i\xd94\x7f\xdc1\xf8\xa2G\xcc0h\x82\x9f\xd4TF\xf5\x8e\xf4\xfb\xd1,\x16\xd2\xe6\x94a\x9d:y\x98\xdc\x94>\xb0\x1e\x10\x06\xd0,\x16\xa4\xd7\x87f\x19~\x071\x1c\xc2\x16B\xb6,\x9d\xa2\xe9I._\x95\xc4~\xd7\x9dh\x96\xe1K\xe0@\xb7\x8b\x8b\x7f\xdd\x04@\xa8\xdf\x870Gw\x8f\t\"\x10\xd0u\xc1\x15((\xc9\xa60I$\xa4Np\xe09{\x0e\xdf\xc5\x16\x9a\xd7o\x04u}\xfe\xe8\xfc\xcfnR\x8a\n\xf0wtb\xb4\xdb\x11Q^\xd5\x83\x08\x04t6]\xfe\xac\xbb\xb5\x97\xf4\xbc\xb1\x9f\x0e\xc7\xf3\xf3\x8b\xac\xc5\x0b\xe8\xdc\xb9\x9b\xd4\xa2B\x8cv+\x8d/\xbeB\xf7\x81Ct7\x1c\xe6\xfc\x1f\xd6#}>\xb2\xab\x97\xd0\xb1\xa3\x8e\xac%\xb1\xed\x8d\x86\xcd\x01\x02\xba\x1b\x8f\xb7\x92\xeb\x0c\x7f\xf3\xb3\xdf\x13\xa0\xb7\xab\x8f\x9e.\x0fz\xbe\x0fV\xc5\x14\xcb0\xd8\xbeV\x8a\xeb\xc8\xfb\xb4\xfek\x1b\x05\x8f|\x0b\xefggq\x9f\xfa\x14%CdU/\xc6v\xe7\x0c:v\xeeA\xf6\xfb\xc9\xa8\xb8\x01u\x80\x82\x03HU\xd9\xd1x\xe9\xba\xe7A\x7f\x08\xf7%\x0f=]\x1ez:\xfb\xf0\x0f\xaa\xef/\x0e\\\xff\x8f\xdb\xd6p\xd1\xa3\x8f\xd0\xbeu\x07\x8d\xbf}\x91\xcc\xc5\x0bH\x9b[\x86\x10\x06\xbc\xe7\x9bi|\xe1\xf7\x98\x0b\xf2\x98\xf0\xbd\xc7@\\\xed\xcc\xc1\x8c\xa0j\x1f\xab\xfd\xb0w\x85\xf5{j\x8b5M\x9e\x06\xcc\x00\xe9y6\xfc\xde\x00\xde!\xc7\xe5\x02\xce*\xc1A\xa58\x90jM\xad;\xf7\x8f\'7\x80x\xe0\x0b\xb9\xbf\xbd\x03s~\xf4E\xca`\xf4\xb7\xb6\xe1z\xef\x18\xbe\x0b-(%I)*$}\xdel,\xc5\x13\x87h\xaa\xd7lE\xf3\xd7\x8e\xd5n\xc4\xcb\xd2\xf6\xf2\x9a\xe7\x14\xfcr\xc8k\xed(\xb5_i\xaa^j\xb2\xde\xdb\xb0\xbd\xe9:q\xeb1\xdd\xad\x02\xcf(T\xa5HP\xb1%\xc0\x87\x12\xfb\xad\x0e\xdf+\xb0x\xd8f\xe1\xa8\xefE\xbc-^[k\xb05\xcauJ\xe0\x00\xea\x85\x92\xf5\xee\xa3\xdbO\xaa/\xc9\xe7fQ\x7f/\xf0e\xc3m\xbf#\xf4\x15\x01\xc9\x0e \xd9\xf8\x8a\x80d\x07\x90l|E@\xb2\x03H6n{\x02\xfe\x0f\x86\x12 A\xa0s!\x17\x00\x00\x00\x00IEND\xaeB`\x82";
```

The Bookclerk mark the branded HTML shows in the header: a
64-pixel PNG drawn at 32, so it stays sharp on dense screens.

### LOGO_CONTENT_ID

```rust
pub const LOGO_CONTENT_ID: &str = "mark@bookclerk";
```

The `Content-ID` the HTML names the logo by (`cid:` URL).

### INVITE_TYPE

```rust
pub const INVITE_TYPE: &str = "account_invite";
```

The notification type the host gives invite mail: its recipient may not
know Bookclerk yet, so it carries the introduction.

### INVITED_BY_FIELD

```rust
pub const INVITED_BY_FIELD: &str = "Invited by";
```

The field an invite names the Administrator who sent it in.

### INTRO_TITLE

```rust
pub const INTRO_TITLE: &str = "New to Bookclerk?";
```

The introduction's heading, for recipients who may not know Bookclerk.

### INTRO_TEXT

```rust
pub const INTRO_TEXT: &str = "Bookclerk is a home for audiobooks. It gathers the books bought from multiple audiobook storefronts into one library that runs on its owner's own computer. With your sign-in you can browse the shelves, listen in your browser and keep your own wishlist. It's free and open source.";
```

The introduction, in both parts.

### BOOKCLERK_SITE

```rust
pub const BOOKCLERK_SITE: &str = "https://bookclerk.app";
```

Bookclerk's own site, linked from every footer.
