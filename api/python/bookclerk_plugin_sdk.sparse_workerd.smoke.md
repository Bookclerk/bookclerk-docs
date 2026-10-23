# bookclerk_plugin_sdk.sparse_workerd.smoke

Out-of-tree workerd plugin smoke: ensure → materialize → describe + health.

Spawns the pinned Cloudflare `workerd` with a materialized Cap'n Proto config
and POSTs `describe` (plus a `POST /invoke` `health` call for storefront /
remoteLibrary guests) to the HTTP bridge. Does not require the Rust
`bookclerk-workerd` binary.

## Functions

### run_smoke

```python
def run_smoke(plugin_dir: Path) -> str
```

Smoke a workerd plugin without the Rust `bookclerk-workerd` binary.

**Args**

- `plugin_dir`: Path to a `runtime = "workerd"` plugin root.

**Returns**

Multi-line status including plugin id and JSON describe/health detail.

**Raises**

- `FileNotFoundError`: If `plugin.toml` is missing.
- `ValueError`: If the manifest is invalid or not a workerd plugin.
- `TimeoutError`: If the bridge health endpoint never becomes ready.
- `RuntimeError`: If the describe/health bridge call fails.

**Examples**

```python
>>> # print(run_smoke(Path("./my-workerd-plugin")))
```
