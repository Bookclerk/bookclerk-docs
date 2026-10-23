# bookclerk_plugin_echo_workerd_rust

Echo workerd guest logic compiled to Wasm.

JS `modules/index.js` loads this module and forwards Workers RPC methods.
Identity comes from `plugin.toml` (`api_version = 3`) via the launcher's
manifest projection; Wasm still implements health / diagnose / onEvent / CLI.

## Modules

### [`bookclerk_plugin_echo_workerd_rust`](bookclerk_plugin_echo_workerd_rust.md)

*1 function*

