# bookclerk_workerd::bridge_stdio

Bookclerk Cap'n Proto stdio adapter over the workerd HTTP bridge.

Toward the host this process is a [`PluginWorker`](../bookclerk_plugin_abi/index.md#pluginworker). The control plane
(`describe` / `open` / `shutdown`) is a JSON policy handshake with the
adapter isolate; every ABI *method* call on an exported entrypoint is one
`POST /invoke` carrying Cap'n Proto bytes ([`crate::invoke`](invoke.md)), so the typed
clients from `bookclerk-plugin-abi` (`ContentSourceClient`,
`EventConsumerClient`, `JobRunnerClient`, `DatabaseClient`, …) are reused
unchanged over the [`InvokeClient`](invoke.md#invokeclient) hook. Object bodies stream over the
dedicated `/destination/get` / `/destination/put` routes. The exported
[`Entrypoints`](../bookclerk_plugin_abi/index.md#entrypoints) follow the verified `plugin.toml` capabilities (the host
allowlist), never a guest-declared widening. See `docs/workerd-bridge.md`.

With a [`Backend::Native`](#backend) guest the isolate stays the control plane
(`describe` merge, `open` policy, `shutdown`) while every entrypoint
capability is the guest's own typed Cap'n Proto client, forwarded to the
host without entering JavaScript memory.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Backend`](#backend) | enum | Data plane behind the workerd control plane. |
| [`mediate_bridge_stdio`](#mediate_bridge_stdio) | function | Serves Bookclerk capnp on stdio while talking HTTP/JSRPC to the isolate. |

## Enums

### Backend

```rust
pub enum Backend {
    Author,
    Native(PluginClient),
}
```

Data plane behind the workerd control plane.

**Variants**

- `Author`: Author isolate (`runtime = "workerd"`): every entrypoint call is a
  Cap'n Proto `POST /invoke` (or a streamed body route) on the bridge
  worker.
- `Native`: Verified native guest (inherited `BOOKCLERK_GATEWAY_GUEST_RPC`): entrypoint calls
  are typed Cap'n Proto forwards to the guest's own capabilities. The
  client must have been created with `connect_plugin` on this `LocalSet`
  and its RPC future spawned locally.

## Functions

### mediate_bridge_stdio

```rust
pub async fn mediate_bridge_stdio(
    http: BridgeHttp,
    table: GrantedTable,
    capabilities: PluginCapabilities,
    backend: Backend,
) -> Result<()>
```

Serves Bookclerk capnp on stdio while talking HTTP/JSRPC to the isolate.

Must run inside a `LocalSet` (same thread as the granted HTTP server).
`capabilities` come from the verified install receipt and covering grant
and decide which [`Entrypoints`](../bookclerk_plugin_abi/index.md#entrypoints) `open` exports; `backend` selects the
data plane.

#### Errors

Returns a plugin error when the vat fails.
