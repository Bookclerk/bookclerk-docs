**bookclerk_config > desktop**

# Module: desktop

## Contents

**Functions**

- [`graphical_session_available`](#graphical_session_available) - True when it is reasonable to start the in-process tray (windowing + session bus).

---

## bookclerk_config::desktop::graphical_session_available

*Function*

True when it is reasonable to start the in-process tray (windowing + session bus).

Linux requires a display (`DISPLAY` or `WAYLAND_DISPLAY`) **and** a session-bus
signal (`DBUS_SESSION_BUS_ADDRESS` or `XDG_RUNTIME_DIR`). Windows and macOS
return true; headless CI/servers on Linux return false.

```rust
fn graphical_session_available() -> bool
```



