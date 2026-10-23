**bookclerk_plugin_source_audible > qr**

# Module: qr

## Contents

**Enums**

- [`QrRenderMode`](#qrrendermode) - How to print the login QR in a terminal.

**Functions**

- [`render_login_qr`](#render_login_qr) - Render `url` as a terminal QR code string (plus the raw URL).

---

## bookclerk_plugin_source_audible::qr::QrRenderMode

*Enum*

How to print the login QR in a terminal.

**Variants:**
- `Unicode` - Dense Unicode half-blocks (default; works in most modern terminals).
- `Ascii` - ASCII `#` / space art for limited terminals.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> QrRenderMode`
- **PartialEq**
  - `fn eq(self: &Self, other: &QrRenderMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> QrRenderMode`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::qr::render_login_qr

*Function*

Render `url` as a terminal QR code string (plus the raw URL).

# Errors

Returns an error when the operation fails.

```rust
fn render_login_qr(url: &str, mode: QrRenderMode) -> crate::error::Result<String>
```



