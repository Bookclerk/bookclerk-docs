# bookclerk_loopback_probe

Same-AppContainer loopback probe (experiment E3).

Binds `127.0.0.1:0`, spawns a child of this binary that connects back, and
completes a four-byte echo. Used by `tests/windows_loopback.rs`.

## Modules

