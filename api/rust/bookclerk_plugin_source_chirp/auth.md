**bookclerk_plugin_source_chirp > auth**

# Module: auth

## Contents

**Structs**

- [`ChirpAuthFile`](#chirpauthfile) - Credential envelope for one Chirp account.

---

## bookclerk_plugin_source_chirp::auth::ChirpAuthFile

*Struct*

Credential envelope for one Chirp account.

Persisted as JSON in `encrypted_secrets` (see [`crate::db`](db.md)); never written
to disk under `Accounts/`.

**Fields:**
- `access_token: String` - JWT from GraphQL `signIn` (`user.token`).
- `web_token: Option<String>` - Longer-lived web JWT when present (`user.webToken`).
- `email: String` - Sign-in email used as fallback account id.
- `user_id: Option<String>` - Chirp user id from `signIn` when present.
- `marketplace: String` - Marketplace / locale hint stored for Bookclerk (defaults to `us`).
- `label: Option<String>` - Optional operator-facing account label in Accounts UI.

**Methods:**

- `fn account_id(self: &Self) -> &str` - Stable account id: prefer Chirp user id, else email.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ChirpAuthFile`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



