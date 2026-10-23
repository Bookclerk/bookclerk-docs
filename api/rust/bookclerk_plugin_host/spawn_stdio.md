**bookclerk_plugin_host > spawn_stdio**

# Module: spawn_stdio

## Contents

**Functions**

- [`note_spawn_stage`](#note_spawn_stage) - Record one secret-free startup line.
- [`recent_spawn_diagnostics`](#recent_spawn_diagnostics) - Bounded startup log for the current process.
- [`retry_abandoned_acl_journals`](#retry_abandoned_acl_journals) - Retry abandoned journals for every plugin under `files_dir/plugin-state`.
- [`revoke_plugin_state_before_purge`](#revoke_plugin_state_before_purge) - Revoke this plugin's journal before `--purge-state` deletes the directory.

**Constants**

- [`TEST_CHANNEL_IDENT_ENV`](#test_channel_ident_env) - Set to `1` or `true` in the host process to give each proxy a channel tag.
- [`TEST_CHANNEL_TAG_FILE`](#test_channel_tag_file) - Session-directory file holding the tag when [`TEST_CHANNEL_IDENT_ENV`](spawn_stdio.md#bookclerk_plugin_hostspawn_stdiotest_channel_ident_env) is set.
- [`TEST_INJECT_EXTRA_ENDPOINT_ENV`](#test_inject_extra_endpoint_env) - Host-process description of one extra endpoint to add to the guest handoff.
- [`TEST_OBSERVE_EXTRA_ENDPOINT_ENV`](#test_observe_extra_endpoint_env) - Host-process request to publish candidate-endpoint observation metadata.

---

## bookclerk_plugin_host::spawn_stdio::TEST_CHANNEL_IDENT_ENV

*Constant*: `&str`

Set to `1` or `true` in the host process to give each proxy a channel tag.

The variable is not copied into the guest. The tag is written under the
gateway session directory as [`TEST_CHANNEL_TAG_FILE`](spawn_stdio.md#bookclerk_plugin_hostspawn_stdiotest_channel_tag_file) and answered only
by that session's proxy. Production leaves it unset.



## bookclerk_plugin_host::spawn_stdio::TEST_CHANNEL_TAG_FILE

*Constant*: `&str`

Session-directory file holding the tag when [`TEST_CHANNEL_IDENT_ENV`](spawn_stdio.md#bookclerk_plugin_hostspawn_stdiotest_channel_ident_env) is set.



## bookclerk_plugin_host::spawn_stdio::TEST_INJECT_EXTRA_ENDPOINT_ENV

*Constant*: `&str`

Host-process description of one extra endpoint to add to the guest handoff.

Unix: `fd:<n>` of a socket in this process. Windows:
`handle:<read>,handle:<write>` of the guest pipe ends in this process.
The variable is not copied into the guest. Production leaves it unset, so
the guest preserve list stays [`bookclerk_sandbox::GUEST_PROXY_FD`](../bookclerk_sandbox/link.md#bookclerk_sandboxlinkguest_proxy_fd) only.
Setting it transfers the endpoint. It does not, by itself, publish the
candidate metadata the guest uses to observe that slot.



## bookclerk_plugin_host::spawn_stdio::TEST_OBSERVE_EXTRA_ENDPOINT_ENV

*Constant*: `&str`

Host-process request to publish candidate-endpoint observation metadata.

`1` or `true` names the fixed Unix slot, and on Windows the handles from
[`TEST_INJECT_EXTRA_ENDPOINT_ENV`](spawn_stdio.md#bookclerk_plugin_hostspawn_stdiotest_inject_extra_endpoint_env). A Windows `handle:<read>,handle:<write>`
value names that candidate without transferring it. The variable is not
copied into the guest. Unset means the guest reports `not-run`, which is
not proof the endpoint is absent.



## bookclerk_plugin_host::spawn_stdio::note_spawn_stage

*Function*

Record one secret-free startup line.

The line is a `bookclerk::spawn` tracing event and a byte-capped ring
entry. Raw stderr is written only when [`spawn_diag_stderr_enabled`](../bookclerk_sandbox/spawn_diag.md#bookclerk_sandboxspawn_diagspawn_diag_stderr_enabled) is
set (`BOOKCLERK_SPAWN_DIAG=1|true|stderr`). Session challenges are stripped.
Plugin stderr is not stored here; the per-session tail is separate.

```rust
fn note_spawn_stage(message: &str)
```



## bookclerk_plugin_host::spawn_stdio::recent_spawn_diagnostics

*Function*

Bounded startup log for the current process.

A deadline handler prints this before exiting. The snapshot is a second
capped copy of the ring, not a join of unbounded strings.

```rust
fn recent_spawn_diagnostics() -> String
```



## bookclerk_plugin_host::spawn_stdio::retry_abandoned_acl_journals

*Function*

Retry abandoned journals for every plugin under `files_dir/plugin-state`.

Host startup calls this so a crash does not wait for the next spawn of the
same plugin. A live session's owner lock is left alone.

```rust
fn retry_abandoned_acl_journals(files_dir: &std::path::Path)
```



## bookclerk_plugin_host::spawn_stdio::revoke_plugin_state_before_purge

*Function*

Revoke this plugin's journal before `--purge-state` deletes the directory.

`discard_rejected` deletes `session-*.json.rejected` with plugin state.
A readable journal that is still present is never discarded that way.

# Errors

Returns an error when a journal file is still present, when a live session
holds one, or when a rejected record remains and `discard_rejected` is
false. Purge then leaves plugin-state in place.

```rust
fn revoke_plugin_state_before_purge(state_root: &std::path::Path, discard_rejected: bool) -> std::result::Result<(), String>
```



