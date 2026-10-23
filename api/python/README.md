# Python plugin SDK

The `bookclerk-plugin-sdk` package. Start with
[`bookclerk_plugin_sdk.workerd`](bookclerk_plugin_sdk.workerd.md), which holds the entrypoint classes a plugin extends.

| Module | Summary |
| --- | --- |
| [`bookclerk_plugin_sdk`](bookclerk_plugin_sdk.md) | Bookclerk Python plugin guest SDK. |
| [`bookclerk_plugin_sdk.abi`](bookclerk_plugin_sdk.abi.md) | GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp. |
| [`bookclerk_plugin_sdk.cli`](bookclerk_plugin_sdk.cli.md) | CLI entry: `bookclerk-plugin` / `python -m bookclerk_plugin_sdk`. |
| [`bookclerk_plugin_sdk.db_value`](bookclerk_plugin_sdk.db_value.md) | Universal Cap'n database value domain (`DbValue`) and unpacked codec. |
| [`bookclerk_plugin_sdk.guest_sql`](bookclerk_plugin_sdk.guest_sql.md) | Guest SQL classification — must match host `guest_statement_kind`. |
| [`bookclerk_plugin_sdk.path_guard`](bookclerk_plugin_sdk.path_guard.md) | Path containment helpers for authoring tools and sparse workerd launchers. |
| [`bookclerk_plugin_sdk.plugin_migration`](bookclerk_plugin_sdk.plugin_migration.md) | Resource bounds for plugin-owned `databaseMigrations` registration. |
| [`bookclerk_plugin_sdk.sparse_workerd`](bookclerk_plugin_sdk.sparse_workerd.md) | Sparse out-of-tree workerd launcher (no Rust `bookclerk-workerd` binary). |
| [`bookclerk_plugin_sdk.sparse_workerd.config`](bookclerk_plugin_sdk.sparse_workerd.config.md) | Materialize Cap'n Proto workerd config + bridge (mirrors config.rs). |
| [`bookclerk_plugin_sdk.sparse_workerd.ensure`](bookclerk_plugin_sdk.sparse_workerd.ensure.md) | Download / refresh the pinned Cloudflare `workerd` binary (mirrors ensure.rs). |
| [`bookclerk_plugin_sdk.sparse_workerd.smoke`](bookclerk_plugin_sdk.sparse_workerd.smoke.md) | Out-of-tree workerd plugin smoke: ensure → materialize → describe + health. |
| [`bookclerk_plugin_sdk.tools`](bookclerk_plugin_sdk.tools.md) | check / fmt / types / package / sync-embed — mirrors Rust/TS author tools. |
| [`bookclerk_plugin_sdk.workerd`](bookclerk_plugin_sdk.workerd.md) | Workerd author runtime — `BookclerkEntrypoint` and the named entrypoints. |
