**bookclerk_plugin_sdk**

# Module: bookclerk_plugin_sdk

## Contents

**Modules**

- [`callback_tunnel`](#callback_tunnel) - Multiplexed byte tunnel for host-owned OAuth callback listeners.
- [`database_adapter`](#database_adapter) - Database adapter helpers for third-party database plugin authors.
- [`http`](#http) - Native guest HTTP through the workerd socket proxy.
- [`mux`](#mux) - Bidirectional multiplexed byte streams with per-stream credit windows.
- [`net`](#net) - Mediated TCP sockets for native guests (Workers `connect()` equivalent).
- [`protocol`](#protocol) - Shared role method names and typed method payloads.
- [`workerd`](#workerd) - Workerd guest bridge — same [`crate::PluginWorker`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolespluginworker) contract as native.

**Functions**

- [`serve`](#serve) - Serves a [`PluginWorker`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolespluginworker) on stdin/stdout (Cap'n Proto RPC).

---

## Module: callback_tunnel

Multiplexed byte tunnel for host-owned OAuth callback listeners.

Thin wrappers around [`crate::mux`]: the host `TunnelHost::open`s one
stream per accepted browser connection; the guest `TunnelGuest::accept`s
them. Flow control is the mux per-stream credit window.



## Module: database_adapter

Database adapter helpers for third-party database plugin authors.

# Audience

Rust authors building a **database** guest (`kind = "database"`). Use this
module with the `db` feature enabled on `bookclerk-plugin-sdk`:

```toml
[dependencies]
bookclerk-plugin-sdk = { version = "0.1", features = ["db"] }
```

# What to use

| Need | Entry point |
| --- | --- |
| Engine error → structured [`crate::PluginError`](../bookclerk_plugin_abi/error.md#bookclerk_plugin_abierrorpluginerror) | [`plugin_error_from_engine`](database_adapter/errors.md#bookclerk_plugin_sdkdatabase_adaptererrorsplugin_error_from_engine), [`plugin_error_from_db_err`](database_adapter/errors.md#bookclerk_plugin_sdkdatabase_adaptererrorsplugin_error_from_db_err) |
| Typed SQL `NULL` for proxy row decode | [`typed_null`](database_adapter/migrate.md#bookclerk_plugin_sdkdatabase_adaptermigratetyped_null) |

First-party platform database guests use the internal `bookclerk-db-guest`
crate for SeaORM session workers; those helpers are not part of the stable
third-party author API.



## Module: http

Native guest HTTP through the workerd socket proxy.

When [`crate::SOCKET_PROXY_ENV`](net.md#bookclerk_plugin_sdknetsocket_proxy_env) is unset (crate tests, host-owned D1),
this module wraps [`reqwest::Client`](https://docs.rs/reqwest/latest/reqwest/async_impl/client/struct.Client.html) over ambient TCP. When the proxy
is set, every request is `CONNECT` + rustls over the Unix or Windows
named-pipe proxy — nested `NetPolicy::Deny` guests must not call
`socket(AF_INET)`. When [`crate::NESTED_NATIVE_JAIL_ENV`](net.md#bookclerk_plugin_sdknetnested_native_jail_env) is `1` without a
proxy, construction fails closed rather than falling back to ambient TCP.

Fetch domain grants do **not** imply TCP. Native manifests must declare
`capabilities.network.tcp` (or rely on a host overlay) for each
`host:port` this client dials.



## Module: mux

Bidirectional multiplexed byte streams with per-stream credit windows.

Used for the native-behind-workerd socket proxy (guest `Mux::open`,
gateway `Mux::accept`) and the OAuth callback tunnel (host opens, guest
accepts). Either side may initiate; connection ids from a client are odd
and from a server are even so they cannot collide.

The peer is untrusted. Frame length is rejected before a payload buffer is
allocated. Each stream buffers at most [`MAX_BUFFERED_PER_STREAM`](mux.md#bookclerk_plugin_sdkmuxmax_buffered_per_stream) unread
bytes, the connection buffers at most [`MAX_AGGREGATE_BUFFERED`](mux.md#bookclerk_plugin_sdkmuxmax_aggregate_buffered), and at
most [`MAX_LIVE_STREAMS`](mux.md#bookclerk_plugin_sdkmuxmax_live_streams) stream objects exist. That cap covers streams
waiting in the accept queue and streams the peer has already closed: a
`Close` frame does not free the slot while the [`MuxStream`](mux.md#bookclerk_plugin_sdkmuxmuxstream) is still
alive. Credit and close records for a generation stay reserved until the
writer drops both, so a blocked writer cannot accumulate them after the
map slot is released. A new Open is refused while either record vector or
that reservation table is already at the cap. Each slot has a generation
so dropping an old stream cannot remove a replacement that reused the id.
Window updates saturate at [`INITIAL_WINDOW`](mux.md#bookclerk_plugin_sdkmuxinitial_window) and are coalesced until the
writer accepts them, so a full control queue cannot drop receive credit.
`Close` is written only after Data already accepted for that same stream.
Window updates and other streams' control frames are not stuck behind one
unread stream. A stream flush or shutdown waits until that stream's
accepted output is written. `Mux::shutdown` and dropping the last mux
cancel both tasks without draining queued frames.

Frame layout (big-endian):
```text
u32 length_of_rest | u8 type | u32 conn_id | payload
```
Types: `Open=1`, `Data=2`, `Close=3`, `Window=4`. Data payloads are capped
at 1 MiB. A `Window` frame carries a `u32` credit (bytes the peer may
send). Each stream starts with [`INITIAL_WINDOW`](mux.md#bookclerk_plugin_sdkmuxinitial_window) bytes of send credit.



## Module: net

Mediated TCP sockets for native guests (Workers `connect()` equivalent).

Native plugins must not call `socket(AF_INET)` / `connect` themselves. The
guest jail denies ambient internet; this module speaks HTTP CONNECT through
`BOOKCLERK_SOCKET_PROXY`, which applies the same
[`bookclerk_plugin_manifest::EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy) as workerd `fetch()`/`connect()`.

Production native-behind-workerd sets `fd:<n>` or `handle:<n>` and multiplexes
CONNECT streams over that inherited link ([`crate::mux`](mux.md)). A per-session
challenge ([`SESSION_CHALLENGE_ENV`](net.md#bookclerk_plugin_sdknetsession_challenge_env)) is written before the first mux frame
so a numeric descriptor is not an identity. Pathname, `abstract:`, and
`\\.\pipe\` forms remain for tests and older launchers. Native plugins must
be rebuilt against this SDK; there is no ambient TCP fallback.



## Module: protocol

Shared role method names and typed method payloads.

Audience: guest authors who need the Cap'n Proto method parameter / result
structs (generated from `schema/plugin.capnp`) without depending on host
crates. Wire types live in `bookclerk_plugin_abi`; this module re-exports
them for a stable SDK import path.

# Trust boundary

External plugins are **untrusted** relative to the host. The host must never
hand them `library.db`, `master.key`, or the Bookclerk files-dir root. Plugins
receive only a scoped `plugin_data_dir` / `cache_dir`, and credentials are
host-mediated (login returns a blob the host seals; scan and fetch receive
that blob from the host). Scan returns book DTOs for the host to upsert.

Prefer product docs under `docs/plugins.md` for jail / capability rules.



## bookclerk_plugin_sdk::serve

*Function*

Serves a [`PluginWorker`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolespluginworker) on stdin/stdout (Cap'n Proto RPC).

Must run on a current-thread tokio runtime inside a `LocalSet` (this helper
creates the `LocalSet`). Abort is capability drop / stream cancel.

# Errors

Returns [`SdkError`](error.md#bookclerk_plugin_sdkerrorsdkerror) when the vat fails.

```rust
fn serve<impl PluginWorker>(plugin: impl Trait) -> Result<()>
```



## Module: workerd

Workerd guest bridge — same [`crate::PluginWorker`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolespluginworker) contract as native.

Audience: authors of `runtime = "workerd"` plugins (JS modules and optional
Rust→Wasm). Dual-stack with native guests:

- **Native:** `use bookclerk_plugin_sdk::{PluginRoot, serve};`
- **Workerd:** JS modules import the package; Rust implements ABI dispatch:

```js
import { BookclerkEntrypoint, CliEntrypoint } from "@bookclerk/plugin-sdk/workerd";
import { initSync, cliInvoke } from "./pkg/….js";
import wasmModule from "./pkg/…_bg.wasm";
initSync({ module: wasmModule });
export class Cli extends CliEntrypoint { /* forwards to wasm */ }
export default class EchoPlugin extends BookclerkEntrypoint { /* event(batch) */ }
```

`bookclerk-workerd` injects `@bookclerk/plugin-sdk/workerd` (and the package
root) into the isolate — authors do not vendor a relative filepath. Optional
offline vendors can copy [`EMBED_BOOKCLERK_PLUGIN_JS_SRC`](workerd.md#bookclerk_plugin_sdkworkerdembed_bookclerk_plugin_js_src) via
`bookclerk-plugin sync-embed` (crate `bookclerk-plugin-tools`).

Layout (see `examples/plugins-echo-workerd-rust/`):

```text
plugin.toml                 runtime = "workerd"
modules/index.js            package import + wasm glue
modules/pkg/*.js + *.wasm   wasm-bindgen output (`./build-wasm.sh`)
src/lib.rs                  ABI-typed `dispatch_json`
```



