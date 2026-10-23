**bookclerk_integrations > factory**

# Module: factory

## Contents

**Functions**

- [`from_config`](#from_config) - Construct an empty registry (no in-process adapters).
- [`register_builtins`](#register_builtins) - Register first-party integrations into an existing registry.

---

## bookclerk_integrations::factory::from_config

*Function*

Construct an empty registry (no in-process adapters).

Hosts that also load plugins should prefer
`bookclerk_plugin_host::load_integrations`.

# Arguments

* `config` - Loaded Bookclerk configuration.

# Returns

On success, the inner `IntegrationRegistry` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn from_config(_config: &bookclerk_config::Config) -> crate::error::Result<crate::registry::IntegrationRegistry>
```



## bookclerk_integrations::factory::register_builtins

*Function*

Register first-party integrations into an existing registry.

This function is intentionally a no-op; ABS and other adapters load as
external guests through `bookclerk_plugin_host::load_integrations`.

# Arguments

* `config` - Loaded Bookclerk configuration.
* `registry` - Configured content-source or integration registry.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn register_builtins(_config: &bookclerk_config::Config, _registry: & mut crate::registry::IntegrationRegistry) -> crate::error::Result<()>
```



