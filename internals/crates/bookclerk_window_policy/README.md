# bookclerk_window_policy

What the desktop window's custom scheme lets a page reach.

The desktop window runs the host in-process and hands every request its
webview makes on the app scheme to the router, adding the operator token
itself (`bookclerkd::embedded`). That makes the scheme handler an
authorization boundary over bytes the host did not produce: the method,
headers and path a page sent, and the URLs the window is asked to open.
This crate is that boundary's policy, kept free of I/O so the
`window_policy` fuzz target and the unit tests drive the same code the
window does:

- [`refusal`]: a request from a page that is not the app's (by `Origin`,
  or for a write by `Referer`) is refused, and so are the node API and
  the live socket, before routing.
- [`rewrite_request_headers`]: the headers the router sees. The page's
  credentials, origins and forwarding headers are dropped; `Host`,
  `Origin` ([`EMBEDDED_ORIGIN`](#embedded_origin)) and the operator token are set.
- [`classify_navigation`]: the window stays on the app's pages, except
  for a library's sign-in while a join started from the window is open.
- [`join_sign_in_ttl`]: how long that sign-in stays open, from the join
  start's answer.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Refusal`](#refusal) | enum | Why a request was refused before it reached the router. |
| [`Navigation`](#navigation) | enum | Where the window may go, as `classify_navigation` decides. |
| [`LiveRefusal`](#liverefusal) | enum | Why the desktop window's `live` command refused a caller. |
| [`refusal`](#refusal-1) | function | Refuses requests from other origins and paths the window must not reach. |
| [`classify_navigation`](#classify_navigation) | function | Where the window may go. |
| [`live_refusal`](#live_refusal) | function | Decides whether a page may start the window's live updates, which carry the operator's feed. |
| [`join_sign_in_ttl`](#join_sign_in_ttl) | function | How long the window may show the library's sign-in, from a successful join start's JSON answer. |
| [`rewrite_request_headers`](#rewrite_request_headers) | function | The headers the router sees for an embedded request: the page's own, less credentials, origins and forwarding headers, plus `Host`, `Origin` (`EMBEDDED_ORIGIN`) and, when operator auth is on, the operator's bearer token. |
| [`EMBEDDED_HOST`](#embedded_host) | constant | The `Host` every embedded request is routed under. |
| [`EMBEDDED_ORIGIN`](#embedded_origin) | constant | The origin the router sees every embedded request come from: loopback, so a join's return address is one the library accepts. |
| [`APP_ORIGINS`](#app_origins) | constant | Origins of the window's own pages: the custom scheme, and the form WebView2 (Windows) gives it. |
| [`JOIN_START_PATH`](#join_start_path) | constant | The route that starts a join; its answer opens the sign-in window. |
| [`NODE_API_PATH`](#node_api_path) | constant | The node API's path prefix: other nodes reach it over TLS, never the window. |
| [`LIVE_PATH`](#live_path) | constant | The live WebSocket's path, which a custom scheme cannot carry. The same value as `bookclerkd`'s `live::LIVE_PATH` (a test there checks). |
| [`MAX_JOIN_SIGN_IN_SECS`](#max_join_sign_in_secs) | constant | Longest sign-in a join start may open, in seconds, whatever its answer says. |
| [`MAIN_WINDOW_LABEL`](#main_window_label) | constant | The label of the desktop app's one window, the only webview the `live` command answers. |

## Enums

### Refusal

```rust
pub enum Refusal {
    NodeApi,
    LiveSocket,
    ForeignOrigin,
}
```

Why a request was refused before it reached the router.

**Variants**

- `NodeApi`: A node API path: other nodes reach it over TLS, never the window.
- `LiveSocket`: The live WebSocket, which a custom scheme cannot carry.
- `ForeignOrigin`: A request a page from another origin sent.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Navigation

```rust
pub enum Navigation {
    App,
    JoinSignIn,
    JoinReturn(String),
    Refuse,
}
```

Where the window may go, as [`classify_navigation`](#classify_navigation) decides.

**Variants**

- `App`: One of the app's own pages.
- `JoinSignIn`: A library's sign-in page, while a join is open.
- `JoinReturn`: The library sending the window back after sign-in. The window opens
  this path and query on the app scheme instead; it always starts with
  the join callback path.
- `Refuse`: Anywhere else: refused.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### LiveRefusal

```rust
pub enum LiveRefusal {
    OtherWebview,
    ForeignPage,
}
```

Why the desktop window's `live` command refused a caller.

**Variants**

- `OtherWebview`: A webview other than the main window called it.
- `ForeignPage`: The calling page is not one of the app's own pages, such as a
  library's sign-in page shown during a join.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### refusal

```rust
pub fn refusal(method: &Method, headers: &HeaderMap, path: &str) -> Option<Refusal>
```

Refuses requests from other origins and paths the window must not reach.

#### Arguments

* `method` - The request method.
* `headers` - The headers the webview sent.
* `path` - The request path, undecoded (as the router matches it).

#### Returns

`Some` with the reason for a refused request, `None` otherwise. An
origin refusal wins over a path one.

### classify_navigation

```rust
pub fn classify_navigation(raw: &str, join_open: bool) -> Navigation
```

Where the window may go.

#### Arguments

* `raw` - The URL the window is about to open.
* `join_open` - Whether a join's sign-in is under way.

#### Returns

The app's own pages always; the library's return to
`http://localhost/join/callback` (any loopback host and port) as a
[`Navigation::JoinReturn`](#navigation) while a join is open; other `https` pages,
or loopback `http` ones, only while a join is open.

### live_refusal

```rust
pub fn live_refusal(webview_label: &str, page_url: &str) -> Option<LiveRefusal>
```

Decides whether a page may start the window's live updates, which carry
the operator's feed.

#### Arguments

* `webview_label` - The calling webview's label.
* `page_url` - The URL of the page that called, as the webview reports
  it.

#### Returns

`None` when the main window's app page called; otherwise why it was
refused. A wrong webview wins over a foreign page.

### join_sign_in_ttl

```rust
pub fn join_sign_in_ttl(body: &[u8]) -> Option<Duration>
```

How long the window may show the library's sign-in, from a successful
join start's JSON answer.

#### Arguments

* `body` - The answer's body.

#### Returns

`expires_in_secs`, at most [`MAX_JOIN_SIGN_IN_SECS`](#max_join_sign_in_secs), when the body is a
JSON object with a string `authorize_url` and an unsigned integer
`expires_in_secs`; `None` otherwise.

### rewrite_request_headers

```rust
pub fn rewrite_request_headers(page: &HeaderMap, bearer: Option<&str>) -> HeaderMap
```

The headers the router sees for an embedded request: the page's own,
less credentials, origins and forwarding headers, plus `Host`, `Origin`
([`EMBEDDED_ORIGIN`](#embedded_origin)) and, when operator auth is on, the operator's bearer
token.

#### Arguments

* `page` - The headers the webview sent.
* `bearer` - The operator token, or `None` when operator auth is off.

#### Returns

The rewritten headers. A token that is not a valid header value is left
out, so the request goes unauthenticated (fail closed); the caller can
tell from the missing `Authorization`.

## Constants

### EMBEDDED_HOST

```rust
pub const EMBEDDED_HOST: &str = "localhost";
```

The `Host` every embedded request is routed under.

### EMBEDDED_ORIGIN

```rust
pub const EMBEDDED_ORIGIN: &str = "http://localhost";
```

The origin the router sees every embedded request come from: loopback,
so a join's return address is one the library accepts.

### APP_ORIGINS

```rust
pub const APP_ORIGINS: [&str; 2] = _;
```

Origins of the window's own pages: the custom scheme, and the form
WebView2 (Windows) gives it.

### JOIN_START_PATH

```rust
pub const JOIN_START_PATH: &str = "/api/config/join/start";
```

The route that starts a join; its answer opens the sign-in window.

### NODE_API_PATH

```rust
pub const NODE_API_PATH: &str = "/api/node";
```

The node API's path prefix: other nodes reach it over TLS, never the
window.

### LIVE_PATH

```rust
pub const LIVE_PATH: &str = "/api/live";
```

The live WebSocket's path, which a custom scheme cannot carry. The same
value as `bookclerkd`'s `live::LIVE_PATH` (a test there checks).

### MAX_JOIN_SIGN_IN_SECS

```rust
pub const MAX_JOIN_SIGN_IN_SECS: u64 = 3600;
```

Longest sign-in a join start may open, in seconds, whatever its answer
says.

### MAIN_WINDOW_LABEL

```rust
pub const MAIN_WINDOW_LABEL: &str = "main";
```

The label of the desktop app's one window, the only webview the `live`
command answers.
