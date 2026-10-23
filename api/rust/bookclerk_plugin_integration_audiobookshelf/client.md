**bookclerk_plugin_integration_audiobookshelf > client**

# Module: client

## Contents

**Structs**

- [`AbsApiClient`](#absapiclient) - Thin ABS REST client using Bearer auth.
- [`AbsItemMedia`](#absitemmedia) - Nested `media` object on an ABS library item.
- [`AbsItemMetadata`](#absitemmetadata) - Matching fields from ABS item metadata (all optional).
- [`AbsLibrary`](#abslibrary) - Audiobookshelf library.
- [`AbsLibraryItem`](#abslibraryitem) - Audiobookshelf library item.
- [`AbsMediaProgress`](#absmediaprogress) - Audiobookshelf media progress.
- [`AbsUser`](#absuser) - Audiobookshelf user.
- [`AbsUserDetail`](#absuserdetail) - Full ABS user payload including listening progress.
- [`AuthorizeResponse`](#authorizeresponse) - `POST /api/authorize` payload used to validate an API token.
- [`LoginResponse`](#loginresponse) - `POST /login` payload; `user` is required to map an [`ExternalUser`](../bookclerk_integrations/types.md#bookclerk_integrationstypesexternaluser).

---

## bookclerk_plugin_integration_audiobookshelf::client::AbsApiClient

*Struct*

Thin ABS REST client using Bearer auth.

**Methods:**

- `fn new<impl Into<String>, impl Into<String>>(base_url: impl Trait, api_key: impl Trait) -> Result<Self>` - Build a client. `base_url` should be scheme+host (no trailing slash).
- `fn base_url(self: &Self) -> &str` - Base URL.
- `fn authorize(self: &Self) -> Result<AuthorizeResponse>` - `POST /api/authorize` — validate API token / key.
- `fn login(self: &Self, username: &str, password: &str) -> Result<LoginResponse>` - Username/password login (`POST /login`). Password is never logged.
- `fn authenticate_user(self: &Self, username: &str, password: &str) -> Result<ExternalUser>` - Map a successful login into an [`ExternalUser`](../bookclerk_integrations/types.md#bookclerk_integrationstypesexternaluser).
- `fn get_user(self: &Self, user_id: &str) -> Result<AbsUserDetail>` - `GET /api/users/{id}` — full user including `mediaProgress`.
- `fn get_library_item(self: &Self, item_id: &str) -> Result<AbsLibraryItem>` - `GET /api/items/{id}` — library item metadata for matching.
- `fn list_libraries(self: &Self) -> Result<Vec<AbsLibrary>>` - `GET /api/libraries`
- `fn scan_library(self: &Self, library_id: &str, force: bool) -> Result<()>` - `POST /api/libraries/{id}/scan`
- `fn list_users(self: &Self) -> Result<Vec<AbsUser>>` - `GET /api/users` (admin).
- `fn search_library(self: &Self, library_id: &str, q: &str) -> Result<Value>` - `GET /api/libraries/{id}/search?q=`

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AbsApiClient`



## bookclerk_plugin_integration_audiobookshelf::client::AbsItemMedia

*Struct*

Nested `media` object on an ABS library item.

**Fields:**
- `metadata: Option<AbsItemMetadata>` - Title / author / ASIN / ISBN used to match Bookclerk library rows.



## bookclerk_plugin_integration_audiobookshelf::client::AbsItemMetadata

*Struct*

Matching fields from ABS item metadata (all optional).

**Fields:**
- `title: Option<String>` - Title string when ABS stored one.
- `author_name: Option<String>` - Display author name when ABS stored one.
- `asin: Option<String>` - Amazon ASIN when ABS stored one.
- `isbn: Option<String>` - ISBN when ABS stored one.



## bookclerk_plugin_integration_audiobookshelf::client::AbsLibrary

*Struct*

Audiobookshelf library.

**Fields:**
- `id: String` - Identifier.
- `name: String` - Display or configuration name.
- `media_type: Option<String>` - Media type.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AbsLibrary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_integration_audiobookshelf::client::AbsLibraryItem

*Struct*

Audiobookshelf library item.

**Fields:**
- `id: String` - Identifier.
- `media: Option<AbsItemMedia>` - Media.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AbsLibraryItem`



## bookclerk_plugin_integration_audiobookshelf::client::AbsMediaProgress

*Struct*

Audiobookshelf media progress.

**Fields:**
- `id: String` - Identifier.
- `library_item_id: String` - Library item Identifier.
- `duration: Option<f64>` - Duration.
- `progress: Option<f64>` - Progress.
- `current_time: Option<f64>` - Current time.
- `is_finished: bool` - Is finished.
- `last_update: Option<i64>` - Last update.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AbsMediaProgress`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_integration_audiobookshelf::client::AbsUser

*Struct*

Audiobookshelf user.

**Fields:**
- `id: String` - Identifier.
- `username: String` - Username.
- `token: Option<String>` - Token.
- `typ: Option<String>` - Typ.
- `user_type: Option<String>` - User type.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AbsUser`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_integration_audiobookshelf::client::AbsUserDetail

*Struct*

Full ABS user payload including listening progress.

**Fields:**
- `id: String` - Identifier.
- `username: String` - Username.
- `media_progress: Vec<AbsMediaProgress>` - Media progress.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AbsUserDetail`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_integration_audiobookshelf::client::AuthorizeResponse

*Struct*

`POST /api/authorize` payload used to validate an API token.

**Fields:**
- `user: Option<AbsUser>` - User bound to the API token when authorize succeeds.
- `server_settings: Option<serde_json::Value>` - Optional server settings object from ABS (unused by Bookclerk).



## bookclerk_plugin_integration_audiobookshelf::client::LoginResponse

*Struct*

`POST /login` payload; `user` is required to map an [`ExternalUser`](../bookclerk_integrations/types.md#bookclerk_integrationstypesexternaluser).

**Fields:**
- `user: Option<AbsUser>` - Authenticated ABS user; missing `user` fails `authenticate_user`.
- `server_settings: Option<serde_json::Value>` - Optional server settings object from ABS (unused by Bookclerk).



