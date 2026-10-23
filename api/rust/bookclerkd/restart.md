**bookclerkd > restart**

# Module: restart

## Contents

**Functions**

- [`is_requested`](#is_requested) - True once [`request`](restart.md#bookclerkdrestartrequest) was called.
- [`reexec`](#reexec) - Starts this binary again with the same arguments and environment.
- [`request`](#request) - Asks the daemon to restart once in-flight requests finish.
- [`requested`](#requested) - Completes when a restart is asked for.

---

## bookclerkd::restart::is_requested

*Function*

True once [`request`](restart.md#bookclerkdrestartrequest) was called.

```rust
fn is_requested() -> bool
```



## bookclerkd::restart::reexec

*Function*

Starts this binary again with the same arguments and environment.

# Errors

Returns the error when the binary cannot be found or started; on Unix a
successful call does not return.

```rust
fn reexec() -> std::io::Result<()>
```



## bookclerkd::restart::request

*Function*

Asks the daemon to restart once in-flight requests finish.

```rust
fn request(reason: &'static str)
```



## bookclerkd::restart::requested

*Function*

Completes when a restart is asked for.

```rust
fn requested()
```



