**bookclerk_plugin_sdk > workerd**

# Module: workerd

## Contents

**Constants**

- [`DEFAULT_MAIN_MODULE`](#default_main_module) - Suggested main module filename for JS+Wasm workerd guests.
- [`DEFAULT_MODULES_DIR`](#default_modules_dir) - Suggested modules directory name relative to the plugin root.
- [`EMBED_BOOKCLERK_PLUGIN_JS`](#embed_bookclerk_plugin_js) - Filename historically used by optional `sync-embed` vendors.
- [`EMBED_BOOKCLERK_PLUGIN_JS_SRC`](#embed_bookclerk_plugin_js_src) - Source text of the embeddable workerd runtime helper.

---

## bookclerk_plugin_sdk::workerd::DEFAULT_MAIN_MODULE

*Constant*: `&str`

Suggested main module filename for JS+Wasm workerd guests.

Matches the common `plugin.toml` `[workerd] main_module = "index.js"` default
and the layout under [`DEFAULT_MODULES_DIR`](workerd.md#bookclerk_plugin_sdkworkerddefault_modules_dir).



## bookclerk_plugin_sdk::workerd::DEFAULT_MODULES_DIR

*Constant*: `&str`

Suggested modules directory name relative to the plugin root.

Matches the `plugin.toml` `[workerd] modules_dir` default (`"modules"`).



## bookclerk_plugin_sdk::workerd::EMBED_BOOKCLERK_PLUGIN_JS

*Constant*: `&str`

Filename historically used by optional `sync-embed` vendors.

Prefer importing `@bookclerk/plugin-sdk/workerd` (injected by
`bookclerk-workerd`) over shipping this file next to guest modules.



## bookclerk_plugin_sdk::workerd::EMBED_BOOKCLERK_PLUGIN_JS_SRC

*Constant*: `&str`

Source text of the embeddable workerd runtime helper.

Contains the `BookclerkEntrypoint` base class, the named `*Entrypoint`
bases, and the `RpcTarget` stream classes mirrored by the npm package. Written to
`modules/@bookclerk/plugin-sdk/workerd.js` by `bookclerk-plugin sync-embed`
(crate `bookclerk-plugin-tools`) when authors need an offline vendor.



