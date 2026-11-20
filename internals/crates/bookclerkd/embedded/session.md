# bookclerkd::embedded::session

Who the desktop window is: the patron whose session this branch keeps
sealed for its window, and the session cookies the window holds for them
where the page cannot see them.

The window acts as the operator only until the library has an active
Owner (first run, or a computer that is only a clerk of another
library). After that it acts as a patron: the one whose portal session
is sealed in this branch's `encrypted_secrets` ([`remembered`](#remembered)), or
nobody until they sign in. The files directory, and so the database, is
already this OS account's own, so the sealed row is per OS account
without a file of its own. [`WindowJar`](#windowjar) keeps that patron's portal
session cookie, and the operator session cookie an elevation sets, so
the routes the web app uses for elevation, impersonation and sign-out
work unchanged; `Set-Cookie` never reaches the page.

See `docs/adr/desktop-window-identity.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WindowSession`](#windowsession) | struct | A patron's raw portal session token, as the window keeps it: never printed by `Debug`, and wiped from memory when dropped. |
| [`WindowJar`](#windowjar) | struct | The session cookies the window holds: never sent to the page, added to each request instead of anything the page sent. |
| [`mint`](#mint) | function | Mints a window session for `user_id`: a portal session labelled **Desktop app on `<computer>`** in the patron's session list, lasting `DESKTOP_SESSION_TTL`. |
| [`patron`](#patron) | function | The patron a window session signs in as, when it still can: the session exists and has not expired, and its patron exists and is not disabled. |
| [`end`](#end) | function | Ends a session the window no longer uses (best effort). |
| [`remembered`](#remembered) | function | The patron session this branch's window last signed in with. |
| [`remember`](#remember) | function | Seals `session` as the one this branch's window signs in with, replacing any other. |
| [`forget`](#forget) | function | Forgets the session this branch's window signs in with. |
| [`DESKTOP_SESSION_TTL`](#desktop_session_ttl) | constant | How long a window session lasts before the window must be signed in again. The window renews it on every launch, so only a computer left unopened this long signs out. |

## Structs

### WindowSession

```rust
pub struct WindowSession(/* private */);
```

A patron's raw portal session token, as the window keeps it: never
printed by `Debug`, and wiped from memory when dropped.

#### WindowSession::new

```rust
pub fn new(raw: &str) -> Option<Self>
```

Wraps a raw session token, and registers it with log redaction.

##### Arguments

* `raw` - The token, as the portal session cookie carries it.

##### Returns

The session, or `None` for an empty token.

#### WindowSession::expose

```rust
pub fn expose(&self) -> &str
```

The raw token, for the cookie and the session lookup only.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WindowJar

```rust
pub struct WindowJar {
    pub portal: Option<String>,
    pub operator: Option<String>,
    pub operator_until: Option<Instant>,
}
```

The session cookies the window holds: never sent to the page, added to
each request instead of anything the page sent.

**Fields**

- `portal`: The patron's portal session token (`bookclerk_portal_session`).
- `operator`: An operator session token (`bookclerk_operator_session`), present
  while the patron is elevated.
- `operator_until`: When the operator cookie's `Max-Age` runs out; `None` for a cookie
  with no `Max-Age`, which lasts until the window closes.

#### WindowJar::cookie_header

```rust
pub fn cookie_header(&self) -> Option<String>
```

The `Cookie` header for the jar's sessions.

##### Returns

`name=value` pairs joined by `; `, or `None` when the jar is empty.

#### WindowJar::drop_expired

```rust
pub fn drop_expired(&mut self, now: Instant)
```

Drops the operator cookie once its `Max-Age` has run out, as a
browser would, so an elevation that ended on its own stops counting.

##### Arguments

* `now` - The current time.

#### WindowJar::absorb

```rust
pub fn absorb(&mut self, set_cookies: &[HeaderValue])
```

Applies the router's `Set-Cookie` headers to the jar.

Only the two session cookies are kept; an empty value or
`Max-Age=0` removes one. Other cookies (the sign-in provider's
transaction cookie) are ignored: the window cannot follow a provider
sign-in anyway.

##### Arguments

* `set_cookies` - The response's `Set-Cookie` values, in order.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### mint

```rust
pub async fn mint(library: &LibraryStore, user_id: i64) -> Option<WindowSession>
```

Mints a window session for `user_id`: a portal session labelled
**Desktop app on `<computer>`** in the patron's session list, lasting
[`DESKTOP_SESSION_TTL`](#desktop_session_ttl).

#### Arguments

* `library` - The library on this computer.
* `user_id` - The patron.

#### Returns

The new session's raw token, or `None` when the patron does not exist,
is disabled, or the session cannot be stored.

### patron

```rust
pub async fn patron(
    library: &LibraryStore,
    session: &WindowSession,
) -> Result<Option<i64>, LibraryError>
```

The patron a window session signs in as, when it still can: the
session exists and has not expired, and its patron exists and is not
disabled.

#### Arguments

* `library` - The library on this computer.
* `session` - The window's session.

#### Returns

The patron's user id, or `None` when the session has ended.

#### Errors

Returns the read error, so a database that is briefly unreachable is
not taken for an ended session.

### end

```rust
pub async fn end(library: &LibraryStore, session: &WindowSession)
```

Ends a session the window no longer uses (best effort).

#### Arguments

* `library` - The library on this computer.
* `session` - The session to end.

### remembered

```rust
pub async fn remembered(
    library: &LibraryStore,
    node_id: &str,
) -> Result<Option<WindowSession>, LibraryError>
```

The patron session this branch's window last signed in with.

#### Arguments

* `library` - The library on this computer.
* `node_id` - This branch's node id, which keys the sealed row.

#### Returns

The session, or `None` when none is kept.

#### Errors

Returns the read error, or an error when the row does not unseal.

### remember

```rust
pub async fn remember(
    library: &LibraryStore,
    node_id: &str,
    session: &WindowSession,
) -> Result<(), LibraryError>
```

Seals `session` as the one this branch's window signs in with,
replacing any other.

#### Arguments

* `library` - The library on this computer.
* `node_id` - This branch's node id, which keys the sealed row.
* `session` - The patron's session.

#### Errors

Returns an error when the session cannot be sealed or stored.

### forget

```rust
pub async fn forget(library: &LibraryStore, node_id: &str) -> Result<(), LibraryError>
```

Forgets the session this branch's window signs in with.

#### Arguments

* `library` - The library on this computer.
* `node_id` - This branch's node id, which keys the sealed row.

#### Errors

Returns the delete error.

## Constants

### DESKTOP_SESSION_TTL

```rust
pub const DESKTOP_SESSION_TTL: Duration = _;
```

How long a window session lasts before the window must be signed in
again. The window renews it on every launch, so only a computer left
unopened this long signs out.
