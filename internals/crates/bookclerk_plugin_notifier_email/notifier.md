# bookclerk_plugin_notifier_email::notifier

The instance's settings and the `notifier` entrypoint.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Settings`](#settings) | struct | The instance's settings, checked. |
| [`EmailNotifier`](#emailnotifier) | struct | `notifier` entrypoint: one SMTP submission per message. |
| [`SERVER_SETTING`](#server_setting) | constant | The key of the setting that names the server. |

## Structs

### Settings

```rust
pub struct Settings {
    pub server: HostPort,
    pub security: Security,
    pub credentials: Option<Credentials>,
    pub from: String,
    pub from_name: String,
    pub to: String,
}
```

The instance's settings, checked.

**Fields**

- `server`: Where the server is, as the host approved it.
- `security`: How the connection is protected.
- `credentials`: Sign-in, when a user name is set.
- `from`: Sender address.
- `from_name`: Sender display name.
- `to`: The `to` setting, unparsed; empty when only patrons' addresses are
  used.

#### Settings::read

```rust
pub fn read(
    config: &Value,
    secrets: &Value,
    network: &NetworkGrant,
) -> Result<Self, PluginError>
```

Reads and checks the settings.

##### Arguments

* `config` - The decoded `CONFIG` document.
* `secrets` - The decoded `SECRETS` document.
* `network` - The network grant in effect.

##### Errors

`invalid_params` when a setting is missing or malformed;
`network_not_granted` while the server's destination awaits
approval.

Implements: `Clone`, `Debug`

### EmailNotifier

```rust
pub struct EmailNotifier<T> {
    /* private fields */
}
```

`notifier` entrypoint: one SMTP submission per message.

#### EmailNotifier::new

```rust
pub fn new(settings: Result<Settings, PluginError>, transport: T) -> Self
```

Builds the entrypoint from the instance's bindings.

##### Arguments

* `settings` - [`Settings::read`](#settingsread)'s answer.
* `transport` - Where connections come from.

#### EmailNotifier::with_fixed

```rust
pub fn with_fixed(self, boundary: &str, now: i64) -> Self
```

Fixes the boundary and clock, so tests see the same message every
run.

##### Arguments

* `boundary` - The MIME boundary.
* `now` - The time, Unix seconds.

Implements: `Notifier`

## Constants

### SERVER_SETTING

```rust
pub const SERVER_SETTING: &str = "server";
```

The key of the setting that names the server.
