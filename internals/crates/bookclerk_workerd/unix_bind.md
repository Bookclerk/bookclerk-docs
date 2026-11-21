# bookclerk_workerd::unix_bind

Unix-domain listeners that stay under `sockaddr_un` limits.

A host's temp or plugin-state directory can sit under a long prefix (a
checkout under `/home/runner/work/bookclerk/bookclerk/` on GitHub Actions is
already ~38 bytes). Plugin scratch plus a workerd leaf used to overflow
Linux's 108-byte `sun_path`.

Linux GRANTED (workerd ↔ launcher) stays `unix-abstract:` — workerd is a
child of this launcher, not a sibling guest. Other Unix: bind a pathname
under the host-chosen session dir and pass a **relative** `unix:granted.sock`
to workerd (cwd is that directory).

Native-behind-workerd no longer binds a named socket-proxy listener. The
host delivers an inherited duplex (`fd:` / `handle:`) and
[`crate::socket_proxy::spawn_link`](socket_proxy.md#spawn_link) multiplexes CONNECT streams over it.

Do not fall back to `std::env::temp_dir()`: inside `bookclerk-workerd` that
is the process scratch (`TMPDIR`).

| Item | Kind | Summary |
| --- | --- | --- |
| [`clear_cloexec`](#clear_cloexec) | function | Clears `FD_CLOEXEC` so a spawned child inherits `fd`. |
| [`set_cloexec`](#set_cloexec) | function | Sets `FD_CLOEXEC` so a later `workerd` spawn cannot keep a sibling link open. |
| [`random_abstract_name`](#random_abstract_name) | function | Random Linux abstract name (`bc-{kind}-{16 hex}`), always far under `sun_path`. |
| [`bind_granted`](#bind_granted) | function | Binds the adapter-private GRANTED channel. |
| [`bind_rpc`](#bind_rpc) | function | Binds the bridge RPC listener workerd inherits with `--socket-fd`. |
| [`bind_egress`](#bind_egress) | function | Binds the egress relay listener workerd's `hostegress` service dials. |
| [`connect_pathname`](#connect_pathname) | function | Connects to a pathname Unix socket, falling back like `bind_pathname` when the absolute path exceeds `sun_path`. |
| [`bind_abstract`](#bind_abstract) | function | Binds a Linux abstract-namespace Unix listener (`sun_path[0] == NUL`). |
| [`bind_pathname`](#bind_pathname) | function | Binds a pathname socket, falling back when the absolute path exceeds `sun_path`. |
| [`GRANTED_SOCK_FILE`](#granted_sock_file) | constant | Pathname leaf for the GRANTED channel on non-Linux Unix. |
| [`GRANTED_RELATIVE_ADDR`](#granted_relative_addr) | constant | workerd / KJ address for a relative GRANTED pathname (cwd = state dir). |
| [`RPC_SOCK_FILE`](#rpc_sock_file) | constant | Pathname leaf for the bridge RPC socket on non-Linux Unix. |
| [`EGRESS_SOCK_FILE`](#egress_sock_file) | constant | Pathname leaf for the egress relay socket on non-Linux Unix. |

## Functions

### clear_cloexec

```rust
pub fn clear_cloexec(fd: RawFd) -> Result<()>
```

Clears `FD_CLOEXEC` so a spawned child inherits `fd`.

Used for the workerd `--socket-fd` RPC listener.

#### Errors

Returns when `fcntl` `F_GETFD` / `F_SETFD` fails.

### set_cloexec

```rust
pub fn set_cloexec(fd: RawFd) -> Result<()>
```

Sets `FD_CLOEXEC` so a later `workerd` spawn cannot keep a sibling link open.

#### Errors

Returns when `fcntl` `F_GETFD` / `F_SETFD` fails.

### random_abstract_name

```rust
pub fn random_abstract_name(kind: &str) -> String
```

Random Linux abstract name (`bc-{kind}-{16 hex}`), always far under `sun_path`.

### bind_granted

```rust
pub fn bind_granted(state_dir: &Path) -> Result<(String, UnixListener)>
```

Binds the adapter-private GRANTED channel.

Linux returns a `unix-abstract:` address for workerd `external`. Other Unix
binds [`GRANTED_SOCK_FILE`](#granted_sock_file) under `state_dir` and returns
[`GRANTED_RELATIVE_ADDR`](#granted_relative_addr) (workerd's cwd is that directory).

#### Errors

Returns an I/O error when the listener cannot be bound.

### bind_rpc

```rust
pub fn bind_rpc(state_dir: &Path) -> Result<(BridgeAddr, UnixListener)>
```

Binds the bridge RPC listener workerd inherits with `--socket-fd`.

Linux binds an abstract name; other Unix binds [`RPC_SOCK_FILE`](#rpc_sock_file) under
`state_dir`.

#### Arguments

* `state_dir` - The session's state directory.

#### Returns

The address the launcher connects to and the listener to hand workerd.

#### Errors

Returns an I/O error when the listener cannot be bound.

### bind_egress

```rust
pub fn bind_egress(state_dir: &Path) -> Result<(String, UnixListener)>
```

Binds the egress relay listener workerd's `hostegress` service dials.

Linux returns a `unix-abstract:` address; other Unix binds
[`EGRESS_SOCK_FILE`](#egress_sock_file) under `state_dir` and returns `unix:egress.sock`
(workerd's cwd is that directory).

#### Arguments

* `state_dir` - The session's state directory.

#### Returns

The workerd `external` address and the listener the relay accepts on.

#### Errors

Returns an I/O error when the listener cannot be bound.

### connect_pathname

```rust
pub async fn connect_pathname(path: &Path) -> Result<UnixStream>
```

Connects to a pathname Unix socket, falling back like [`bind_pathname`](#bind_pathname)
when the absolute path exceeds `sun_path`.

#### Errors

Returns an I/O error when the connection is refused or the path has no
parent or file name.

### bind_abstract

```rust
pub fn bind_abstract(name: &[u8]) -> Result<UnixListener>
```

Binds a Linux abstract-namespace Unix listener (`sun_path[0] == NUL`).

#### Errors

Returns an I/O error when the name is empty/too long or `bind(2)` fails.

### bind_pathname

```rust
pub fn bind_pathname(path: &Path) -> Result<UnixListener>
```

Binds a pathname socket, falling back when the absolute path exceeds `sun_path`.

On Linux the fallback binds via `/proc/self/fd/{dirfd}/name` so the inode
still lands at `path`. Peers that `connect()` must use a short name (relative
path, `/proc/self/fd`, or abstract) — the absolute path remains too long.

#### Errors

Returns an I/O error when the parent cannot be created or `bind(2)` fails.

## Constants

### GRANTED_SOCK_FILE

```rust
pub const GRANTED_SOCK_FILE: &str = "granted.sock";
```

Pathname leaf for the GRANTED channel on non-Linux Unix.

### GRANTED_RELATIVE_ADDR

```rust
pub const GRANTED_RELATIVE_ADDR: &str = "unix:granted.sock";
```

workerd / KJ address for a relative GRANTED pathname (cwd = state dir).

### RPC_SOCK_FILE

```rust
pub const RPC_SOCK_FILE: &str = "rpc.sock";
```

Pathname leaf for the bridge RPC socket on non-Linux Unix.

### EGRESS_SOCK_FILE

```rust
pub const EGRESS_SOCK_FILE: &str = "egress.sock";
```

Pathname leaf for the egress relay socket on non-Linux Unix.
