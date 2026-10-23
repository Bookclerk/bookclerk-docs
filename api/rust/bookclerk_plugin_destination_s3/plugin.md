**bookclerk_plugin_destination_s3 > plugin**

# Module: plugin

## Contents

**Structs**

- [`S3Destination`](#s3destination) - S3 destination capability (multipart sink for streamed puts).
- [`S3Root`](#s3root) - Root capability for the optional S3 destination guest.

---

## bookclerk_plugin_destination_s3::plugin::S3Destination

*Struct*

S3 destination capability (multipart sink for streamed puts).

**Methods:**

- `fn from_config(config: &ExtensibleConfig) -> std::result::Result<Self, bookclerk_plugin_sdk::PluginError>` - Builds a destination from the `open` `Bindings::config` payload
- `fn from_bindings(bindings: &Bindings) -> std::result::Result<Self, bookclerk_plugin_sdk::PluginError>` - `Self::from_config` plus credentials from `SECRETS` when `CONFIG` omits them.

**Trait Implementations:**

- **Destination**
  - `fn head(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list(self: &'life0 Self, options: ListOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn get(self: &'life0 Self, key: &'life1 str, range: Option<ByteRange>) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put(self: &'life0 Self, key: &'life1 str, body: Pin<Box<dyn AsyncRead>>, options: WriteOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn copy(self: &'life0 Self, from: &'life1 str, to: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn delete(self: &'life0 Self, key: &'life1 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn commit(self: &'life0 Self, key: &'life1 str, commit_token: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn abort_stage(self: &'life0 Self, key: &'life1 str, commit_token: &'life2 str) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_destination_s3::plugin::S3Root

*Struct*

Root capability for the optional S3 destination guest.

**Unit Struct**

**Trait Implementations:**

- **PluginWorker**
  - `fn describe(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn open(self: &'life0 Self, _invocation: Invocation, bindings: Bindings) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



