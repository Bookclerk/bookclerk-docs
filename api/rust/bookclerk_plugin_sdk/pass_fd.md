**bookclerk_plugin_sdk > pass_fd**

# Module: pass_fd

## Contents

**Functions**

- [`fd_proc_path`](#fd_proc_path) - Builds the OS path that refers to an open descriptor by number.
- [`recv_passed_fd`](#recv_passed_fd) - Receives one file descriptor from the host side channel (SCM_RIGHTS).

**Constants**

- [`PLUGIN_FD_CHANNEL`](#plugin_fd_channel) - Fixed descriptor number the host leaves open for the plugin side channel.
- [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env) - Environment variable the host *would* set to the decimal form of [`PLUGIN_FD_CHANNEL`](pass_fd.md#bookclerk_plugin_sdkpass_fdplugin_fd_channel).

---

## bookclerk_plugin_sdk::pass_fd::PLUGIN_FD_CHANNEL

*Constant*: `i32`

Fixed descriptor number the host leaves open for the plugin side channel.

Always `3` (after stdin/stdout/stderr). Guests must not close or reuse this
FD for other purposes while the channel is active.



## bookclerk_plugin_sdk::pass_fd::PLUGIN_FD_CHANNEL_ENV

*Constant*: `&str`

Environment variable the host *would* set to the decimal form of [`PLUGIN_FD_CHANNEL`](pass_fd.md#bookclerk_plugin_sdkpass_fdplugin_fd_channel).

Hosts do **not** set this. Presence still arms `crate::FetchWorkDir::open`
/ `crate::UploadFile::open` to call [`recv_passed_fd`](pass_fd.md#bookclerk_plugin_sdkpass_fdrecv_passed_fd) instead of path
strings — only set it when the host is actually sending an FD.



## bookclerk_plugin_sdk::pass_fd::fd_proc_path

*Function*

Builds the OS path that refers to an open descriptor by number.

Linux uses `/proc/self/fd/{fd}`; macOS and other Unix use `/dev/fd/{fd}`.
The path is only valid while the descriptor remains open in this process.

# Arguments

* `fd` - Open raw file descriptor previously received from the host.

# Returns

PathBuf that can be passed to std / tokio filesystem APIs for as long as
`fd` stays open.

```rust
fn fd_proc_path(fd: i32) -> std::path::PathBuf
```



## bookclerk_plugin_sdk::pass_fd::recv_passed_fd

*Function*

Receives one file descriptor from the host side channel (SCM_RIGHTS).

Blocks until the host sends a control message on [`PLUGIN_FD_CHANNEL`](pass_fd.md#bookclerk_plugin_sdkpass_fdplugin_fd_channel).
Hosts do not send; prefer path strings unless you are implementing a
native shortcut that actually `sendmsg`s.

# Returns

Raw OS file descriptor owned by the caller. Prefer wrapping it immediately
(see [`crate::FetchWorkDir`](fetch_dir.md#bookclerk_plugin_sdkfetch_dirfetchworkdir) / [`crate::UploadFile`](fetch_dir.md#bookclerk_plugin_sdkfetch_diruploadfile)) so Drop closes it.

# Errors

Returns [`SdkError`](error.md#bookclerk_plugin_sdkerrorsdkerror) when [`PLUGIN_FD_CHANNEL_ENV`](pass_fd.md#bookclerk_plugin_sdkpass_fdplugin_fd_channel_env) is unset or not equal to
[`PLUGIN_FD_CHANNEL`](pass_fd.md#bookclerk_plugin_sdkpass_fdplugin_fd_channel), when `recvmsg` fails, or when the platform does not
support the side channel (non-Unix).

```rust
fn recv_passed_fd() -> crate::error::Result<i32>
```



