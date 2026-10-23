# bookclerk_plugin_e2e

Support for the staged first-party plugin end-to-end suite.

The suite (`tests/staged_plugins.rs`) spawns every guest from a real
installation: platform guests under `$BOOKCLERK_FILES_DIR/plugins/`
(`cargo install-platform`) and optional/example guests under
`$BOOKCLERK_PLUGIN_ARTIFACTS` (`cargo stage-plugins`). It lives in its own
package so ordinary `bookclerk-plugin-host` test runs never need that
installation.

`tests/native_gateway.rs` is separate: it needs no staged installation and
launches the test-only `native_gateway_probe` bin through the production
native-behind-workerd path (CI check `native_gateway`, all three OSes).

| Variable | Effect |
| --- | --- |
| `BOOKCLERK_PLUGIN_ARTIFACTS` / `BOOKCLERK_FILES_DIR` | Staging roots; unset skips locally |
| `BOOKCLERK_STAGED_PLUGINS=a,b` | Check only these ids (plus platform guests); unset checks all |
| `BOOKCLERK_REQUIRE_STAGED_PLUGINS=1` | Missing roots or ledgers fail instead of skipping (CI) |

## Modules

### [`bookclerk_plugin_e2e`](bookclerk_plugin_e2e.md)

*1 enum, 4 functions, 5 constants*

