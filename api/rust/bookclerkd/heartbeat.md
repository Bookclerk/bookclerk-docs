**bookclerkd > heartbeat**

# Module: heartbeat

## Contents

**Functions**

- [`spawn_heartbeat`](#spawn_heartbeat) - Starts the heartbeat loop. The first heartbeat is written at once.

---

## bookclerkd::heartbeat::spawn_heartbeat

*Function*

Starts the heartbeat loop. The first heartbeat is written at once.

```rust
fn spawn_heartbeat(state: std::sync::Arc<crate::api::AppState>)
```



