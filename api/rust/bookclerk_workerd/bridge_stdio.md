**bookclerk_workerd > bridge_stdio**

# Module: bridge_stdio

## Contents

**Enums**

- [`Backend`](#backend) - Data plane behind the workerd control plane.

**Functions**

- [`mediate_bridge_stdio`](#mediate_bridge_stdio) - Serves Bookclerk capnp on stdio while talking HTTP/JSRPC to the isolate.

---

## bookclerk_workerd::bridge_stdio::Backend

*Enum*

Data plane behind the workerd control plane.

**Variants:**
- `Author` - Author isolate (`runtime = "workerd"`): every entrypoint call is a
- `Native(bookclerk_plugin_abi::PluginClient)` - Verified native guest (inherited `BOOKCLERK_GATEWAY_GUEST_RPC`): entrypoint calls



## bookclerk_workerd::bridge_stdio::mediate_bridge_stdio

*Function*

Serves Bookclerk capnp on stdio while talking HTTP/JSRPC to the isolate.

Must run inside a `LocalSet` (same thread as the granted HTTP server).
`capabilities` come from the verified install receipt and covering grant
and decide which [`Entrypoints`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesentrypoints) `open` exports; `backend` selects the
data plane.

# Errors

Returns a plugin error when the vat fails.

```rust
fn mediate_bridge_stdio(http: crate::bridge_http::BridgeHttp, table: crate::granted::GrantedTable, capabilities: bookclerk_plugin_abi::PluginCapabilities, backend: Backend) -> anyhow::Result<()>
```



