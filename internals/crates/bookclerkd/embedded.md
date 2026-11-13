# bookclerkd::embedded

The host embedded in a desktop window, with no listener.

[`EmbeddedHost::start`](#embeddedhoststart) runs the same host `bookclerkd` runs (control
plane, plugin deployments, jobs, events, scheduler, cluster links) but
binds no socket: not `daemon.listen`, not the node API's TLS listener.
The window's webview loads the SPA over its own custom scheme, and the
shell passes each request to [`EmbeddedHost::handle`](#embeddedhosthandle), which calls the
router in-process.

**Who the caller is.** The window belongs to whoever runs the desktop app
on this machine, the person the tray already hands an operator session
to. Every request is the operator: the host adds the current operator
token as `Authorization: Bearer`, read again for every request so a
rotation applies at once, and the token never reaches the page. Whatever
the page sent as cookies, `Authorization`, `Origin`, `Referer` or
forwarding headers is dropped first, and `Set-Cookie` is dropped from
every response, so the page holds no credential of its own. With
`daemon.auth.enabled = false` no token is added. The router sees every
request as coming from [`EMBEDDED_ORIGIN`](../bookclerk_window_policy/README.md#embedded_origin), a loopback origin, so a join
asks the library to send the window back there.

**Other pages.** Because the token is added here, a page from anywhere
else must not be able to reach the scheme: a request whose `Origin` is
not one of [`APP_ORIGINS`](../bookclerk_window_policy/README.md#app_origins) (or, for a write, whose `Referer` is from
elsewhere) is refused with `403` before routing.

**Leaving the app.** The window shows only app pages, with one exception:
joining a library signs the person in on the library's own site. From a
successful `POST /api/config/join/start` until the library sends the
window back (or the sign-in expires), [`EmbeddedHost::navigation`](#embeddedhostnavigation) lets
the window open `https` pages; the library's return to
`http://localhost/join/callback` is handed back to the app scheme.

**What the window cannot reach.** The request carries no peer address,
so routes that admit only a direct loopback connection (the tray
handoff) and the node API's transport policy refuse it. The node API
(`/api/node/...`) is for other nodes over TLS and is refused here before
routing. The live socket (`/api/live`) needs a WebSocket, which a custom
scheme cannot carry; it is refused, and the SPA falls back to polling.

**Responses** are collected whole, up to [`MAX_RESPONSE_BYTES`](#max_response_bytes) and
[`RESPONSE_TIMEOUT`](#response_timeout); a custom scheme cannot stream.

| Item | Kind | Summary |
| --- | --- | --- |
| [`EmbeddedOptions`](#embeddedoptions) | struct | Where the embedded host reads its files and configuration, as `bookclerkd`'s `--bookclerk-files` and `--config` do. |
| [`EmbeddedHost`](#embeddedhost) | struct | A running host with no listener, answering requests handed to it. |
| [`MAX_RESPONSE_BYTES`](#max_response_bytes) | constant | Largest response body the window receives. Larger answers (none of the web app's routes produce one) become `502 Bad Gateway`. |
| [`RESPONSE_TIMEOUT`](#response_timeout) | constant | Longest a single request may take before it is answered with `504 Gateway Timeout`. A route that streams without end (none should be reachable here) would otherwise hold the request forever. |

## Re-exports

- [`bookclerk_window_policy::Navigation`](../bookclerk_window_policy/README.md#navigation)
- [`bookclerk_window_policy::APP_ORIGINS`](../bookclerk_window_policy/README.md#app_origins)
- [`bookclerk_window_policy::EMBEDDED_ORIGIN`](../bookclerk_window_policy/README.md#embedded_origin)

## Structs

### EmbeddedOptions

```rust
pub struct EmbeddedOptions {
    pub bookclerk_files: Option<PathBuf>,
    pub config: Option<PathBuf>,
}
```

Where the embedded host reads its files and configuration, as
`bookclerkd`'s `--bookclerk-files` and `--config` do.

**Fields**

- `bookclerk_files`: Bookclerk files directory; `None` uses `BOOKCLERK_FILES_DIR` or the
  platform default.
- `config`: Path to `config.toml`; `None` uses the one in the files directory.

Implements: `Clone`, `Debug`, `Default`

### EmbeddedHost

```rust
pub struct EmbeddedHost {
    /* private fields */
}
```

A running host with no listener, answering requests handed to it.

#### EmbeddedHost::start

```rust
pub async fn start(options: EmbeddedOptions, syslog_identifier: &str) -> Result<Self>
```

Loads the configuration, installs logging and starts the host.

Call it once per process, from inside a Tokio runtime that outlives
the host: the background runtimes run on it.

##### Arguments

* `options` - Where the files directory and configuration are.
* `syslog_identifier` - The name the OS log facility records for this
  process.

##### Returns

The running host.

##### Errors

Returns an error when the configuration cannot be loaded or the host
cannot start (an interrupted clerk join, a control plane that does
not open, or operator auth that cannot be built).

#### EmbeddedHost::handle

```rust
pub async fn handle(&self, request: Request<Vec<u8>>) -> Response<Vec<u8>>
```

Answers one request from the window.

##### Arguments

* `request` - The webview's request. Only its method, path, query,
  body and non-credential headers are used.

##### Returns

The router's response, whole, without `Set-Cookie`; or `403` for a
request from another origin, `404` for a refused path, `502` for an
oversized body, `504` on timeout.

#### EmbeddedHost::navigation

```rust
pub fn navigation(&self, url: &str) -> Navigation
```

Decides whether the window may show `url`.

##### Arguments

* `url` - Where the window is about to go.

##### Returns

What the window should do (see [`Navigation`](../bookclerk_window_policy/README.md#navigation)). A
`Navigation::JoinReturn` also closes the join's sign-in window.

#### EmbeddedHost::shutdown

```rust
pub async fn shutdown(&self)
```

Stops the integration watchers before the process exits, as
`bookclerkd` does on shutdown.

Implements: `Debug`

## Constants

### MAX_RESPONSE_BYTES

```rust
pub const MAX_RESPONSE_BYTES: usize = _;
```

Largest response body the window receives. Larger answers (none of the
web app's routes produce one) become `502 Bad Gateway`.

### RESPONSE_TIMEOUT

```rust
pub const RESPONSE_TIMEOUT: Duration = _;
```

Longest a single request may take before it is answered with
`504 Gateway Timeout`. A route that streams without end (none should be
reachable here) would otherwise hold the request forever.
