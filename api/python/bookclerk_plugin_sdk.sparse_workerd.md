# bookclerk_plugin_sdk.sparse_workerd

Sparse out-of-tree workerd launcher (no Rust `bookclerk-workerd` binary).

Downloads the pinned Cloudflare `workerd`, materializes Cap'n Proto + bridge
assets, and runs describe/health smoke tests. Public helpers are re-exported
from [`.config`](bookclerk_plugin_sdk.sparse_workerd.config.md), [`.ensure`](bookclerk_plugin_sdk.sparse_workerd.ensure.md), and [`.smoke`](bookclerk_plugin_sdk.sparse_workerd.smoke.md).

## Re-exports

- [`GeneratedConfig`](bookclerk_plugin_sdk.sparse_workerd.config.md#generatedconfig) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`allocate_workerd_state_dir`](bookclerk_plugin_sdk.sparse_workerd.config.md#allocate_workerd_state_dir) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`binary_name`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#binary_name) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`default_cache_dir`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#default_cache_dir) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`egress_domains_for`](bookclerk_plugin_sdk.sparse_workerd.config.md#egress_domains_for) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`ensure_workerd`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#ensure_workerd) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`load_pin`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#load_pin) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`materialize_config`](bookclerk_plugin_sdk.sparse_workerd.config.md#materialize_config) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`package_root`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#package_root) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`platform_key`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#platform_key) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`plugin_global_outbound`](bookclerk_plugin_sdk.sparse_workerd.config.md#plugin_global_outbound) from `bookclerk_plugin_sdk.sparse_workerd.config`
- [`resolve_under`](bookclerk_plugin_sdk.path_guard.md#resolve_under) from `bookclerk_plugin_sdk.path_guard`
- [`run_smoke`](bookclerk_plugin_sdk.sparse_workerd.smoke.md#run_smoke) from `bookclerk_plugin_sdk.sparse_workerd.smoke`
- [`validate_fetch_url`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#validate_fetch_url) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
- [`validate_spawn_executable`](bookclerk_plugin_sdk.sparse_workerd.ensure.md#validate_spawn_executable) from `bookclerk_plugin_sdk.sparse_workerd.ensure`
