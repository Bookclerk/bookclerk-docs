**bookclerk_dev > ui**

# Module: ui

## Contents

**Functions**

- [`ensure_ui_dist`](#ensure_ui_dist) - Rebuilds `ui/dist` when `index.html` is missing or older than SPA sources.

---

## bookclerk_dev::ui::ensure_ui_dist

*Function*

Rebuilds `ui/dist` when `index.html` is missing or older than SPA sources.

`cargo dev` previously compiled only Rust crates, so a `git pull` that
changed `ui/src` still served the previous Vite output from `ui/dist`.

# Arguments

* `root` - Cargo workspace root (the directory that contains `ui/`).

# Errors

Returns an error when `npm ci` / `npm run build` fail, `npm` is missing, or
`ui/dist/index.html` is still absent after the build.

```rust
fn ensure_ui_dist(root: &std::path::Path) -> anyhow::Result<()>
```



