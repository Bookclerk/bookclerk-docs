**bookclerk_plugin_source_graphicaudio > auth**

# Module: auth

## Contents

**Structs**

- [`GraphicAudioAuthFile`](#graphicaudioauthfile) - Credential envelope for one GraphicAudio account.

---

## bookclerk_plugin_source_graphicaudio::auth::GraphicAudioAuthFile

*Struct*

Credential envelope for one GraphicAudio account.

Persisted as JSON in `encrypted_secrets` (see [`crate::db`](db.md)); never written
to disk under `Accounts/`.

**Fields:**
- `token: String` - Opaque device activation token from `POST activation/login`.
- `client_id: String` - Device / client id sent at login (stable for this auth file; used by `device`).
- `email: String` - Sign-in email (also the Bookclerk account id).
- `marketplace: String` - Marketplace / locale hint stored for Bookclerk (defaults to `us`).
- `label: Option<String>` - Optional operator-facing account label in Accounts UI.

**Methods:**

- `fn account_id(self: &Self) -> &str` - Stable account id for library rows (email).
- `fn has_device_token(self: &Self) -> bool` - True when an Access App device token is present.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> GraphicAudioAuthFile`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



