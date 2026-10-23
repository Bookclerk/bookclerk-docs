**bookclerk_plugin_tools > smoke**

# Module: smoke

## Contents

**Functions**

- [`smoke_plugin`](#smoke_plugin) - Smokes a `runtime = "workerd"` plugin: ensure pin → materialize → describe + health.

---

## bookclerk_plugin_tools::smoke::smoke_plugin

*Function*

Smokes a `runtime = "workerd"` plugin: ensure pin → materialize → describe + health.

Spawns unconfined `workerd serve` on a free loopback port (jailed guests use
a listen FD via the launcher instead). Kills the child before returning.

# Arguments

* `plugin_dir` - Directory containing `plugin.toml` with `runtime = "workerd"`.

# Returns

Multi-line success text including pretty-printed describe/health JSON.

# Errors

Returns a plain `String` error when the manifest is not workerd, workerd
cannot be ensured, health never becomes ready, or RPC calls fail.

```rust
fn smoke_plugin(plugin_dir: &std::path::Path) -> Result<String, String>
```



