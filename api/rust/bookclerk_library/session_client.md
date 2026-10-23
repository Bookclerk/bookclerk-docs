**bookclerk_library > session_client**

# Module: session_client

## Contents

**Structs**

- [`SessionClientInfo`](#sessionclientinfo) - Parsed client presentation for a session row.

**Functions**

- [`classify_session_client`](#classify_session_client) - Classify a User-Agent (or missing UA for API/bearer clients).

---

## bookclerk_library::session_client::SessionClientInfo

*Struct*

Parsed client presentation for a session row.

**Fields:**
- `user_agent: Option<String>` - Raw User-Agent (or synthetic `API` / empty).
- `device_type: String` - High-level device class: `desktop`, `mobile`, `tablet`, `api`, or `unknown`.
- `client_label: String` - Human label such as `Windows`, `macOS`, `Android`, `iPhone`, or `API`.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &SessionClientInfo) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> SessionClientInfo`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::session_client::classify_session_client

*Function*

Classify a User-Agent (or missing UA for API/bearer clients).

# Arguments

* `user_agent` - Optional `User-Agent` header value.
* `is_api` - When true (no browser UA / bearer-style client), label as API.

# Panics

Panics when an internal invariant does not hold.

```rust
fn classify_session_client(user_agent: Option<&str>, is_api: bool) -> SessionClientInfo
```



