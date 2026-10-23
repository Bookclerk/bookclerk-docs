**bookclerk_config > redact**

# Module: redact

## Contents

**Functions**

- [`contains_registered_secret`](#contains_registered_secret) - True if `haystack` still contains any registered exact secret (post-redaction check).
- [`is_sensitive_field`](#is_sensitive_field) - Returns true when a tracing field name should never be logged in cleartext.
- [`is_upload_identifying_field`](#is_upload_identifying_field) - Field names scrubbed from **remote** diagnostics uploads
- [`redact_field_value`](#redact_field_value) - Redact a field value: sensitive names are fully replaced; others are scrubbed.
- [`redact_str`](#redact_str) - Scrub known secrets: exact registered values, then pattern matches.
- [`register_secret`](#register_secret) - Register a known secret so it is always scrubbed from logs and diagnostics.
- [`register_secrets`](#register_secrets) - Register several secrets at once.
- [`register_secrets_from_env`](#register_secrets_from_env) - Register secrets commonly present in the process environment / config.
- [`sanitize_for_remote_upload`](#sanitize_for_remote_upload) - Extra sanitization for payloads that leave the machine (collector / B2).
- [`truncate_upload_message`](#truncate_upload_message) - Cap message length used in remote diagnostics events.
- [`unregister_secret`](#unregister_secret) - Forget a secret [`register_secret`](redact.md#bookclerk_configredactregister_secret) registered, with its encoded forms,

**Constants**

- [`REDACTED`](#redacted) - Replacement token used wherever a secret or sensitive value is scrubbed.

---

## bookclerk_config::redact::REDACTED

*Constant*: `&str`

Replacement token used wherever a secret or sensitive value is scrubbed.



## bookclerk_config::redact::contains_registered_secret

*Function*

True if `haystack` still contains any registered exact secret (post-redaction check).

```rust
fn contains_registered_secret(haystack: &str) -> bool
```



## bookclerk_config::redact::is_sensitive_field

*Function*

Returns true when a tracing field name should never be logged in cleartext.

```rust
fn is_sensitive_field(name: &str) -> bool
```



## bookclerk_config::redact::is_upload_identifying_field

*Function*

Field names scrubbed from **remote** diagnostics uploads
in addition to [`is_sensitive_field`](redact.md#bookclerk_configredactis_sensitive_field). Local journal/stderr may still show them.

```rust
fn is_upload_identifying_field(name: &str) -> bool
```



## bookclerk_config::redact::redact_field_value

*Function*

Redact a field value: sensitive names are fully replaced; others are scrubbed.

```rust
fn redact_field_value(name: &str, value: &str) -> String
```



## bookclerk_config::redact::redact_str

*Function*

Scrub known secrets: exact registered values, then pattern matches.

```rust
fn redact_str(input: &str) -> String
```



## bookclerk_config::redact::register_secret

*Function*

Register a known secret so it is always scrubbed from logs and diagnostics.

Call this whenever a passphrase, OAuth token, AWS key, etc. enters process
memory. Values shorter than 6 characters are ignored.

Also registers common encodings (percent-encoding, `+` for spaces) so a secret
cannot slip through via URL/query embedding.

```rust
fn register_secret<impl AsRef<str>>(value: impl Trait)
```



## bookclerk_config::redact::register_secrets

*Function*

Register several secrets at once.

```rust
fn register_secrets<I, S>(values: I)
```



## bookclerk_config::redact::register_secrets_from_env

*Function*

Register secrets commonly present in the process environment / config.

Safe to call multiple times. Does not log values.

```rust
fn register_secrets_from_env()
```



## bookclerk_config::redact::sanitize_for_remote_upload

*Function*

Extra sanitization for payloads that leave the machine (collector / B2).

Emails are **partially** masked (local-part truncated) so triage can still see
rough account shape without shipping a clear address. Other identifying
fields (title, path, account, …) remain fully [`REDACTED`](redact.md#bookclerk_configredactredacted).

```rust
fn sanitize_for_remote_upload(name: &str, value: &str) -> String
```



## bookclerk_config::redact::truncate_upload_message

*Function*

Cap message length used in remote diagnostics events.

```rust
fn truncate_upload_message(message: &str) -> String
```



## bookclerk_config::redact::unregister_secret

*Function*

Forget a secret [`register_secret`](redact.md#bookclerk_configredactregister_secret) registered, with its encoded forms,
once nothing in the process holds it any more (a short-lived bearer
token whose holder was dropped), so per-request credentials do not pile
up in the registry every log line is checked against. Lines captured
while it was registered were scrubbed then. Only for values unique to
one holder: the registry keeps no count, so a value two holders share
would stop being scrubbed for both.

```rust
fn unregister_secret<impl AsRef<str>>(value: impl Trait)
```



