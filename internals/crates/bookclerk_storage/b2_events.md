# bookclerk_storage::b2_events

Backblaze B2 event notifications and notification-rule replies, read
into checked types.

A B2 bucket announces object writes, deletes and hide markers by posting
signed batches of events to a webhook (an event notification rule). The
host checks the signature and stores each request, and the `b2`
destination reads the stored bodies to serve `changes`. The guest also
reads the bucket's rules back from `b2_get_bucket_notification_rules` and
`b2_set_bucket_notification_rules` to keep its own rules in place. Both
come from outside Bookclerk, so this module is the boundary that reads
them: pure functions over bytes, with no I/O, fuzzed by the `b2_event`
target.

A webhook body lists events:

```json
{"events": [{"eventType": "b2:ObjectCreated:Upload", "bucketName": "books",
  "objectName": "library/a.m4b", "eventTimestamp": 1684793309123,
  "eventId": "...", "objectSize": 12, "objectVersionId": "4_z..."}]}
```

Every event the feed reports becomes a `changed` entry with no object
details: B2 may deliver events late, twice or out of order, and an event
for one version says little about the name's current version (deleting
a version can uncover an older one), so the host probes each key.
Events of other types, for another bucket, or for names outside the
destination prefix are skipped. Anything else unreadable is an error:
the caller counts the delivery as a change it could not read.

| Item | Kind | Summary |
| --- | --- | --- |
| [`B2EventScope`](#b2eventscope) | struct | Which objects a `b2` destination owns. |
| [`B2SigningSecret`](#b2signingsecret) | struct | The shared secret B2 signs a rule's webhook requests with: 32 ASCII letters or digits, as B2 requires. Its `Debug` output never shows it. |
| [`NotificationRule`](#notificationrule) | struct | One event notification rule as B2 lists it. |
| [`parse_b2_events`](#parse_b2_events) | function | Reads one webhook body into changes for `scope`, in the order it lists them. |
| [`parse_notification_rules`](#parse_notification_rules) | function | Reads a `b2_get_bucket_notification_rules` or `b2_set_bucket_notification_rules` reply. |
| [`rule_names`](#rule_names) | function | The names of the destination's rules for `prefix`: `bookclerk-`, 16 hex digits of the prefix's SHA-256, and the category, one per entry of `EVENT_CATEGORIES`. B2 refuses two rules for one prefix and event type, so the names follow the prefix, not the instance. |
| [`own_rules`](#own_rules) | function | The destination's rules for `prefix`, posting to `url` signed with `secret`. |
| [`own_rules_in_place`](#own_rules_in_place) | function | True when `existing` holds the destination's rules for `prefix` as `own_rules` would write them, turned on and not suspended. |
| [`rules_with`](#rules_with) | function | The rules to send back to keep every rule in `existing` except the destination's own for `prefix`, then add `own` (empty to remove them). A suspended rule of the destination's is dropped too: B2 lets it only be deleted. |
| [`MAX_B2_EVENTS`](#max_b2_events) | constant | Most events read from one webhook body. B2 batches at most 50; the cap bounds what a crafted body can make the feed hold. |
| [`MAX_NOTIFICATION_RULES`](#max_notification_rules) | constant | Most rules one rules reply may list. B2 allows 25 per bucket. |
| [`SIGNING_SECRET_LEN`](#signing_secret_len) | constant | Length of a rule's signing secret, in characters. |
| [`EVENTS_PER_BATCH`](#events_per_batch) | constant | Events per webhook request the destination's rules ask for (B2's maximum). |
| [`EVENT_CATEGORIES`](#event_categories) | constant | The event-type categories the feed reports, each covered by one of the destination's rules (B2 refuses a rule that mixes categories). |

## Structs

### B2EventScope

```rust
pub struct B2EventScope<'a> {
    pub bucket: &'a str,
    pub prefix: &'a str,
}
```

Which objects a `b2` destination owns.

**Fields**

- `bucket`: Bucket name; events for other buckets are skipped.
- `prefix`: Normalized key prefix (empty, or ending in `/`); it is stripped from
  every name, and names outside it are skipped.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### B2SigningSecret

```rust
pub struct B2SigningSecret(/* private */);
```

The shared secret B2 signs a rule's webhook requests with: 32 ASCII
letters or digits, as B2 requires. Its `Debug` output never shows it.

#### B2SigningSecret::parse

```rust
pub fn parse(secret: &str) -> Result<Self, String>
```

Checks a secret.

##### Errors

Returns a message when the secret is not [`SIGNING_SECRET_LEN`](#signing_secret_len)
ASCII letters or digits.

#### B2SigningSecret::expose

```rust
pub fn expose(&self) -> &str
```

The secret itself, for the rule B2 stores.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### NotificationRule

```rust
pub struct NotificationRule {
    pub name: String,
    pub event_types: Vec<String>,
    pub object_name_prefix: String,
    pub is_enabled: bool,
    pub is_suspended: bool,
    pub url: Option<String>,
    pub signing_secret: Option<B2SigningSecret>,
    pub max_events_per_batch: Option<u64>,
    pub raw: Map<String, Value>,
}
```

One event notification rule as B2 lists it.

**Fields**

- `name`: The rule's name, unique in the bucket.
- `event_types`: The event types it matches.
- `object_name_prefix`: The name prefix it matches; empty for every name.
- `is_enabled`: Whether it is turned on.
- `is_suspended`: Whether B2 suspended it after failed deliveries; a suspended rule
  sends nothing and can only be deleted.
- `url`: The webhook URL, for a webhook target.
- `signing_secret`: The signing secret, when the rule has one and it is well formed.
- `max_events_per_batch`: Events per request.
- `raw`: The rule as listed, without the fields B2 computes, so a rule that is
  not the destination's own can be sent back unchanged (setting rules
  replaces them all).

Implements: `Clone`, `Debug`, `PartialEq`

## Functions

### parse_b2_events

```rust
pub fn parse_b2_events(
    body: &[u8],
    scope: B2EventScope<'_>,
) -> Result<Vec<ParsedEvent>, EventParseError>
```

Reads one webhook body into changes for `scope`, in the order it lists
them.

The body is read as a JSON tree first, so a repeated member counts once,
as its last value, and the body and each event must be JSON objects.

#### Arguments

* `body` - the request body as the host stored it.
* `scope` - the destination's bucket and key prefix.

#### Returns

A `changed` entry per reported event under `scope`, carrying the event
time as twenty zero-padded digits of Unix milliseconds (so times compare
as text), when the event states one.

#### Errors

Returns an [`EventParseError`](s3_events.md#eventparseerror) when the body is too large, is not a list
of events, lists too many, or names an unreadable object.

### parse_notification_rules

```rust
pub fn parse_notification_rules(bytes: &[u8]) -> ReplyResult<Vec<NotificationRule>>
```

Reads a `b2_get_bucket_notification_rules` or
`b2_set_bucket_notification_rules` reply.

#### Arguments

* `bytes` - the reply body.

#### Returns

The bucket's rules, in the order the reply lists them.

#### Errors

Returns [`B2ReplyError`](b2_api.md#b2replyerror) for an oversized reply, one that is not a JSON
object with a list of rule objects, more than
[`MAX_NOTIFICATION_RULES`](#max_notification_rules) rules, a rule without a name, or two rules
with one name.

### rule_names

```rust
pub fn rule_names(prefix: &str) -> [String; 3]
```

The names of the destination's rules for `prefix`: `bookclerk-`, 16 hex
digits of the prefix's SHA-256, and the category, one per entry of
[`EVENT_CATEGORIES`](#event_categories). B2 refuses two rules for one prefix and event type,
so the names follow the prefix, not the instance.

### own_rules

```rust
pub fn own_rules(prefix: &str, url: &str, secret: &B2SigningSecret) -> Vec<Value>
```

The destination's rules for `prefix`, posting to `url` signed with
`secret`.

### own_rules_in_place

```rust
pub fn own_rules_in_place(
    existing: &[NotificationRule],
    prefix: &str,
    url: &str,
    secret: &B2SigningSecret,
) -> bool
```

True when `existing` holds the destination's rules for `prefix` as
[`own_rules`](#own_rules) would write them, turned on and not suspended.

B2 does not document whether a listed rule repeats its signing secret
and batch size, so a field the listing leaves out is not held against
the rule; one it gives must match.

### rules_with

```rust
pub fn rules_with(
    existing: &[NotificationRule],
    prefix: &str,
    own: Vec<Value>,
) -> Vec<Value>
```

The rules to send back to keep every rule in `existing` except the
destination's own for `prefix`, then add `own` (empty to remove them).
A suspended rule of the destination's is dropped too: B2 lets it only
be deleted.

## Constants

### MAX_B2_EVENTS

```rust
pub const MAX_B2_EVENTS: usize = MAX_EVENT_RECORDS;
```

Most events read from one webhook body. B2 batches at most 50; the cap
bounds what a crafted body can make the feed hold.

### MAX_NOTIFICATION_RULES

```rust
pub const MAX_NOTIFICATION_RULES: usize = 100;
```

Most rules one rules reply may list. B2 allows 25 per bucket.

### SIGNING_SECRET_LEN

```rust
pub const SIGNING_SECRET_LEN: usize = 32;
```

Length of a rule's signing secret, in characters.

### EVENTS_PER_BATCH

```rust
pub const EVENTS_PER_BATCH: u32 = 50;
```

Events per webhook request the destination's rules ask for (B2's
maximum).

### EVENT_CATEGORIES

```rust
pub const EVENT_CATEGORIES: [(&str, &str); 3] = _;
```

The event-type categories the feed reports, each covered by one of the
destination's rules (B2 refuses a rule that mixes categories).
