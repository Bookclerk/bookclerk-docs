**bookclerk_integrations > types**

# Module: types

## Contents

**Structs**

- [`ExternalUser`](#externaluser) - External user resolved via integration credential login or watcher.
- [`IntegrationHealth`](#integrationhealth) - Health snapshot for one integration.
- [`ListeningProgressSnapshot`](#listeningprogresssnapshot) - One listening / progress row from any integration (ABS, plugins, …).
- [`SyncListeningProviderResult`](#synclisteningproviderresult) - Per-integration outcome from `crate::IntegrationRegistry::sync_listening_progress_all`.
- [`SyncListeningSummary`](#synclisteningsummary) - Result of syncing listening progress across one or more integrations.

---

## bookclerk_integrations::types::ExternalUser

*Struct*

External user resolved via integration credential login or watcher.

**Fields:**
- `provider: String` - Integration / storefront id that owns this external identity.
- `external_user_id: String` - Remote system's user id (never a Bookclerk user id).
- `display_name: Option<String>` - Optional human-facing name from the remote system.
- `access_token: Option<String>` - Ephemeral token from the remote system (e.g. ABS JWT). Never persisted.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ExternalUser`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_integrations::types::IntegrationHealth

*Struct*

Health snapshot for one integration.

**Fields:**
- `id: String` - Integration id this health row describes (`audiobookshelf`, …).
- `enabled: bool` - When true, this feature or account is active.
- `ok: bool` - When true, the last health probe succeeded.
- `detail: Option<String>` - Optional human-readable health or error detail.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> IntegrationHealth`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_integrations::types::ListeningProgressSnapshot

*Struct*

One listening / progress row from any integration (ABS, plugins, …).

Hosts upsert these into the generic `listening_progress` table; ranking never
sees the originating adapter—only these fields.

**Fields:**
- `external_user_id: String` - Remote system's user id (never a Bookclerk user id).
- `external_item_id: String` - Remote library item id for progress matching.
- `identity_id: Option<i64>` - Bookclerk portal identity row id when already linked.
- `title: Option<String>` - Display title as shown on the storefront or library card.
- `authors: Option<String>` - Comma-separated author names when the storefront provides them.
- `asin: Option<String>` - Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn: Option<String>` - Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `progress: Option<f64>` - Fraction complete in `[0.0, 1.0]` when the remote reports it.
- `current_time_seconds: Option<f64>` - Playback position in seconds from the start of the title.
- `duration_seconds: Option<f64>` - Total title duration in seconds when known.
- `is_finished: bool` - When true, the remote marks this title as finished.
- `last_listened_at: Option<chrono::DateTime<chrono::Utc>>` - UTC timestamp of the last playback event when known.

**Trait Implementations:**

- **Default**
  - `fn default() -> ListeningProgressSnapshot`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ListeningProgressSnapshot`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_integrations::types::SyncListeningProviderResult

*Struct*

Per-integration outcome from `crate::IntegrationRegistry::sync_listening_progress_all`.

**Fields:**
- `id: String` - Stable identifier for this item.
- `upserted: usize` - Number of listening_progress rows written or updated.
- `error: Option<String>` - Operator-facing error text when this step failed; `None` on success.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SyncListeningProviderResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_integrations::types::SyncListeningSummary

*Struct*

Result of syncing listening progress across one or more integrations.

**Fields:**
- `upserted: usize` - Number of listening_progress rows written or updated.
- `by_provider: Vec<SyncListeningProviderResult>` - Per-integration breakdown of upsert counts and errors.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> SyncListeningSummary`
- **Clone**
  - `fn clone(self: &Self) -> SyncListeningSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



