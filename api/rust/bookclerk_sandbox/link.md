**bookclerk_sandbox > link**

# Module: link

## Contents

**Structs**

- [`DuplexHalf`](#duplexhalf) - One unidirectional pipe end (stdio).
- [`DuplexLink`](#duplexlink) - One end of a host-created duplex IPC link (socketpair / overlapped pipe).
- [`JailHandoff`](#jailhandoff) - One-shot Windows jail stdin line. Bounded JSON; never a general broker.
- [`JailHandoffExtra`](#jailhandoffextra) - One extra handle named in the child environment.
- [`StdioEnds`](#stdioends) - Host and guest ends of a pair of unidirectional stdio pipes (RPC).

**Enums**

- [`LinkSpec`](#linkspec) - How a confined child names an inherited duplex or pipe end.
- [`LinkSpecError`](#linkspecerror) - Why a [`LinkSpec`](link.md#bookclerk_sandboxlinklinkspec) string could not be parsed.

**Functions**

- [`inherit_fd_at`](#inherit_fd_at) - `dup2` `src` onto `dest` and clear `FD_CLOEXEC` on the destination.
- [`with_fd_spawn_lock`](#with_fd_spawn_lock) - Serialize macOS descriptor allocation with in-process `Command` spawns.

**Constants**

- [`GATEWAY_GUEST_RPC_ENV`](#gateway_guest_rpc_env) - Env var naming the guest Cap'n Proto link for `bookclerk-workerd`.
- [`GATEWAY_GUEST_RPC_WRITE_ENV`](#gateway_guest_rpc_write_env) - Windows-only env var naming the gateway's overlapped write half of guest RPC.
- [`GATEWAY_PROXY_ENV`](#gateway_proxy_env) - Env var naming the multiplexed CONNECT-proxy link for `bookclerk-workerd`.
- [`GATEWAY_PROXY_FD`](#gateway_proxy_fd) - Child fd the gateway uses for the proxy mux (`BOOKCLERK_GATEWAY_PROXY`).
- [`GATEWAY_PROXY_WRITE_ENV`](#gateway_proxy_write_env) - Windows-only env var naming the gateway's overlapped write half of the proxy.
- [`GATEWAY_RPC_FD`](#gateway_rpc_fd) - Child fd the gateway uses for the guest RPC socket (`BOOKCLERK_GATEWAY_GUEST_RPC`).
- [`GUEST_PROXY_FD`](#guest_proxy_fd) - Child fd the native guest uses for the proxy mux (`BOOKCLERK_SOCKET_PROXY`).
- [`JAIL_HANDOFF_ENV`](#jail_handoff_env) - When `1`, `bookclerk-jail` reads one [`JailHandoff`](link.md#bookclerk_sandboxlinkjailhandoff) JSON line from stdin.
- [`SOCKET_PROXY_ENV`](#socket_proxy_env) - Env var naming the socket-proxy link for a native guest (SDK `net::connect`).
- [`SOCKET_PROXY_WRITE_ENV`](#socket_proxy_write_env) - Windows-only env var naming the guest's overlapped write half of the proxy.
- [`TEST_EXTRA_ENDPOINT_FD`](#test_extra_endpoint_fd) - Test-only extra endpoint in the native guest. Production preserve lists omit it.
- [`WORKERD_STATE_DIR_ENV`](#workerd_state_dir_env) - Host-chosen workerd session directory (`0700`, under `$TMPDIR`).

---

## bookclerk_sandbox::link::DuplexHalf

*Struct*

One unidirectional pipe end (stdio).

**Methods:**

- `fn as_raw_fd(self: &Self) -> std::os::fd::RawFd` - Unix raw fd.
- `fn into_owned_fd(self: Self) -> std::os::fd::OwnedFd` - Consume as [`std::os::fd::OwnedFd`](https://doc.rust-lang.org/stable/std/os/fd/owned/struct.OwnedFd.html).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::link::DuplexLink

*Struct*

One end of a host-created duplex IPC link (socketpair / overlapped pipe).

**Methods:**

- `fn pair() -> io::Result<(Self, Self)>` - Create a connected duplex pair. Both ends stay non-inheritable / `CLOEXEC`.
- `fn try_clone(self: &Self) -> io::Result<Self>` - Duplicate this end. Both copies stay `CLOEXEC` / non-inheritable.
- `fn as_raw_fd(self: &Self) -> std::os::fd::RawFd` - Unix raw fd (still owned by `self`).
- `fn into_owned_fd(self: Self) -> std::os::fd::OwnedFd` - Consume this end as an [`std::os::fd::OwnedFd`](https://doc.rust-lang.org/stable/std/os/fd/owned/struct.OwnedFd.html).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::link::GATEWAY_GUEST_RPC_ENV

*Constant*: `&str`

Env var naming the guest Cap'n Proto link for `bookclerk-workerd`.

Unix: one duplex socket, `fd:<n>`. Windows: the gateway's overlapped read
half (guest stdout → gateway), `handle:<n>`. The write half is
[`GATEWAY_GUEST_RPC_WRITE_ENV`](link.md#bookclerk_sandboxlinkgateway_guest_rpc_write_env).



## bookclerk_sandbox::link::GATEWAY_GUEST_RPC_WRITE_ENV

*Constant*: `&str`

Windows-only env var naming the gateway's overlapped write half of guest RPC.

`handle:<n>` writes into the guest's synchronous stdin. Absent on Unix,
where [`GATEWAY_GUEST_RPC_ENV`](link.md#bookclerk_sandboxlinkgateway_guest_rpc_env) is a full-duplex socket.



## bookclerk_sandbox::link::GATEWAY_PROXY_ENV

*Constant*: `&str`

Env var naming the multiplexed CONNECT-proxy link for `bookclerk-workerd`.

Unix: one duplex socket, `fd:<n>`. Windows: the gateway's overlapped read
half, `handle:<n>`. The write half is [`GATEWAY_PROXY_WRITE_ENV`](link.md#bookclerk_sandboxlinkgateway_proxy_write_env).



## bookclerk_sandbox::link::GATEWAY_PROXY_FD

*Constant*: `i32`

Child fd the gateway uses for the proxy mux (`BOOKCLERK_GATEWAY_PROXY`).



## bookclerk_sandbox::link::GATEWAY_PROXY_WRITE_ENV

*Constant*: `&str`

Windows-only env var naming the gateway's overlapped write half of the proxy.

Absent on Unix, where [`GATEWAY_PROXY_ENV`](link.md#bookclerk_sandboxlinkgateway_proxy_env) is a full-duplex socket.



## bookclerk_sandbox::link::GATEWAY_RPC_FD

*Constant*: `i32`

Child fd the gateway uses for the guest RPC socket (`BOOKCLERK_GATEWAY_GUEST_RPC`).



## bookclerk_sandbox::link::GUEST_PROXY_FD

*Constant*: `i32`

Child fd the native guest uses for the proxy mux (`BOOKCLERK_SOCKET_PROXY`).



## bookclerk_sandbox::link::JAIL_HANDOFF_ENV

*Constant*: `&str`

When `1`, `bookclerk-jail` reads one [`JailHandoff`](link.md#bookclerk_sandboxlinkjailhandoff) JSON line from stdin.



## bookclerk_sandbox::link::JailHandoff

*Struct*

One-shot Windows jail stdin line. Bounded JSON; never a general broker.

The host derives every handle from values it just duplicated with
`duplicate_handle_into` (Windows). The jail is the sole spawner and marks
the inherited entries inheritable immediately before `CreateProcess`.
Entries with `inherit: false` are named in the child environment and are
not placed on the inherit list.

**Fields:**
- `v: u32` - Wire version. Only `1` is accepted.
- `stdin: Option<u64>` - Guest stdin handle (RPC read end), when the jail should not create a pipe.
- `stdout: Option<u64>` - Guest stdout handle (RPC write end), when the jail should not create a pipe.
- `extra: Vec<JailHandoffExtra>` - Extra handles exported as `env=handle:<value>`.

**Methods:**

- `fn inherited_extra_handles(self: &Self) -> Vec<u64>` - Handle values `CreateProcess` should list.
- `fn to_line(self: &Self) -> io::Result<String>` - Encode as a single JSON line (no trailing newline; the caller writes `\n`).
- `fn from_line(line: &str) -> io::Result<Self>` - Parse one JSON line. Rejects `v != 1` and oversize input.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> JailHandoff`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &JailHandoff) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::link::JailHandoffExtra

*Struct*

One extra handle named in the child environment.

**Fields:**
- `env: String` - Environment variable the child should see (`BOOKCLERK_SOCKET_PROXY`, …).
- `handle: u64` - Handle value in the jail process (already duplicated).
- `inherit: bool` - When false, the child learns `handle:<value>` and the handle stays off

**Methods:**

- `fn inherited<impl Into<String>>(env: impl Trait, handle: u64) -> Self` - Inherited handle. The child receives the object and `env=handle:<value>`.
- `fn named_only<impl Into<String>>(env: impl Trait, handle: u64) -> Self` - Name the handle in the child environment without inheriting the object.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> JailHandoffExtra`
- **PartialEq**
  - `fn eq(self: &Self, other: &JailHandoffExtra) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_sandbox::link::LinkSpec

*Enum*

How a confined child names an inherited duplex or pipe end.

**Variants:**
- `Fd(i32)` - Unix inherited file descriptor (`fd:3`).
- `Handle(u64)` - Windows inherited handle value (`handle:18446744073709551615`).

**Methods:**

- `fn parse(spec: &str) -> Result<Self, LinkSpecError>` - Parse `fd:<n>` or `handle:<n>`.
- `fn fd(n: i32) -> Self` - `fd:<n>` form used on Unix.
- `fn handle(n: u64) -> Self` - `handle:<n>` form used on Windows.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LinkSpec`
- **PartialEq**
  - `fn eq(self: &Self, other: &LinkSpec) -> bool`
- **FromStr**
  - `fn from_str(s: &str) -> Result<Self, <Self as >::Err>`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::link::LinkSpecError

*Enum*

Why a [`LinkSpec`](link.md#bookclerk_sandboxlinklinkspec) string could not be parsed.

**Variants:**
- `Unrecognized(String)` - Missing `fd:` / `handle:` prefix or empty remainder.
- `BadNumber(String)` - Prefix present but the number did not parse.

**Traits:** Error, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &LinkSpecError) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> LinkSpecError`



## bookclerk_sandbox::link::SOCKET_PROXY_ENV

*Constant*: `&str`

Env var naming the socket-proxy link for a native guest (SDK `net::connect`).

Unix: one duplex socket, `fd:<n>`. Windows product spawns name the guest's
overlapped read half here; the write half is [`SOCKET_PROXY_WRITE_ENV`](link.md#bookclerk_sandboxlinksocket_proxy_write_env).



## bookclerk_sandbox::link::SOCKET_PROXY_WRITE_ENV

*Constant*: `&str`

Windows-only env var naming the guest's overlapped write half of the proxy.

The SDK still accepts a single `handle:<n>` duplex when this is unset
(unit tests). Product spawns set both halves so a pending read cannot lock
a write on the same pipe.



## bookclerk_sandbox::link::StdioEnds

*Struct*

Host and guest ends of a pair of unidirectional stdio pipes (RPC).

**Fields:**
- `host_stdin: DuplexHalf` - Parent writes this into the child's stdin.
- `host_stdout: DuplexHalf` - Parent reads this from the child's stdout.
- `guest_stdin: DuplexHalf` - Child stdin read end.
- `guest_stdout: DuplexHalf` - Child stdout write end.

**Methods:**

- `fn pair() -> io::Result<Self>` - Two unidirectional pipes: host stdin-write / stdout-read, guest the opposite.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_sandbox::link::TEST_EXTRA_ENDPOINT_FD

*Constant*: `i32`

Test-only extra endpoint in the native guest. Production preserve lists omit it.

The slot sits above the descriptors a dynamic loader opens after the jail
closes every fd it does not preserve. A low number would be reused by that
loader; an open non-socket there is a numeric collision, not proof the
endpoint is absent. Production guests still preserve only [`GUEST_PROXY_FD`](link.md#bookclerk_sandboxlinkguest_proxy_fd).



## bookclerk_sandbox::link::WORKERD_STATE_DIR_ENV

*Constant*: `&str`

Host-chosen workerd session directory (`0700`, under `$TMPDIR`).



## bookclerk_sandbox::link::inherit_fd_at

*Function*

`dup2` `src` onto `dest` and clear `FD_CLOEXEC` on the destination.

Call from `Command::pre_exec` only. `src` stays `CLOEXEC` in the parent.
The child must close `src` when it differs from `dest` so the pre-dup
descriptor does not survive `exec`. After `dup2` the child sees `dest`
without `CLOEXEC`; the guest sets `FD_CLOEXEC` again when it adopts that
descriptor.

# Errors

Returns an I/O error when `dup2` or `fcntl` fails.

# Safety

Must run in the child after `fork` and before `exec`. `src` must be open.

```rust
fn inherit_fd_at(src: std::os::fd::RawFd, dest: std::os::fd::RawFd) -> io::Result<()>
```



## bookclerk_sandbox::link::with_fd_spawn_lock

*Function*

Serialize macOS descriptor allocation with in-process `Command` spawns.

macOS has no `SOCK_CLOEXEC` or `pipe2`. Socket and pipe creation in this
crate holds the lock from allocation until `FD_CLOEXEC` is set. Every
`Command::spawn` in that process must hold it for the `spawn` call, or a
concurrent `fork` inherits the new descriptor. Linux `SOCK_CLOEXEC` /
`pipe2` are atomic, so this is a no-op there. Do not create a link while
already inside the lock: the mutex is not reentrant.

```rust
fn with_fd_spawn_lock<R, impl FnOnce() -> R>(f: impl Trait) -> R
```



