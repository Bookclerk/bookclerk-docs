**bookclerk_plugin_integration_audiobookshelf > brand**

# Module: brand

## Contents

**Functions**

- [`brand_for_id`](#brand_for_id) - Brand for a provider id when it aliases this plugin.
- [`matches_id`](#matches_id) - Whether `id` refers to this integration (`audiobookshelf` / `abs`).

**Constants**

- [`BRAND`](#brand) - Canonical ABS brand for connect-portal buttons.

---

## bookclerk_plugin_integration_audiobookshelf::brand::BRAND

*Constant*: `bookclerk_integrations::Brand`

Canonical ABS brand for connect-portal buttons.



## bookclerk_plugin_integration_audiobookshelf::brand::brand_for_id

*Function*

Brand for a provider id when it aliases this plugin.

```rust
fn brand_for_id(id: &str) -> Option<bookclerk_integrations::Brand>
```



## bookclerk_plugin_integration_audiobookshelf::brand::matches_id

*Function*

Whether `id` refers to this integration (`audiobookshelf` / `abs`).

```rust
fn matches_id(id: &str) -> bool
```



