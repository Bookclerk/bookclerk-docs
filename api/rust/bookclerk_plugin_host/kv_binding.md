**bookclerk_plugin_host > kv_binding**

# Module: kv_binding

## Contents

**Structs**

- [`KvNamespaces`](#kvnamespaces) - Host-owned facts a session needs before it can hand a guest its KV
- [`LibraryKvNamespace`](#librarykvnamespace) - [`KvNamespace`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abiroleskvnamespace) server for one `(plugin_key, namespace)` on the library.

**Constants**

- [`KV_GRANT_BINDING`](#kv_grant_binding) - Grant token that authorizes every `[[kv_namespaces]]` row of a plugin.

---

## bookclerk_plugin_host::kv_binding::KV_GRANT_BINDING

*Constant*: `&str`

Grant token that authorizes every `[[kv_namespaces]]` row of a plugin.



## bookclerk_plugin_host::kv_binding::KvNamespaces

*Struct*

Host-owned facts a session needs before it can hand a guest its KV
bindings: the library store plus the granted rows.

**Methods:**

- `fn new(store: LibraryStore, plugin_key: &str, manifest: &PluginManifest, grant: &PluginGrant) -> Option<Self>` - Builds the KV hook for `plugin_key`, or `None` when the manifest
- `fn binding_names(self: &Self) -> Vec<String>` - Granted binding names in manifest order.
- `fn namespaces(self: &Self) -> Vec<(String, String, Arc<dyn KvNamespace>)>` - Namespaces shaped for `HostBindings::kv` as `(binding, namespace, kv)`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> KvNamespaces`



## bookclerk_plugin_host::kv_binding::LibraryKvNamespace

*Struct*

[`KvNamespace`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abiroleskvnamespace) server for one `(plugin_key, namespace)` on the library.

**Methods:**

- `fn new(store: LibraryStore, plugin_key: &str, row: &KvNamespaceManifest) -> Self` - Builds one namespace directly (tests and in-process hosts).
- `fn namespace(self: &Self) -> &str` - Namespace this binding is scoped to.

**Trait Implementations:**

- **KvNamespace**
  - `fn get(self: &'life0 Self, key: String) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn put(self: &'life0 Self, key: String, value: Vec<u8>, options: KvPutOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn delete(self: &'life0 Self, key: String) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn list(self: &'life0 Self, options: KvListOptions) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



