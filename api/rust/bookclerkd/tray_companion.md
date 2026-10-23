**bookclerkd > tray_companion**

# Module: tray_companion

## Contents

**Functions**

- [`maybe_spawn_tray`](#maybe_spawn_tray) - Start the tray companion on a background OS thread when appropriate.
- [`update_tray_after_reload`](#update_tray_after_reload) - Refresh tray listen URL + operator auth after a successful reload / rebind.

---

## bookclerkd::tray_companion::maybe_spawn_tray

*Function*

Start the tray companion on a background OS thread when appropriate.

Returns a shared config handle so the daemon can refresh `base_url` / token
after a listen rebind or auth reload. No-ops (returns `None`) when:
- `[daemon].tray = false` / `BOOKCLERK_NO_TRAY` / `BOOKCLERK_DAEMON_TRAY=0`
- no windowing / session-bus environment (typical systemd/Docker hosts)

```rust
fn maybe_spawn_tray(config: &bookclerk_config::Config, auth: &crate::auth::OperatorAuthState) -> Option<bookclerk_tray::SharedTrayConfig>
```



## bookclerkd::tray_companion::update_tray_after_reload

*Function*

Refresh tray listen URL + operator auth after a successful reload / rebind.

```rust
fn update_tray_after_reload(tray: &bookclerk_tray::SharedTrayConfig, config: &bookclerk_config::Config, auth: &crate::auth::OperatorAuthState)
```



