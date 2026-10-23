**bookclerk_sandbox > platform > windows_spawn**

# Module: platform::windows_spawn

## Contents

**Structs**

- [`AclGrant`](#aclgrant) - RAII guard that revokes a temporary Package-SID ACE on drop.
- [`AclJournalEntry`](#acljournalentry) - One planned DACL mutation for a session's package SID.
- [`AppContainerLaunch`](#appcontainerlaunch) - Pure AppContainer launch plan (capabilities only — no profile side effects).
- [`AppContainerSession`](#appcontainersession) - Owned AppContainer profile for one isolated launch.

**Functions**

- [`grant_path_access`](#grant_path_access) - Grant the Package SID access to `path` for one RPC / spawn allowlist entry.
- [`is_os_managed_path`](#is_os_managed_path) - Is os managed path.
- [`plan_acl_journal`](#plan_acl_journal) - Planned ACL mutations for one sibling, derived from `spec`.
- [`plan_appcontainer`](#plan_appcontainer) - Map a confinement [`Policy`](../bookclerk_sandbox.md#bookclerk_sandboxpolicy) into AppContainer network capabilities.
- [`profile_name_for_label`](#profile_name_for_label) - Sanitize a policy label into a diagnostics-friendly stem (not a full moniker).
- [`protect_host_journal_dir`](#protect_host_journal_dir) - Replace the DACL on `path` with a protected ACL for Administrators, SYSTEM,
- [`recent_platform_spawn_diagnostics`](#recent_platform_spawn_diagnostics) - Startup lines recorded by AppContainer profile, DACL, and proxy stages.
- [`revoke_acl_journal`](#revoke_acl_journal) - Revoke every entry. A missing path is success. A second call is a no-op
- [`revoke_acl_journal_retain`](#revoke_acl_journal_retain) - Revoke every entry, including the ones after a failure.
- [`run_appcontainer`](#run_appcontainer) - Spawn `program` inside an AppContainer derived from `policy`, proxy stdio,
- [`run_appcontainer_with_handoff`](#run_appcontainer_with_handoff) - [`run_appcontainer`](windows_spawn.md#bookclerk_sandboxplatformwindows_spawnrun_appcontainer) plus an optional host [`crate::JailHandoff`](../link.md#bookclerk_sandboxlinkjailhandoff) of inherited handles.
- [`run_unconfined_with_handoff`](#run_unconfined_with_handoff) - Launch without an AppContainer, inheriting the host-duplicated handoff handles.
- [`set_test_fail_acl_revoke`](#set_test_fail_acl_revoke) - Fail one journal entry on this thread, then continue with the rest.
- [`unique_profile_moniker`](#unique_profile_moniker) - Build a collision-resistant AppContainer profile moniker (≤ 64 characters).

---

## bookclerk_sandbox::platform::windows_spawn::AclGrant

*Struct*

RAII guard that revokes a temporary Package-SID ACE on drop.

With a per-launch Package SID, `REVOKE_ACCESS` removes only this launch’s
trustee ACEs on the path (not another job’s grants).

**Methods:**

- `fn path(self: &Self) -> &Path` - Path this grant covers.
- `fn is_active(self: &Self) -> bool` - Whether an ACE was actually written (false for ambient OS skips).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Drop**
  - `fn drop(self: & mut Self)`



## bookclerk_sandbox::platform::windows_spawn::AclJournalEntry

*Struct*

One planned DACL mutation for a session's package SID.

The host builds this from the spec it already handed to the jail. Revoke
matches only `package_sid`, so a second live session on the same path keeps
its grant.

**Fields:**
- `path: std::path::PathBuf` - Filesystem path the jail was going to ACE.
- `package_sid: String` - Package SID trustee. Revoke removes only this trustee.
- `is_dir: bool` - Directory at grant time. Selects the revoke inheritance flag.
- `propagate: bool` - `true` for inheritable leaf grants (`SetSecurityInfo` on revoke).

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> AclJournalEntry`
- **PartialEq**
  - `fn eq(self: &Self, other: &AclJournalEntry) -> bool`



## bookclerk_sandbox::platform::windows_spawn::AppContainerLaunch

*Struct*

Pure AppContainer launch plan (capabilities only — no profile side effects).

**Fields:**
- `label_stem: String` - Sanitized diagnostics stem derived from the policy label (not a moniker).
- `capability_names: Vec<&'static str>` - Named capability SIDs to grant (well-known capability names).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AppContainerLaunch) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> AppContainerLaunch`



## bookclerk_sandbox::platform::windows_spawn::AppContainerSession

*Struct*

Owned AppContainer profile for one isolated launch.

When `delete_on_drop` is true (the default for
`Self::create`), dropping the session deletes the profile after the guest
and its Job Object have exited. Host-owned sessions created for long-lived
plugins use `Self::attach` so the jail does not delete the profile.

**Methods:**

- `fn create(label: &str) -> Result<Self, SandboxError>` - Create a fresh profile with a unique moniker derived from `label`.
- `fn attach(profile_name: &str) -> Result<Self, SandboxError>` - Open (ensure) an existing profile without taking deletion ownership.
- `fn profile_name(self: &Self) -> &str` - Profile moniker passed to CreateAppContainerProfile.
- `fn package_sid(self: &Self) -> &str` - Package SID as an SDDL string.
- `fn delete_on_drop(self: &Self) -> bool` - Whether [`Drop`](https://doc.rust-lang.org/stable/core/ops/drop/trait.Drop.html) will call DeleteAppContainerProfile.
- `fn disarm_delete(self: & mut Self)` - Relinquish deletion ownership (e.g. transfer to another owner).
- `fn arm_delete(self: & mut Self)` - Take deletion ownership (DeleteAppContainerProfile on drop).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Drop**
  - `fn drop(self: & mut Self)`



## bookclerk_sandbox::platform::windows_spawn::grant_path_access

*Function*

Grant the Package SID access to `path` for one RPC / spawn allowlist entry.

When `write` is true the ACE includes generic write; otherwise read+execute.
The returned [`AclGrant`](windows_spawn.md#bookclerk_sandboxplatformwindows_spawnaclgrant) revokes the ACE on drop.

Ambient OS runtime paths: read grants are no-ops (OS ALL APPLICATION PACKAGES
already covers them); write grants fail closed without calling an ACL API.

# Errors

Returns an error when the operation fails.

```rust
fn grant_path_access(package_sid: &str, path: &std::path::Path, write: bool) -> Result<AclGrant, crate::SandboxError>
```



## bookclerk_sandbox::platform::windows_spawn::is_os_managed_path

*Function*

Is os managed path.

```rust
fn is_os_managed_path(_path: &std::path::Path) -> bool
```



## bookclerk_sandbox::platform::windows_spawn::plan_acl_journal

*Function*

Planned ACL mutations for one sibling, derived from `spec`.

Ambient OS paths are omitted. `profile_dirs` are extra inheritable write
grants (the AppContainer folder and `Temp`). Ancestor traverse entries use
`propagate = false`.

```rust
fn plan_acl_journal(spec: &crate::Spec, package_sid: &str, program: Option<&std::path::Path>, profile_dirs: &[std::path::PathBuf]) -> Vec<AclJournalEntry>
```



## bookclerk_sandbox::platform::windows_spawn::plan_appcontainer

*Function*

Map a confinement [`Policy`](../bookclerk_sandbox.md#bookclerk_sandboxpolicy) into AppContainer network capabilities.

This is pure: it does **not** create or ensure an AppContainer profile.
Create a profile with `AppContainerSession::create` (or attach a host-owned
one) before [`run_appcontainer`](windows_spawn.md#bookclerk_sandboxplatformwindows_spawnrun_appcontainer).

| [`NetPolicy`](../bookclerk_sandbox.md#bookclerk_sandboxnetpolicy) | Capabilities |
| --- | --- |
| `Deny` | _(none)_ |
| `Outbound` | `internetClient` |
| `OutboundListen` | `internetClient`, `privateNetworkClientServer` |
| `Full` | `internetClient`, `internetClientServer`, `privateNetworkClientServer` |

```rust
fn plan_appcontainer(policy: &crate::Policy) -> AppContainerLaunch
```



## bookclerk_sandbox::platform::windows_spawn::profile_name_for_label

*Function*

Sanitize a policy label into a diagnostics-friendly stem (not a full moniker).

Prefer [`unique_profile_moniker`](windows_spawn.md#bookclerk_sandboxplatformwindows_spawnunique_profile_moniker) for CreateAppContainerProfile names.

```rust
fn profile_name_for_label(label: &str) -> String
```



## bookclerk_sandbox::platform::windows_spawn::protect_host_journal_dir

*Function*

Replace the DACL on `path` with a protected ACL for Administrators, SYSTEM,
and the owner. Inheritance from the parent is blocked.

# Errors

Returns `SandboxError::Backend` when the security descriptor cannot be
built or applied. Non-Windows hosts succeed without changing permissions.

```rust
fn protect_host_journal_dir(path: &std::path::Path) -> Result<(), crate::SandboxError>
```



## bookclerk_sandbox::platform::windows_spawn::recent_platform_spawn_diagnostics

*Function*

Startup lines recorded by AppContainer profile, DACL, and proxy stages.

Byte-capped. Raw copies are not written to the daemon log unless
`BOOKCLERK_SPAWN_DIAG` is set. A jailed child's own copy stays on its
stderr pipe so the host forwarder can re-emit it as tracing.

```rust
fn recent_platform_spawn_diagnostics() -> String
```



## bookclerk_sandbox::platform::windows_spawn::revoke_acl_journal

*Function*

Revoke every entry. A missing path is success. A second call is a no-op
for SIDs that are already gone.

# Errors

Returns `SandboxError::Backend` when a present path cannot be updated.

```rust
fn revoke_acl_journal(entries: &[AclJournalEntry]) -> Result<(), crate::SandboxError>
```



## bookclerk_sandbox::platform::windows_spawn::revoke_acl_journal_retain

*Function*

Revoke every entry, including the ones after a failure.

On success `entries` is empty. On failure it keeps only the entries whose
revoke failed. Later entries are still attempted, so one path that always
denies `WRITE_DAC` does not pin the rest of the journal. A missing path
counts as already revoked and is not retained.

# Errors

Returns the first `SandboxError::Backend`. [`set_test_fail_acl_revoke`](windows_spawn.md#bookclerk_sandboxplatformwindows_spawnset_test_fail_acl_revoke)
forces one index to fail before any Win32 call. That hook is thread-local
and is not read from the environment.

```rust
fn revoke_acl_journal_retain(entries: & mut Vec<AclJournalEntry>) -> Result<(), crate::SandboxError>
```



## bookclerk_sandbox::platform::windows_spawn::run_appcontainer

*Function*

Spawn `program` inside an AppContainer derived from `policy`, proxy stdio,
and return the child's exit code.

When `session` is `None`, a unique profile is created for this launch and
deleted after the process tree exits. When `Some`, the caller owns the
profile (typical for long-lived plugin guests).

Stdio is proxied (not inherited) because Win32 AppContainer launch via
`CreateProcess` with extended attributes does not reliably inherit the
launcher's redirected pipes; the jail process sits in the middle and copies
bytes until the guest exits.

# Errors

Returns `SandboxError::Backend` when the profile, ACLs, or CreateProcess
step fails. On non-Windows hosts this always fails.

```rust
fn run_appcontainer(policy: &crate::Policy, program: &std::path::Path, args: &[std::ffi::OsString], session: Option<&AppContainerSession>) -> Result<u32, crate::SandboxError>
```



## bookclerk_sandbox::platform::windows_spawn::run_appcontainer_with_handoff

*Function*

[`run_appcontainer`](windows_spawn.md#bookclerk_sandboxplatformwindows_spawnrun_appcontainer) plus an optional host [`crate::JailHandoff`](../link.md#bookclerk_sandboxlinkjailhandoff) of inherited handles.

# Errors

Returns `SandboxError::Backend` when the profile, ACLs, or CreateProcess
step fails. On non-Windows hosts this always fails.

```rust
fn run_appcontainer_with_handoff(policy: &crate::Policy, program: &std::path::Path, args: &[std::ffi::OsString], session: Option<&AppContainerSession>, handoff: Option<&crate::JailHandoff>) -> Result<u32, crate::SandboxError>
```



## bookclerk_sandbox::platform::windows_spawn::run_unconfined_with_handoff

*Function*

Launch without an AppContainer, inheriting the host-duplicated handoff handles.

Used when `[plugins].isolation = off` on Windows: the jail is still the
sole `CreateProcess` so `PROC_THREAD_ATTRIBUTE_HANDLE_LIST` can name the
session links. There is no package SID and no ACL mutation.

# Errors

Returns `SandboxError::Backend` when CreateProcess fails. On non-Windows
hosts this always fails.

```rust
fn run_unconfined_with_handoff(program: &std::path::Path, args: &[std::ffi::OsString], handoff: &crate::JailHandoff) -> Result<u32, crate::SandboxError>
```



## bookclerk_sandbox::platform::windows_spawn::set_test_fail_acl_revoke

*Function*

Fail one journal entry on this thread, then continue with the rest.

`None` clears the hook. Production hosts do not call this. The value is
not taken from the environment.

```rust
fn set_test_fail_acl_revoke(index: Option<usize>)
```



## bookclerk_sandbox::platform::windows_spawn::unique_profile_moniker

*Function*

Build a collision-resistant AppContainer profile moniker (≤ 64 characters).

Format: `bc.<sanitized-label-stem>.<16 hex random>`. Concurrent launches with
the same policy label always receive distinct Package SIDs.

```rust
fn unique_profile_moniker(label: &str) -> String
```



