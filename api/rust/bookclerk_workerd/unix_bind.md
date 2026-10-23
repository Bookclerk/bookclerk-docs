**bookclerk_workerd > unix_bind**

# Module: unix_bind

## Contents

**Functions**

- [`bind_abstract`](#bind_abstract) - Binds a Linux abstract-namespace Unix listener (`sun_path[0] == NUL`).
- [`bind_granted`](#bind_granted) - Binds the adapter-private GRANTED channel.
- [`bind_pathname`](#bind_pathname) - Binds a pathname socket, falling back when the absolute path exceeds `sun_path`.
- [`clear_cloexec`](#clear_cloexec) - Clears `FD_CLOEXEC` so a spawned child inherits `fd`.
- [`random_abstract_name`](#random_abstract_name) - Random Linux abstract name (`bc-{kind}-{16 hex}`), always far under `sun_path`.
- [`set_cloexec`](#set_cloexec) - Sets `FD_CLOEXEC` so a later `workerd` spawn cannot keep a sibling link open.

**Constants**

- [`GRANTED_RELATIVE_ADDR`](#granted_relative_addr) - workerd / KJ address for a relative GRANTED pathname (cwd = state dir).
- [`GRANTED_SOCK_FILE`](#granted_sock_file) - Pathname leaf for the GRANTED channel on non-Linux Unix.

---

## bookclerk_workerd::unix_bind::GRANTED_RELATIVE_ADDR

*Constant*: `&str`

workerd / KJ address for a relative GRANTED pathname (cwd = state dir).



## bookclerk_workerd::unix_bind::GRANTED_SOCK_FILE

*Constant*: `&str`

Pathname leaf for the GRANTED channel on non-Linux Unix.



## bookclerk_workerd::unix_bind::bind_abstract

*Function*

Binds a Linux abstract-namespace Unix listener (`sun_path[0] == NUL`).

# Errors

Returns an I/O error when the name is empty/too long or `bind(2)` fails.

```rust
fn bind_abstract(name: &[u8]) -> io::Result<std::os::unix::net::UnixListener>
```



## bookclerk_workerd::unix_bind::bind_granted

*Function*

Binds the adapter-private GRANTED channel.

Linux returns a `unix-abstract:` address for workerd `external`. Other Unix
binds [`GRANTED_SOCK_FILE`](unix_bind.md#bookclerk_workerdunix_bindgranted_sock_file) under `state_dir` and returns
[`GRANTED_RELATIVE_ADDR`](unix_bind.md#bookclerk_workerdunix_bindgranted_relative_addr) (workerd's cwd is that directory).

# Errors

Returns an I/O error when the listener cannot be bound.

```rust
fn bind_granted(state_dir: &std::path::Path) -> io::Result<(String, std::os::unix::net::UnixListener)>
```



## bookclerk_workerd::unix_bind::bind_pathname

*Function*

Binds a pathname socket, falling back when the absolute path exceeds `sun_path`.

On Linux the fallback binds via `/proc/self/fd/{dirfd}/name` so the inode
still lands at `path`. Peers that `connect()` must use a short name (relative
path, `/proc/self/fd`, or abstract) — the absolute path remains too long.

# Errors

Returns an I/O error when the parent cannot be created or `bind(2)` fails.

```rust
fn bind_pathname(path: &std::path::Path) -> io::Result<std::os::unix::net::UnixListener>
```



## bookclerk_workerd::unix_bind::clear_cloexec

*Function*

Clears `FD_CLOEXEC` so a spawned child inherits `fd`.

Used for the workerd `--socket-fd` RPC listener.

# Errors

Returns when `fcntl` `F_GETFD` / `F_SETFD` fails.

```rust
fn clear_cloexec(fd: std::os::fd::RawFd) -> io::Result<()>
```



## bookclerk_workerd::unix_bind::random_abstract_name

*Function*

Random Linux abstract name (`bc-{kind}-{16 hex}`), always far under `sun_path`.

```rust
fn random_abstract_name(kind: &str) -> String
```



## bookclerk_workerd::unix_bind::set_cloexec

*Function*

Sets `FD_CLOEXEC` so a later `workerd` spawn cannot keep a sibling link open.

# Errors

Returns when `fcntl` `F_GETFD` / `F_SETFD` fails.

```rust
fn set_cloexec(fd: std::os::fd::RawFd) -> io::Result<()>
```



