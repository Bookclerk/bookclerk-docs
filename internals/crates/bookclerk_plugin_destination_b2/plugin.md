# bookclerk_plugin_destination_b2::plugin

Destination + job-handler surface for the Backblaze B2 guest.

| Item | Kind | Summary |
| --- | --- | --- |
| [`B2Root`](#b2root) | struct | Root capability for the B2 destination guest. |
| [`destination_from`](#destination_from) | function | Builds the destination from the instance's `CONFIG` and `SECRETS`. Nothing is sent to B2 until the first storage call. |

## Structs

### B2Root

```rust
pub struct B2Root;
```

Root capability for the B2 destination guest.

Implements: `PluginWorker`

## Functions

### destination_from

```rust
pub fn destination_from(
    config: &ExtensibleConfig,
    secrets: &ExtensibleConfig,
    hooks: Option<Box<dyn Webhooks>>,
    state_dir: Option<&Path>,
) -> Result<ObjectStoreDestination<B2Backend>, PluginError>
```

Builds the destination from the instance's `CONFIG` and `SECRETS`.
Nothing is sent to B2 until the first storage call.

#### Arguments

* `config` - the instance settings.
* `secrets` - the instance's sealed secrets.
* `hooks` - the instance's webhook binding, when the host granted one.
* `state_dir` - where the change feed keeps its state (under the
  guest's `HOME`); `None` leaves the feed off.

#### Errors

Returns invalid_params for a missing or malformed setting or secret, and
internal when the client or the feed's state directory cannot be made.
