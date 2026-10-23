# bookclerk_plugin_sdk

Guest-only Bookclerk plugin SDK.

# Audience

Third-party **Rust plugin authors**. Depend on this crate (path/git today;
crates.io later) — not the host `bookclerk-plugin-host` crate, which pulls
library/config and other Bookclerk internals.

# What to use

| Need | Entry point |
| --- | --- |
| Native guest (`runtime = "native"`) | [`PluginWorker`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolespluginworker) / [`serve`](bookclerk_plugin_sdk.md#bookclerk_plugin_sdkserve) (`api_version = 3`) |
| Fetch / upload work paths | [`fetch_work_dir`](fetch_dir.md#bookclerk_plugin_sdkfetch_dirfetch_work_dir), [`upload_file_path`](fetch_dir.md#bookclerk_plugin_sdkfetch_dirupload_file_path) |
| Mediated TCP sockets | [`connect_socket`](net.md#bookclerk_plugin_sdknetconnect) / [`net`](net.md) (Workers `connect()` equivalent) |
| Native HTTPS through the socket proxy | feature `http` → `http::Client` |
| OAuth callback without guest listen | [`callback_tunnel`](callback_tunnel.md) |
| Workerd / Wasm guests | [`workerd`](workerd.md) + npm `@bookclerk/plugin-sdk` |
| ABI DTOs / method names | [`protocol`](protocol.md) (re-exports `bookclerk-plugin-abi`) |
| Database guest session / atomic execution | feature `db` → [`database_adapter`](database_adapter.md) |
| Author CLI (`check` / `fmt` / `sync-embed` / `package` / `smoke`) | separate crate `bookclerk-plugin-tools` (`cargo plugin`) |

```toml
# In a standalone plugin repo / workspace:
[dependencies]
bookclerk-plugin-sdk = { git = "https://github.com/Bookclerk/bookclerk", package = "bookclerk-plugin-sdk" }
# Database guests also need:
# bookclerk-plugin-sdk = { …, features = ["db"] }
```

Authoring helpers (`check` / `fmt` / `sync-embed` / `package` / `smoke`)
live in the separate `bookclerk-plugin-tools` crate so guests never link
`bookclerk-workerd`:

```bash
cargo plugin -- check .   # alias for `cargo run -p bookclerk-plugin-tools --bin bookclerk-plugin --`
cargo plugin -- smoke .
```

# API documentation

Generated rustdoc for this crate is part of the workspace API reference
(`./scripts/generate-api-docs.sh` → `docs/api/rust/`). Style guide:
`docs/code-documentation.md`. Product narrative: `docs/plugins.md`,
`docs/plugin-registry.md`.

# Re-exports

Crate-root `pub use` items are the stable import surface for guests
(describe-metadata DTOs, [`PluginError`](../bookclerk_plugin_abi/error.md#bookclerk_plugin_abierrorpluginerror), fetch path helpers, tunnel types). Prefer
`use bookclerk_plugin_sdk::…` over reaching into submodules unless you need
module-level docs.

## Modules

### [`bookclerk_plugin_sdk`](bookclerk_plugin_sdk.md)

*1 function, 7 modules*

### [`callback_tunnel`](callback_tunnel.md)

*1 type alias, 2 structs*

### [`database_adapter`](database_adapter.md)

*2 modules*

### [`database_adapter::errors`](database_adapter/errors.md)

*2 functions*

### [`database_adapter::migrate`](database_adapter/migrate.md)

*1 function*

### [`db`](db.md)

*2 functions*

### [`db_binding`](db_binding.md)

*2 functions, 7 structs*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`fetch_dir`](fetch_dir.md)

*2 functions, 2 structs*

### [`http`](http.md)

*1 function, 1 module, 1 trait, 1 type alias, 7 structs*

### [`http::redirect`](http/redirect.md)

*1 enum*

### [`json`](json.md)

*4 functions*

### [`manifest_caps`](manifest_caps.md)

*1 function*

### [`mux`](mux.md)

*1 function, 2 structs, 9 constants*

### [`net`](net.md)

*11 constants, 2 enums, 2 functions, 3 structs*

### [`pass_fd`](pass_fd.md)

*2 constants, 2 functions*

### [`protocol`](protocol.md)

*1 module, 2 constants*

### [`workerd`](workerd.md)

*4 constants*

