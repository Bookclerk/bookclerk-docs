**bookclerk_plugin_destination_local > plugin**

# Module: plugin

## Contents

**Structs**

- [`LocalDestination`](#localdestination) - Local filesystem destination capability.
- [`LocalRoot`](#localroot) - Root capability for the platform local destination guest.

---

## bookclerk_plugin_destination_local::plugin::LocalDestination

*Struct*

Local filesystem destination capability.

**Methods:**

- `fn from_config(config: &ExtensibleConfig) -> std::result::Result<Self, bookclerk_plugin_sdk::PluginError>` - Builds a destination from the `open` `Bindings::config` payload

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



## bookclerk_plugin_destination_local::plugin::LocalRoot

*Struct*

Root capability for the platform local destination guest.

**Unit Struct**

**Trait Implementations:**

- **PluginWorker**
  - `fn describe(self: &'life0 Self) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn open(self: &'life0 Self, _invocation: Invocation, bindings: Bindings) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



