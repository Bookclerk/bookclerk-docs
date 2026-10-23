**bookclerk_plugin_source_libro > container**

# Module: container

## Contents

**Enums**

- [`LibroContainer`](#librocontainer) - Preferred Libro.fm download packaging.

---

## bookclerk_plugin_source_libro::container::LibroContainer

*Enum*

Preferred Libro.fm download packaging.

**Variants:**
- `M4b` - Single M4B when the store offers it (default).
- `Zip` - Multi-part ZIP of MP3s.

**Methods:**

- `fn parse(raw: &str) -> Option<Self>` - Parse.

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &LibroContainer) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> LibroContainer`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> LibroContainer`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



