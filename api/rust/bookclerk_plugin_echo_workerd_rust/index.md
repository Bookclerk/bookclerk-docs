# bookclerk_plugin_echo_workerd_rust

Echo workerd guest logic compiled to Wasm.

JS `modules/index.js` loads this module and forwards Workers RPC methods.
Identity comes from `plugin.toml` (`api_version = 3`) via the launcher's
manifest projection; Wasm still implements health / diagnose / onEvent / CLI.

## Functions

### dispatch_json

```rust
pub fn dispatch_json(method: &str, params_json: &str) -> Result<String, String>
```

Dispatches one Workers RPC method and returns the JSON result payload.

Called from the JS `BookclerkEntrypoint` / `Cli` glue (and unit tests). Unknown methods
return `Err`; successful handlers serialize ABI DTOs as JSON text.

#### Arguments

* `method` - Workers RPC method name (`health`, `cliInvoke`, …).
* `params_json` - JSON params object, or empty/`null` when the method takes none.

#### Returns

JSON text of the method result (may be the literal `null`).

#### Errors

Returns an error string when params fail to parse, serialization fails, or
the method is unsupported.
