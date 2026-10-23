**bookclerk_tray**

# Module: bookclerk_tray

## Contents

**Functions**

- [`run_blocking`](#run_blocking) - Run the tray on the current thread until the user chooses Quit tray.
- [`spawn`](#spawn) - Start the tray on a dedicated OS thread (safe beside the Tokio HTTP runtime).

---

## bookclerk_tray::run_blocking

*Function*

Run the tray on the current thread until the user chooses Quit tray.

# Arguments

* `config` - Loaded Bookclerk configuration.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn run_blocking(config: SharedTrayConfig) -> anyhow::Result<()>
```



## bookclerk_tray::spawn

*Function*

Start the tray on a dedicated OS thread (safe beside the Tokio HTTP runtime).

# Arguments

* `config` - Loaded Bookclerk configuration.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn spawn(config: SharedTrayConfig) -> std::thread::JoinHandle<anyhow::Result<()>>
```



