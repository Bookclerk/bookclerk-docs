# bookclerk_plugin_sdk.sparse_workerd

Sparse out-of-tree workerd launcher (no Rust `bookclerk-workerd` binary).

Downloads the pinned Cloudflare `workerd`, materializes Cap'n Proto + bridge
assets, and runs describe/health smoke tests. Public helpers are re-exported
from [`.config`](config.md), [`.ensure`](ensure.md), and [`.smoke`](smoke.md).

## Re-exports

- [`GeneratedConfig`](config.md#generatedconfig) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`allocate_workerd_state_dir`](config.md#allocate_workerd_state_dir) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`binary_name`](ensure.md#binary_name) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`default_cache_dir`](ensure.md#default_cache_dir) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`egress_domains_for`](config.md#egress_domains_for) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`ensure_workerd`](ensure.md#ensure_workerd) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`load_pin`](ensure.md#load_pin) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`materialize_config`](config.md#materialize_config) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`package_root`](ensure.md#package_root) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`platform_key`](ensure.md#platform_key) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`plugin_global_outbound`](config.md#plugin_global_outbound) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`resolve_under`](../path_guard.md#resolve_under) from `bookclerk_plugin_sdk.path_guard`
- [`run_smoke`](smoke.md#run_smoke) from `bookclerk_plugin_sdk.sparse_workerd.smoke`
- [`validate_fetch_url`](ensure.md#validate_fetch_url) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`validate_spawn_executable`](ensure.md#validate_spawn_executable) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
