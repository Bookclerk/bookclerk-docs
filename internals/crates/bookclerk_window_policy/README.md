# bookclerk_window_policy

What the desktop window's custom scheme lets a page reach.

The desktop window runs the host in-process and hands every request its
webview makes on the app scheme to the router, adding its own credential
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
  `Origin` ([`EMBEDDED_ORIGIN`](#embedded_origin)) and the window's own [`Credential`](#credential) are
  set: the patron session from this OS user's profile, or the operator
  token before the library has an Owner.
- [`classify_navigation`]: the window stays on the app's pages, except
  for a library's sign-in while one started from the window is open: a
  join (this install becomes a branch) or a library card (this install
  signs in as a clerk).
- [`sign_in_ttl`]: how long that sign-in stays open, from the start
  route's answer.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Refusal`](#refusal) | enum | Why a request was refused before it reached the router. |
| [`Navigation`](#navigation) | enum | Where the window may go, as `classify_navigation` decides. |
| [`LiveRefusal`](#liverefusal) | enum | Why the desktop window's `live` command refused a caller. |
| [`Credential`](#credential) | enum | The credential the window presents for a request: the page never holds one, so the embedded host chooses it. |
| [`refusal`](#refusal-1) | function | Refuses requests from other origins and paths the window must not reach. |
| [`classify_navigation`](#classify_navigation) | function | Where the window may go. |
| [`live_refusal`](#live_refusal) | function | Decides whether a page may start the window's live updates, which carry the operator's feed. |
| [`sign_in_ttl`](#sign_in_ttl) | function | How long the window may show the library's sign-in, from a successful start route's JSON answer (a join's or a library card's). |
| [`rewrite_request_headers`](#rewrite_request_headers) | function | The headers the router sees for an embedded request: the page's own, less credentials, origins and forwarding headers, plus `Host`, `Origin` (`EMBEDDED_ORIGIN`) and the window's own credential. |
| [`EMBEDDED_HOST`](#embedded_host) | constant | The `Host` every embedded request is routed under. |
| [`EMBEDDED_ORIGIN`](#embedded_origin) | constant | The origin the router sees every embedded request come from: loopback, so a sign-in's return address is one the library accepts. |
| [`APP_ORIGINS`](#app_origins) | constant | Origins of the window's own pages: the custom scheme, and the form WebView2 (Windows) gives it. |
| [`JOIN_START_PATH`](#join_start_path) | constant | The route that starts a join; its answer opens the sign-in window. |
| [`CLERK_START_PATH`](#clerk_start_path) | constant | The route that starts a library card sign-in; its answer opens the sign-in window. |
| [`SIGN_IN_START_PATHS`](#sign_in_start_paths) | constant | The routes whose successful `POST` opens a library's sign-in in the window. |
| [`SIGN_IN_CALLBACK_PATHS`](#sign_in_callback_paths) | constant | The paths a library sends the window back to after its sign-in, on a loopback `http` address: the join's and the library card's callback. |
| [`NODE_API_PATH`](#node_api_path) | constant | The node API's path prefix: other nodes reach it over TLS, never the window. |
| [`LIVE_PATH`](#live_path) | constant | The live WebSocket's path, which a custom scheme cannot carry. The same value as `bookclerkd`'s `live::LIVE_PATH` (a test there checks). |
| [`MAX_SIGN_IN_SECS`](#max_sign_in_secs) | constant | Longest sign-in a start route may open, in seconds, whatever its answer says. |
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
    SignIn,
    SignInReturn(String),
    Refuse,
}
```

Where the window may go, as [`classify_navigation`](#classify_navigation) decides.

**Variants**

- `App`: One of the app's own pages.
- `SignIn`: A library's sign-in page, while a sign-in is open.
- `SignInReturn`: The library sending the window back after sign-in. The window opens
  this path and query on the app scheme instead; it always starts with
  one of [`SIGN_IN_CALLBACK_PATHS`](#sign_in_callback_paths).
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
  library's sign-in page.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Credential

```rust
pub enum Credential<'a> {
    None,
    Operator(&'a str),
    Cookies(&'a str),
}
```

The credential the window presents for a request: the page never holds
one, so the embedded host chooses it.

**Variants**

- `None`: No credential: operator auth is off, or nobody is signed in yet and
  the page should show the sign-in screen.
- `Operator`: The operator's bearer token: the library has no Owner yet (first
  run, or a computer that is only a clerk of another library).
- `Cookies`: The `Cookie` header the window's own session jar holds: the patron
  this OS user's profile names, and an elevated operator session when
  that patron elevated.

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
pub fn classify_navigation(raw: &str, sign_in_open: bool) -> Navigation
```

Where the window may go.

#### Arguments

* `raw` - The URL the window is about to open.
* `sign_in_open` - Whether a library's sign-in is under way.

#### Returns

The app's own pages always; the library's return to one of
[`SIGN_IN_CALLBACK_PATHS`](#sign_in_callback_paths) on a loopback `http` address (any host and
port) as a [`Navigation::SignInReturn`](#navigation) while a sign-in is open; other
`https` pages, or loopback `http` ones, only while a sign-in is open.

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

### sign_in_ttl

```rust
pub fn sign_in_ttl(body: &[u8]) -> Option<Duration>
```

How long the window may show the library's sign-in, from a successful
start route's JSON answer (a join's or a library card's).

#### Arguments

* `body` - The answer's body.

#### Returns

`expires_in_secs`, at most [`MAX_SIGN_IN_SECS`](#max_sign_in_secs), when the body is a
JSON object with a string `authorize_url` and an unsigned integer
`expires_in_secs`; `None` otherwise.

### rewrite_request_headers

```rust
pub fn rewrite_request_headers(page: &HeaderMap, credential: Credential<'_>) -> HeaderMap
```

The headers the router sees for an embedded request: the page's own,
less credentials, origins and forwarding headers, plus `Host`, `Origin`
([`EMBEDDED_ORIGIN`](#embedded_origin)) and the window's own credential.

#### Arguments

* `page` - The headers the webview sent.
* `credential` - What the window presents: nothing, the operator's
  bearer token, or its session jar's `Cookie` header.

#### Returns

The rewritten headers. A token or cookie text that is not a valid header
value is left out, so the request goes unauthenticated (fail closed);
the caller can tell from the missing `Authorization` or `Cookie`.

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
so a sign-in's return address is one the library accepts.

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

### CLERK_START_PATH

```rust
pub const CLERK_START_PATH: &str = "/api/config/clerk/start";
```

The route that starts a library card sign-in; its answer opens the
sign-in window.

### SIGN_IN_START_PATHS

```rust
pub const SIGN_IN_START_PATHS: [&str; 2] = _;
```

The routes whose successful `POST` opens a library's sign-in in the
window.

### SIGN_IN_CALLBACK_PATHS

```rust
pub const SIGN_IN_CALLBACK_PATHS: [&str; 2] = _;
```

The paths a library sends the window back to after its sign-in, on a
loopback `http` address: the join's and the library card's callback.

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

### MAX_SIGN_IN_SECS

```rust
pub const MAX_SIGN_IN_SECS: u64 = 3600;
```

Longest sign-in a start route may open, in seconds, whatever its answer
says.

### MAIN_WINDOW_LABEL

```rust
pub const MAIN_WINDOW_LABEL: &str = "main";
```

The label of the desktop app's one window, the only webview the `live`
command answers.
