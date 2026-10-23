**bookclerk_tray > client**

# Module: client

## Contents

**Structs**

- [`TrayConfig`](#trayconfig) - Configuration for the in-process tray (no child `bookclerkd` process).

**Type Aliases**

- [`SharedTrayConfig`](#sharedtrayconfig) - Shared tray config so the daemon can refresh `base_url` after listen rebinds.

---

## bookclerk_tray::client::SharedTrayConfig

*Type Alias*: `std::sync::Arc<std::sync::Mutex<TrayConfig>>`

Shared tray config so the daemon can refresh `base_url` after listen rebinds.



## bookclerk_tray::client::TrayConfig

*Struct*

Configuration for the in-process tray (no child `bookclerkd` process).

**Fields:**
- `base_url: String` - Daemon HTTP base URL (e.g. `http://127.0.0.1:8787`).
- `auth_enabled: bool` - When true, operator-token auth is required for tray actions.
- `operator_token: Option<String>` - Operator bearer token used for authenticated daemon calls.

**Methods:**

- `fn base_url(listen: &str) -> String` - Builds an HTTP base URL from a daemon listen address.
- `fn ui_url(self: &Self) -> String` - Open the UI via a loopback handoff URL that sets the session cookie.
- `fn open_ui(self: &Self) -> anyhow::Result<()>` - Opens the daemon web UI in the default browser.
- `fn trigger_scan(self: &Self) -> anyhow::Result<()>` - POSTs an authenticated library scan request to the daemon.
- `fn copy_sign_in_link(self: &Self)` - Copy a loopback sign-in link to the clipboard (never the durable token).
- `fn set_listen(self: & mut Self, listen: &str)` - Updates the configured daemon listen address used to derive `Self::base_url`.
- `fn set_base_url<impl Into<String>>(self: & mut Self, base_url: impl Trait)` - Overrides the daemon HTTP base URL used for subsequent calls.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> TrayConfig`



