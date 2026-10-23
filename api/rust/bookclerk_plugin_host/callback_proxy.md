**bookclerk_plugin_host > callback_proxy**

# Module: callback_proxy

## Contents

**Structs**

- [`CallbackProxy`](#callbackproxy) - Active host callback proxy for one OAuth session.

---

## bookclerk_plugin_host::callback_proxy::CallbackProxy

*Struct*

Active host callback proxy for one OAuth session.

**Fields:**
- `public_base: String` - Browser-facing origin (`http://127.0.0.1:<port>`) advertised to the guest.
- `ipc_endpoint: String` - Unix socket path or Windows pipe name the guest LoginServer connects to.

**Methods:**

- `fn start(callback_bind: Option<&str>, scratch: &Path, package_sid: Option<&str>) -> Result<Self>` - Bind the browser TCP listener and the guest IPC endpoint, then spawn
- `fn bind_addr(self: &Self) -> SocketAddr` - Bound TCP address the browser (and guest) should use for the callback.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`



