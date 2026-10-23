# bookclerk_plugin_sdk.cli

CLI entry: `bookclerk-plugin` / `python -m bookclerk_plugin_sdk`.

Dispatches authoring helpers (`check`, `fmt`, `types`, `sync-embed`,
`package`, `smoke`) that mirror the Rust/TypeScript plugin toolchains.

## Functions

### main

```python
def main(argv: list[str] | None=None) -> int
```

Run the `bookclerk-plugin` CLI.

**Args**

- `argv`: Argument vector excluding the program name. Defaults to `sys.argv[1:]`.

**Returns**

Process exit code (`0` success, `1` command failure, `2` usage).

**Examples**

```python
>>> # python -m bookclerk_plugin_sdk check ./my-plugin
>>> # main(["check", "./my-plugin"])
```
