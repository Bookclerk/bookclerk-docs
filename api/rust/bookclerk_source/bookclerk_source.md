**bookclerk_source**

# Module: bookclerk_source

## Contents

**Modules**

- [`abi`](#abi) - Conversions between host storefront types and the typed Cap'n Proto plugin

---

## Module: abi

Conversions between host storefront types and the typed Cap'n Proto plugin
ABI ([`bookclerk_plugin_abi`](../bookclerk_plugin_abi/bookclerk_plugin_abi.md)).

# Audience

The plugin host (`ExternalSource`) and every native storefront guest. Both
sides speak the same generated structs, so the mapping lives here once
instead of being copied into each plugin crate.

Wire optionality follows the ABI convention: empty text / zero numbers mean
"absent" and project to `Option::None` on this side.



