**bookclerk_source > brand**

# Module: brand

## Contents

**Structs**

- [`SourceBrand`](#sourcebrand) - Visual identity for a content source in the connect portal.

---

## bookclerk_source::brand::SourceBrand

*Struct*

Visual identity for a content source in the connect portal.

**Fields:**
- `id: &'static str` - Stable plugin id matching `crate::ContentSource::id` (`audible`, …).
- `name: &'static str` - Human-facing store name shown on the connect button.
- `bg: &'static str` - CSS background color (hex) for the portal button.
- `fg: &'static str` - CSS foreground / text color (hex) for the portal button.
- `accent: &'static str` - CSS accent color (hex) for focus / highlights.
- `icon_url: &'static str` - Remote favicon / app-icon URL (not stored in-repo).

**Methods:**

- `fn logo_href(self: &Self) -> &'static str` - Favicon / app-icon URL used as the portal button logo (`href`).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SourceBrand`
- **PartialEq**
  - `fn eq(self: &Self, other: &SourceBrand) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



