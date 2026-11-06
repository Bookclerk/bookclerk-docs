# bookclerk_tray

In-process system tray for `bookclerkd` (opens the web UI in a browser).

- Linux: StatusNotifierItem via `ksni` (no GTK/WebKit)
- Windows / macOS: `tray-icon` with default features disabled (no GTK)

| Item | Kind | Summary |
| --- | --- | --- |
| [`TrayConfig`](#trayconfig) | struct | Configuration for the in-process tray (no child `bookclerkd` process). |
| [`run_blocking`](#run_blocking) | function | Run the tray on the current thread until the user chooses Quit tray. |
| [`spawn`](#spawn) | function | Start the tray on a dedicated OS thread (safe beside the Tokio HTTP runtime). |
| [`SharedTrayConfig`](#sharedtrayconfig) | type alias | Shared tray config so the daemon can refresh `base_url` after listen rebinds. |

## Structs

### TrayConfig

```rust
pub struct TrayConfig {
    pub base_url: String,
    pub auth_enabled: bool,
    pub operator_token: Option<String>,
}
```

Configuration for the in-process tray (no child `bookclerkd` process).

**Fields**

- `base_url`: Daemon HTTP base URL (e.g. `http://127.0.0.1:8787`).
- `auth_enabled`: When true, operator-token auth is required for tray actions.
- `operator_token`: Operator bearer token used for authenticated daemon calls.

#### TrayConfig::base_url

```rust
pub fn base_url(listen: &str) -> String
```

Builds an HTTP base URL from a daemon listen address.

##### Arguments

* `listen` - `host:port` or an absolute `http(s)://` URL (trailing `/` stripped).

##### Returns

Absolute base URL with an `http://` scheme when `listen` was host-only.

#### TrayConfig::ui_url

```rust
pub fn ui_url(&self) -> String
```

Open the UI via a loopback handoff URL that sets the session cookie.

The durable operator token is never placed in the URL. [`Self::open_ui`](#trayconfigopen_ui)
POSTs `/api/auth/tray-handoff/prepare` with Bearer first, then opens the
returned one-time `?code=` GET (Linux `xdg-open` does not need a fragment).

#### TrayConfig::open_ui

```rust
pub fn open_ui(&self) -> Result<()>
```

Opens the daemon web UI in the default browser.

##### Errors

Returns an error when prepare fails or the OS cannot launch a browser.

#### TrayConfig::trigger_scan

```rust
pub fn trigger_scan(&self) -> Result<()>
```

POSTs an authenticated library scan request to the daemon.

##### Errors

Returns an error when the HTTP request fails or the daemon rejects it.

#### TrayConfig::copy_sign_in_link

```rust
pub fn copy_sign_in_link(&self)
```

Copy a loopback sign-in link to the clipboard (never the durable token).

#### TrayConfig::set_listen

```rust
pub fn set_listen(&mut self, listen: &str)
```

Updates the configured daemon listen address used to derive [`Self::base_url`](#trayconfigbase_url).

##### Arguments

* `listen` - Daemon listen address (`host:port` or URL).

#### TrayConfig::set_base_url

```rust
pub fn set_base_url(&mut self, base_url: impl Into<String>)
```

Overrides the daemon HTTP base URL used for subsequent calls.

##### Arguments

* `base_url` - Absolute HTTP base URL for the daemon.

Implements: `Clone`, `Debug`

## Functions

### run_blocking

```rust
pub fn run_blocking(config: SharedTrayConfig) -> Result<()>
```

Run the tray on the current thread until the user chooses Quit tray.

#### Arguments

* `config` - Loaded Bookclerk configuration.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### spawn

```rust
pub fn spawn(config: SharedTrayConfig) -> JoinHandle<Result<()>>
```

Start the tray on a dedicated OS thread (safe beside the Tokio HTTP runtime).

#### Arguments

* `config` - Loaded Bookclerk configuration.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

## Type aliases

### SharedTrayConfig

```rust
pub type SharedTrayConfig = Arc<Mutex<TrayConfig>>;
```

Shared tray config so the daemon can refresh `base_url` after listen rebinds.
