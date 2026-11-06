# bookclerk_plugin_destination_local::plugin

Destination + job-handler surface for the local filesystem guest.

| Item | Kind | Summary |
| --- | --- | --- |
| [`LocalDestination`](#localdestination) | struct | Local filesystem destination capability. |
| [`LocalRoot`](#localroot) | struct | Root capability for the platform local destination guest. |

## Structs

### LocalDestination

```rust
pub struct LocalDestination {
    /* private fields */
}
```

Local filesystem destination capability.

#### LocalDestination::from_config

```rust
pub fn from_config(config: &ExtensibleConfig) -> Result<Self, PluginError>
```

Builds a destination from the `open` `Bindings::config` payload
(an [`OutputLocalContextDto`](../bookclerk_plugin_abi/kind.md#outputlocalcontextdto) as `application/json`; empty means defaults).

##### Errors

Returns `PluginError::invalid_params` when the payload is not a local
output context, or an internal error when the root cannot be opened.

Implements: `Destination`

### LocalRoot

```rust
pub struct LocalRoot;
```

Root capability for the platform local destination guest.

Implements: `PluginWorker`
