**bookclerk_integrations > brand**

# Module: brand

## Contents

**Structs**

- [`Brand`](#brand) - Visual identity for a store or integration in the portal UI.

---

## bookclerk_integrations::brand::Brand

*Struct*

Visual identity for a store or integration in the portal UI.

**Fields:**
- `id: &'static str` - Stable identifier for this item.
- `name: &'static str` - Operator-facing brand name shown on portal connect buttons.
- `bg: &'static str` - CSS background color for portal connect buttons (`#RRGGBB`).
- `fg: &'static str` - CSS foreground / text color for portal connect buttons.
- `accent: &'static str` - Optional accent (border / focus).
- `icon_url: &'static str` - Remote favicon / app-icon URL (not stored in-repo).

**Methods:**

- `fn logo_href(self: &Self) -> &'static str` - Favicon URL for `<img src>`.

**Traits:** Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Brand`
- **From**
  - `fn from(b: SourceBrand) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



