# bookclerk_library::notifications

Notifications: the router's rules and the shapes the library stores
(`docs/adr/notifications.md`, `docs/server/notifications.md`).

The host decides what is worth a notification and who gets it; a
notifier plugin only sends one message to one destination. This module
is pure. The library database holds the destinations, the
administrators' routes and offers, each patron's choices, every
accepted notification, the delivery log and the inbox
(`LibraryStore::route_notification` and its neighbours), and
`bookclerkd` runs one `notify_send` job per delivery-log row.

Routing, for a notification of type `T`:

- **Operational** (`T` is for administrators): every channel destination
  with a route for `T` whose minimum severity the notification reaches.
- **Patron** (`T` is for one patron): the patron's inbox, always. Then,
  when the patron chose for `T`, the personal destinations they picked
  (when on); when they did not, every personal destination of theirs on
  an instance the administrators offer for `T` with `default_on`. Either
  way only instances the administrators offer for `T` are used.
- **Account** is the host's own mail. It is never routed: each message
  goes by itself, through the instance administrators chose for it
  (`store::account_mail`).

A paused destination is skipped.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AccountMailSettings`](#accountmailsettings) | struct | The notifier instance administrators chose for account mail. |
| [`AccountMail`](#accountmail) | struct | One account-mail message and where its delivery stands. |
| [`MintedToken`](#mintedtoken) | struct | The link token an account-mail send mints: the raw value goes into the message once and only its hash is stored. |
| [`VerifyEmailToken`](#verifyemailtoken) | struct | The token of a verification link, as a patron's browser sends it back: exactly `VERIFY_TOKEN_CHARS` lowercase hex characters. |
| [`AccountMailSettingsRequest`](#accountmailsettingsrequest) | struct | The body of `PUT /api/notifications/account-mail`, read from a JSON object only, before `AccountMailSettingsRequest::validate` checks it. |
| [`VerifyEmailRequest`](#verifyemailrequest) | struct | The body of `POST /api/auth/verify-email`, read from a JSON object only, before `VerifyEmailRequest::validate` checks it. |
| [`HostNotificationType`](#hostnotificationtype) | struct | A notification type the host itself raises. |
| [`NewNotification`](#newnotification) | struct | A notification for the router to accept: rendered once, copied to every destination. |
| [`PendingDelivery`](#pendingdelivery) | struct | One accepted notification's delivery waiting for its send job. |
| [`StalledSend`](#stalledsend) | struct | A send job that has been ready to run for a long time and that nothing claimed. It is stalled only when its notifier instance runs on no live node; otherwise it is waiting for a worker. |
| [`Routed`](#routed) | struct | What the router did with a notification. |
| [`Destination`](#destination) | struct | A destination as stored. |
| [`NewDestination`](#newdestination) | struct | A destination to create. |
| [`Route`](#route) | struct | One operational route. |
| [`PatronOffer`](#patronoffer) | struct | One instance offered to patrons for a patron type. |
| [`PatronChoice`](#patronchoice) | struct | One patron's choice for one patron type. |
| [`StoredNotification`](#storednotification) | struct | A stored notification. |
| [`SendWork`](#sendwork) | struct | Everything one `notify_send` job needs. |
| [`DeliveryRecord`](#deliveryrecord) | struct | One delivery-log row, joined with what a reader needs to show it. |
| [`InboxEntry`](#inboxentry) | struct | One inbox entry. |
| [`DestinationKind`](#destinationkind) | enum | Whether a destination is a library channel or one patron's address. |
| [`DeliveryState`](#deliverystate) | enum | Where one delivery stands. |
| [`AccountMailKind`](#accountmailkind) | enum | What an account-mail message says. |
| [`StallOutcome`](#stalloutcome) | enum | What failing one `StalledSend` did. |
| [`FailureEffect`](#failureeffect) | enum | What a finished send attempt did to its destination. |
| [`FailedAttempt`](#failedattempt) | enum | Whether recording a failed delivery spends one more attempt. |
| [`parse_account_mail_settings`](#parse_account_mail_settings) | function | Parses the body of `PUT /api/notifications/account-mail` from bytes (sans-IO), for the fuzz target and tests. The route reads the same `AccountMailSettingsRequest` with the JSON extractor and calls the same validation. |
| [`parse_verify_email_request`](#parse_verify_email_request) | function | Parses the body of `POST /api/auth/verify-email` from bytes (sans-IO), for the fuzz target and tests. The route reads the same `VerifyEmailRequest` with the JSON extractor and calls the same validation. |
| [`host_notification_type`](#host_notification_type) | function | The host type named `id`, if it is one. |
| [`qualified_plugin_type`](#qualified_plugin_type) | function | The routed id of a plugin's type: `<plugin id>:<type id>`. |
| [`fields_text`](#fields_text) | function | A plain-text body from labelled fields: one `name: value` line each, cut at a character boundary (ending in `…`) when it would pass `maxNotificationTextBytes`, so a raise within the field limits is never refused for its rendered body; the fields themselves are kept whole. |
| [`link_for`](#link_for) | function | The link a notifier is sent: the library's public origin and the path. `None` when either is missing, or when the result is not a link a notifier may be sent (an `http://` origin that is not loopback, say), so the message goes without a link rather than not at all. |
| [`severity_reaches`](#severity_reaches) | function | Whether a route with minimum severity `min` takes a notification of `severity`. |
| [`PAUSE_AFTER_FAILURES`](#pause_after_failures) | constant | Permanent send failures in a row after which a destination is paused. A delivery clears the count; an administrator or the patron resumes a paused destination by hand. |
| [`MAX_ADDRESS_BYTES`](#max_address_bytes) | constant | Longest stored destination address, in bytes of its JSON text. A database whose cell limit is smaller lowers it further. 1 KiB keeps the insert inside the smallest statement payload any adapter advertises. |
| [`MAX_SEND_ATTEMPTS`](#max_send_attempts) | constant | Send attempts one delivery may make, retries included. A delivery whose notifier still asks to retry after this many fails. |
| [`NOTIFICATION_RETENTION_DAYS`](#notification_retention_days) | constant | Days a notification, its delivery log and its inbox rows are kept. |
| [`MAX_DESTINATION_LABEL_BYTES`](#max_destination_label_bytes) | constant | Longest destination label, in bytes. |
| [`MAX_ROUTES_PER_TYPE`](#max_routes_per_type) | constant | Most routes, offers or patron targets one notification type may have. |
| [`IN_APP_DESTINATION`](#in_app_destination) | constant | The reserved destination id of the built-in "In app" target: an operational route to it puts the notification in the inbox of every active Owner and Administrator, where the web app's bell, browser notifications and the desktop window's system notifications show it. It is not a stored destination and runs no plugin. |
| [`TEST_NOTIFICATION_TYPE`](#test_notification_type) | constant | The host's built-in `test` type: an administrator or a patron sends it to one destination to check that it works. |
| [`ACCOUNT_MAIL_RETENTION_DAYS`](#account_mail_retention_days) | constant | How long a delivered or failed account-mail record is kept. |
| [`EMAIL_VERIFICATION_HOURS`](#email_verification_hours) | constant | How long an email verification link works. |
| [`INVITE_DAYS`](#invite_days) | constant | How long an emailed invite works, as for every claim ticket. |
| [`VERIFY_TOKEN_CHARS`](#verify_token_chars) | constant | Length of a verification link's token: 32 random bytes as lowercase hex. |
| [`MAX_INSTANCE_ID_BYTES`](#max_instance_id_bytes) | constant | Longest plugin instance id the account-mail settings body takes. |
| [`AUDIENCES`](#audiences) | constant | Every audience, in the order of the `audience` column's CHECK list. |
| [`SEVERITIES`](#severities) | constant | Every severity, lowest first, in the order of the CHECK lists. |
| [`ALERT_ACTIONS`](#alert_actions) | constant | Every alert action, in the order of the `alert_action` CHECK list. |
| [`HOST_NOTIFICATION_TYPES`](#host_notification_types) | constant | The host's own notification types. Producers of operational and patron events add theirs here as they land. |

## Structs

### AccountMailSettings

```rust
pub struct AccountMailSettings {
    pub plugin_instance_id: String,
    pub tested_at: Option<DateTime<Utc>>,
}
```

The notifier instance administrators chose for account mail.

**Fields**

- `plugin_instance_id`: The instance that sends account mail.
- `tested_at`: When a test message through it was last delivered; `None` until one
  is, and again after the instance changes.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AccountMail

```rust
pub struct AccountMail {
    pub id: String,
    pub kind: AccountMailKind,
    pub user_id: i64,
    pub address: String,
    pub plugin_instance_id: String,
    pub state: DeliveryState,
    pub attempts: i64,
}
```

One account-mail message and where its delivery stands.

**Fields**

- `id`: Message id.
- `kind`: What it says.
- `user_id`: The user it is for.
- `address`: The address it goes to.
- `plugin_instance_id`: The notifier instance that sends it.
- `state`: Where it stands.
- `attempts`: Send attempts made.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### MintedToken

```rust
pub struct MintedToken(/* private */);
```

The link token an account-mail send mints: the raw value goes into the
message once and only its hash is stored.

#### MintedToken::expose

```rust
pub fn expose(&self) -> &str
```

The raw token, for the link in the message. Never logged.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### VerifyEmailToken

```rust
pub struct VerifyEmailToken(/* private */);
```

The token of a verification link, as a patron's browser sends it back:
exactly [`VERIFY_TOKEN_CHARS`](#verify_token_chars) lowercase hex characters.

#### VerifyEmailToken::expose

```rust
pub fn expose(&self) -> &str
```

The raw token, to hash and look up. Never logged.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AccountMailSettingsRequest

```rust
pub struct AccountMailSettingsRequest(/* private */);
```

The body of `PUT /api/notifications/account-mail`, read from a JSON
object only, before [`AccountMailSettingsRequest::validate`](#accountmailsettingsrequestvalidate) checks it.

The route reads it with the web framework's JSON extractor and
[`parse_account_mail_settings`](#parse_account_mail_settings) reads it from bytes; both go through
this type's reader and the same validation.

#### AccountMailSettingsRequest::validate

```rust
pub fn validate(self) -> Result<Option<String>>
```

Checks the instance id.

##### Returns

The instance id: 1 to [`MAX_INSTANCE_ID_BYTES`](#max_instance_id_bytes) printable ASCII
bytes with no space, or `None` for `null`.

##### Errors

[`crate::LibraryError::Invalid`](README.md#libraryerror) when the id breaks those rules.

Implements: `Debug`, `Deserialize<'de>`

### VerifyEmailRequest

```rust
pub struct VerifyEmailRequest(/* private */);
```

The body of `POST /api/auth/verify-email`, read from a JSON object
only, before [`VerifyEmailRequest::validate`](#verifyemailrequestvalidate) checks it.

The route reads it with the web framework's JSON extractor and
[`parse_verify_email_request`](#parse_verify_email_request) reads it from bytes; both go through
this type's reader and the same validation.

#### VerifyEmailRequest::validate

```rust
pub fn validate(self) -> Result<VerifyEmailToken>
```

Checks the token.

##### Returns

The token: exactly [`VERIFY_TOKEN_CHARS`](#verify_token_chars) lowercase hex characters.

##### Errors

[`crate::LibraryError::Invalid`](README.md#libraryerror) when the token breaks that rule.

Implements: `Debug`, `Deserialize<'de>`

### HostNotificationType

```rust
pub struct HostNotificationType {
    pub id: &'static str,
    pub audience: NotificationAudience,
    pub severity: NotificationSeverity,
    pub title: &'static str,
}
```

A notification type the host itself raises.

**Fields**

- `id`: Type id; host types carry no plugin prefix.
- `audience`: Who it is for.
- `severity`: Default severity.
- `title`: Title shown in the routing matrix and on each notification.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### NewNotification

```rust
pub struct NewNotification {
    pub notification_type: String,
    pub audience: NotificationAudience,
    pub severity: NotificationSeverity,
    pub title: String,
    pub text: String,
    pub link_path: Option<String>,
    pub fields: Vec<NotificationField>,
    pub group_key: Option<String>,
    pub alert_action: NotificationAlertAction,
    pub alert_key: Option<String>,
    pub source: String,
    pub patron: Option<i64>,
}
```

A notification for the router to accept: rendered once, copied to every
destination.

**Fields**

- `notification_type`: Qualified type (`test`, `<plugin>:<type>`).
- `audience`: Who it is for; decides how it is routed.
- `severity`: How urgent it is.
- `title`: Short title.
- `text`: Plain-text body.
- `link_path`: Path inside the library to link to; the send job prefixes the
  public origin.
- `fields`: Labelled values.
- `group_key`: Grouping key.
- `alert_action`: Alert lifecycle step.
- `alert_key`: Alert key; required unless `alert_action` is `none`.
- `source`: Empty for a host type, else the raising plugin's id.
- `patron`: Patron a patron-audience notification is for; required for that
  audience and refused for the others.

#### NewNotification::check

```rust
pub fn check(&self, notification_id: &str) -> Result<(), NotificationError>
```

Checks the notification the way every send will be checked, so a bad
one is refused before anything is stored.

##### Arguments

* `notification_id` - The id it will be stored under.

##### Errors

The first [`NotificationError`](../bookclerk_plugin_abi/notifications.md#notificationerror) the rendered message or its link path
breaks; `NotificationError::BadId` for a patron that does not match
the audience.

#### NewNotification::message

```rust
pub fn message(&self, notification_id: &str, link: Option<String>) -> NotificationMessage
```

The message a notifier is sent, with `link` already built from the
public origin.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PendingDelivery

```rust
pub struct PendingDelivery {
    pub id: String,
    pub plugin_instance_id: String,
}
```

One accepted notification's delivery waiting for its send job.

**Fields**

- `id`: Delivery-log row id.
- `plugin_instance_id`: Notifier instance that sends it.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StalledSend

```rust
pub struct StalledSend {
    pub job_id: String,
    pub run_after: String,
    pub lease_generation: i64,
    pub delivery_id: Option<String>,
    pub account_mail_id: Option<String>,
    pub notifier: Option<String>,
}
```

A send job that has been ready to run for a long time and that nothing
claimed. It is stalled only when its notifier instance runs on no live
node; otherwise it is waiting for a worker.

**Fields**

- `job_id`: The pending send job nothing claimed.
- `run_after`: When the job became ready to run, as stored: the write that fails
  it matches this, so a job retried in the meantime is left alone.
- `lease_generation`: The job's lease generation when it was read, matched the same way.
- `delivery_id`: Its pending delivery, named by the job's dedup key; `None` when the
  delivery is gone, already finished or linked to another job, and
  the job only needs cancelling.
- `account_mail_id`: Its pending account mail, when the job sends one instead of a
  delivery; `None` when the mail is gone, finished or linked to
  another job.
- `notifier`: The notifier instance the delivery or account mail goes through;
  `None` exactly when both of those are.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Routed

```rust
pub struct Routed {
    pub notification_id: String,
    pub deliveries: Vec<PendingDelivery>,
    pub inboxed: bool,
}
```

What the router did with a notification.

**Fields**

- `notification_id`: The id it was stored under.
- `deliveries`: The deliveries it made, each with a send job enqueued (or left for
  the sweep when the enqueue failed).
- `inboxed`: Whether it reached an inbox: its patron's, or the staff's through an
  "In app" route.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Destination

```rust
pub struct Destination {
    pub id: String,
    pub plugin_instance_id: String,
    pub kind: DestinationKind,
    pub user_id: Option<i64>,
    pub label: String,
    pub address_json: String,
    pub failure_count: i64,
    pub paused_at: Option<String>,
    pub paused_reason: Option<String>,
    pub created_at: String,
}
```

A destination as stored.

**Fields**

- `id`: Destination id.
- `plugin_instance_id`: Notifier instance that sends to it.
- `kind`: Channel or personal.
- `user_id`: Owner of a personal destination.
- `label`: Name shown to people.
- `address_json`: Canonical address document (empty for a channel). May hold secrets.
- `failure_count`: Permanent failures since the last delivery.
- `paused_at`: When it was paused; `None` while active.
- `paused_reason`: Why it was paused.
- `created_at`: RFC 3339 creation time.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NewDestination

```rust
pub struct NewDestination {
    pub plugin_instance_id: String,
    pub kind: DestinationKind,
    pub user_id: Option<i64>,
    pub label: String,
    pub address_json: String,
}
```

A destination to create.

**Fields**

- `plugin_instance_id`: Notifier instance that will send to it.
- `kind`: Channel or personal.
- `user_id`: Owner; required for a personal destination, refused for a channel.
- `label`: Name shown to people; one line, at most
  [`MAX_DESTINATION_LABEL_BYTES`](#max_destination_label_bytes).
- `address_json`: Canonical address document the notifier validated (empty for a
  channel).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Route

```rust
pub struct Route {
    pub destination_id: String,
    pub min_severity: NotificationSeverity,
}
```

One operational route.

**Fields**

- `destination_id`: Channel destination, or [`IN_APP_DESTINATION`](#in_app_destination) for the staff inbox.
- `min_severity`: Lowest severity sent.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PatronOffer

```rust
pub struct PatronOffer {
    pub plugin_instance_id: String,
    pub default_on: bool,
}
```

One instance offered to patrons for a patron type.

**Fields**

- `plugin_instance_id`: Notifier instance.
- `default_on`: Whether a patron who has not chosen gets the type there.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### PatronChoice

```rust
pub struct PatronChoice {
    pub notification_type: String,
    pub enabled: bool,
    pub destination_ids: Vec<String>,
}
```

One patron's choice for one patron type.

**Fields**

- `notification_type`: Qualified patron type.
- `enabled`: Whether the patron wants it outside the inbox.
- `destination_ids`: The patron's personal destinations it goes to.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### StoredNotification

```rust
pub struct StoredNotification {
    pub id: String,
    pub notification: NewNotification,
    pub created_at: String,
}
```

A stored notification.

**Fields**

- `id`: Notification id.
- `notification`: The notification as accepted.
- `created_at`: RFC 3339 creation time.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### SendWork

```rust
pub struct SendWork {
    pub delivery_id: String,
    pub state: DeliveryState,
    pub attempts: i64,
    pub plugin_instance_id: String,
    pub notification: StoredNotification,
    pub destination: Option<Destination>,
}
```

Everything one `notify_send` job needs.

**Fields**

- `delivery_id`: Delivery-log row id.
- `state`: Where the delivery stands.
- `attempts`: Attempts made so far.
- `plugin_instance_id`: Notifier instance that sends it.
- `notification`: The notification.
- `destination`: The destination; `None` when it was removed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### DeliveryRecord

```rust
pub struct DeliveryRecord {
    pub id: String,
    pub notification_id: String,
    pub notification_type: String,
    pub title: String,
    pub destination_id: String,
    pub plugin_instance_id: String,
    pub state: DeliveryState,
    pub attempts: i64,
    pub provider_id: Option<String>,
    pub reason: Option<String>,
    pub created_at: String,
    pub updated_at: String,
}
```

One delivery-log row, joined with what a reader needs to show it.

**Fields**

- `id`: Delivery-log row id.
- `notification_id`: Notification id.
- `notification_type`: Qualified type.
- `title`: Notification title.
- `destination_id`: Destination id.
- `plugin_instance_id`: Notifier instance.
- `state`: Where it stands.
- `attempts`: Attempts made.
- `provider_id`: The service's id for the delivered message.
- `reason`: Last failure or retry reason.
- `created_at`: RFC 3339 creation time.
- `updated_at`: RFC 3339 time of the last change.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### InboxEntry

```rust
pub struct InboxEntry {
    pub notification: StoredNotification,
    pub read_at: Option<String>,
}
```

One inbox entry.

**Fields**

- `notification`: The notification.
- `read_at`: When the patron read it; `None` while unread.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### DestinationKind

```rust
pub enum DestinationKind {
    Channel,
    Personal,
}
```

Whether a destination is a library channel or one patron's address.

**Variants**

- `Channel`: A library channel; the notifier instance's own settings say where.
- `Personal`: One patron's personal address.

#### DestinationKind::ALL

```rust
pub const ALL: [Self; 2] = _
```

Every kind, in the order of the `kind` column's CHECK list.

#### DestinationKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### DestinationKind::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the stored spelling; `None` when unknown.

#### DestinationKind::wire

```rust
pub fn wire(self) -> NotifyDestinationKind
```

The kind the notifier sees.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### DeliveryState

```rust
pub enum DeliveryState {
    Pending,
    Delivered,
    Failed,
    Dropped,
}
```

Where one delivery stands.

**Variants**

- `Pending`: Waiting for its send job, or between retries.
- `Delivered`: The notifier delivered it.
- `Failed`: The notifier refused it for good, or it ran out of attempts.
- `Dropped`: Not sent: its destination was paused or removed, or its notifier
  instance is gone.

#### DeliveryState::ALL

```rust
pub const ALL: [Self; 4] = _
```

Every state, in the order of the `state` column's CHECK list.

#### DeliveryState::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### DeliveryState::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the stored spelling; `None` when unknown.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### AccountMailKind

```rust
pub enum AccountMailKind {
    Test,
    Invite,
    VerifyEmail,
}
```

What an account-mail message says.

**Variants**

- `Test`: A test message from the account-mail settings; delivering it marks
  the instance tested.
- `Invite`: An invite: a link that sets up the account (a claim ticket).
- `VerifyEmail`: A link that proves the user's email address.

#### AccountMailKind::ALL

```rust
pub const ALL: [Self; 3] = _
```

Every kind, in the order of the `kind` column's CHECK list.

#### AccountMailKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

The stored spelling.

#### AccountMailKind::parse

```rust
pub fn parse(s: &str) -> Option<Self>
```

Parses the stored spelling; `None` when unknown.

Implements: `Clone`, `ComposeSchema`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`, `ToSchema`

### StallOutcome

```rust
pub enum StallOutcome {
    NotStalled,
    Cancelled,
    Dropped,
    Failed(FailureEffect),
}
```

What failing one [`StalledSend`](#stalledsend) did.

**Variants**

- `NotStalled`: The job was claimed, retried or ended meanwhile; nothing changed.
- `Cancelled`: The job was cancelled; it had no pending delivery to fail.
- `Dropped`: The job was cancelled and its delivery dropped, because the
  destination was already paused; nothing was counted.
- `Failed`: The job was cancelled and its delivery failed, in one write.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### FailureEffect

```rust
pub enum FailureEffect {
    Counted,
    Paused,
}
```

What a finished send attempt did to its destination.

**Variants**

- `Counted`: The destination stays active.
- `Paused`: This failure paused the destination.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### FailedAttempt

```rust
pub enum FailedAttempt {
    New,
    AlreadyCounted,
}
```

Whether recording a failed delivery spends one more attempt.

**Variants**

- `New`: A send the notifier refused for good: this answer is a new attempt.
- `AlreadyCounted`: A delivery that ran out of attempts: its last answer (a retry) was
  already counted when it was recorded.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_account_mail_settings

```rust
pub fn parse_account_mail_settings(body: &[u8]) -> Result<Option<String>>
```

Parses the body of `PUT /api/notifications/account-mail`
from bytes (sans-IO), for the fuzz target and tests. The route reads
the same [`AccountMailSettingsRequest`](#accountmailsettingsrequest) with the JSON extractor and
calls the same validation.

The body is one JSON object whose only key is `plugin_instance_id`: a
string of 1 to [`MAX_INSTANCE_ID_BYTES`](#max_instance_id_bytes) printable ASCII bytes with no
space, or `null` to turn account mail off. The key is required and may
not repeat.

#### Returns

The instance id, or `None` for `null`.

#### Errors

[`crate::LibraryError::Invalid`](README.md#libraryerror) naming the first rule the body breaks.

### parse_verify_email_request

```rust
pub fn parse_verify_email_request(body: &[u8]) -> Result<VerifyEmailToken>
```

Parses the body of `POST /api/auth/verify-email` from bytes (sans-IO),
for the fuzz target and tests. The route reads the same
[`VerifyEmailRequest`](#verifyemailrequest) with the JSON extractor and calls the same
validation.

The body is one JSON object whose only key is `token`: exactly
[`VERIFY_TOKEN_CHARS`](#verify_token_chars) lowercase hex characters. The key is required
and may not repeat.

#### Errors

[`crate::LibraryError::Invalid`](README.md#libraryerror) naming the first rule the body breaks.

### host_notification_type

```rust
pub fn host_notification_type(id: &str) -> Option<&'static HostNotificationType>
```

The host type named `id`, if it is one.

### qualified_plugin_type

```rust
pub fn qualified_plugin_type(plugin_id: &str, type_id: &str) -> String
```

The routed id of a plugin's type: `<plugin id>:<type id>`.

### fields_text

```rust
pub fn fields_text(fields: &[NotificationField]) -> String
```

A plain-text body from labelled fields: one `name: value` line each,
cut at a character boundary (ending in `…`) when it would pass
`maxNotificationTextBytes`, so a raise within the field limits is never
refused for its rendered body; the fields themselves are kept whole.

### link_for

```rust
pub fn link_for(public_origin: Option<&str>, path: Option<&str>) -> Option<String>
```

The link a notifier is sent: the library's public origin and the path.
`None` when either is missing, or when the result is not a link a
notifier may be sent (an `http://` origin that is not loopback, say), so
the message goes without a link rather than not at all.

### severity_reaches

```rust
pub fn severity_reaches(severity: NotificationSeverity, min: NotificationSeverity) -> bool
```

Whether a route with minimum severity `min` takes a notification of
`severity`.

## Constants

### PAUSE_AFTER_FAILURES

```rust
pub const PAUSE_AFTER_FAILURES: i64 = 5;
```

Permanent send failures in a row after which a destination is paused.
A delivery clears the count; an administrator or the patron resumes a
paused destination by hand.

### MAX_ADDRESS_BYTES

```rust
pub const MAX_ADDRESS_BYTES: usize = 1024;
```

Longest stored destination address, in bytes of its JSON text. A
database whose cell limit is smaller lowers it further. 1 KiB keeps the
insert inside the smallest statement payload any adapter advertises.

### MAX_SEND_ATTEMPTS

```rust
pub const MAX_SEND_ATTEMPTS: i64 = 10;
```

Send attempts one delivery may make, retries included. A delivery whose
notifier still asks to retry after this many fails.

### NOTIFICATION_RETENTION_DAYS

```rust
pub const NOTIFICATION_RETENTION_DAYS: i64 = 30;
```

Days a notification, its delivery log and its inbox rows are kept.

### MAX_DESTINATION_LABEL_BYTES

```rust
pub const MAX_DESTINATION_LABEL_BYTES: usize = 128;
```

Longest destination label, in bytes.

### MAX_ROUTES_PER_TYPE

```rust
pub const MAX_ROUTES_PER_TYPE: usize = 32;
```

Most routes, offers or patron targets one notification type may have.

### IN_APP_DESTINATION

```rust
pub const IN_APP_DESTINATION: &str = "in-app";
```

The reserved destination id of the built-in "In app" target: an
operational route to it puts the notification in the inbox of every
active Owner and Administrator, where the web app's bell, browser
notifications and the desktop window's system notifications show it.
It is not a stored destination and runs no plugin.

### TEST_NOTIFICATION_TYPE

```rust
pub const TEST_NOTIFICATION_TYPE: &str = "test";
```

The host's built-in `test` type: an administrator or a patron sends it to
one destination to check that it works.

### ACCOUNT_MAIL_RETENTION_DAYS

```rust
pub const ACCOUNT_MAIL_RETENTION_DAYS: i64 = 30;
```

How long a delivered or failed account-mail record is kept.

### EMAIL_VERIFICATION_HOURS

```rust
pub const EMAIL_VERIFICATION_HOURS: i64 = 24;
```

How long an email verification link works.

### INVITE_DAYS

```rust
pub const INVITE_DAYS: i64 = 7;
```

How long an emailed invite works, as for every claim ticket.

### VERIFY_TOKEN_CHARS

```rust
pub const VERIFY_TOKEN_CHARS: usize = 64;
```

Length of a verification link's token: 32 random bytes as lowercase hex.

### MAX_INSTANCE_ID_BYTES

```rust
pub const MAX_INSTANCE_ID_BYTES: usize = 128;
```

Longest plugin instance id the account-mail settings body takes.

### AUDIENCES

```rust
pub const AUDIENCES: [NotificationAudience; 3] = _;
```

Every audience, in the order of the `audience` column's CHECK list.

### SEVERITIES

```rust
pub const SEVERITIES: [NotificationSeverity; 3] = _;
```

Every severity, lowest first, in the order of the CHECK lists.

### ALERT_ACTIONS

```rust
pub const ALERT_ACTIONS: [NotificationAlertAction; 3] = _;
```

Every alert action, in the order of the `alert_action` CHECK list.

### HOST_NOTIFICATION_TYPES

```rust
pub const HOST_NOTIFICATION_TYPES: &[HostNotificationType] = _;
```

The host's own notification types. Producers of operational and patron
events add theirs here as they land.
