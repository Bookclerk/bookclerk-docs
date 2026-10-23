**bookclerk_plugin_catalog > kind**

# Module: kind

## Contents

**Structs**

- [`RuntimeIdentity`](#runtimeidentity) - Runtime identity used for discovery and install collision checks.

**Enums**

- [`PluginKind`](#pluginkind) - Which Bookclerk surface a plugin implements.

---

## bookclerk_plugin_catalog::kind::PluginKind

*Enum*

Which Bookclerk surface a plugin implements.

**Variants:**
- `Source` - Content-source storefront guest (Audible, Libro.fm, …).
- `Integration` - Portal / library integration guest (not a content storefront).
- `Output` - Destination / output guest (local disk, S3, …).
- `Database` - Library database backend guest (`sqlite`, `d1`, `postgres`).

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Returns the canonical lowercase kind string for manifests and crate names.
- `fn parse(s: &str) -> Option<Self>` - Parse a kind segment from a coordinate or crate name.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginKind) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PluginKind`



## bookclerk_plugin_catalog::kind::RuntimeIdentity

*Struct*

Runtime identity used for discovery and install collision checks.

**Fields:**
- `kind: PluginKind` - Destination backend kind (`local`, `s3`, …).
- `id: String` - Plugin alias (`[a-z0-9_]{2,32}`), unique within one host plugin namespace.

**Methods:**

- `fn new<impl Into<String>>(kind: PluginKind, id: impl Trait) -> Self` - Builds a runtime identity from plugin kind and id.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> RuntimeIdentity`
- **PartialEq**
  - `fn eq(self: &Self, other: &RuntimeIdentity) -> bool`



