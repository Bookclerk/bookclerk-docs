# bookclerk_plugin_tools

Plugin authoring tools behind the `bookclerk-plugin` CLI: `check`, `fmt`,
`sync-embed`, `package`, and `smoke`.

Audience: plugin maintainers validating `plugin.toml` layout and packaging
archives for distribution. Kept out of `bookclerk-plugin-sdk` so guest
plugins never link `bookclerk-workerd` or the packaging stack.

| Helper | Purpose |
| --- | --- |
| [`check_plugin`](check.md#bookclerk_plugin_toolscheckcheck_plugin) / [`fmt_plugin_toml`](bookclerk_plugin_tools.md#bookclerk_plugin_toolsfmt_plugin_toml) / [`package_plugin`](package.md#bookclerk_plugin_toolspackagepackage_plugin) | Validate, format, archive |
| [`sync_embed`](check.md#bookclerk_plugin_toolschecksync_embed) | Optional workerd JS vendor |
| [`run_tools_cli`](cli.md#bookclerk_plugin_toolsclirun) / [`smoke_plugin`](smoke.md#bookclerk_plugin_toolssmokesmoke_plugin) | CLI entry + live workerd smoke |

See `docs/plugins.md` and the crate README.

## Modules

### [`bookclerk_plugin_tools`](bookclerk_plugin_tools.md)

*2 functions*

### [`check`](check.md)

*1 enum, 4 functions*

### [`cli`](cli.md)

*1 function*

### [`package`](package.md)

*1 function*

### [`smoke`](smoke.md)

*1 function*

