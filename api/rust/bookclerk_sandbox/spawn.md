# bookclerk_sandbox::spawn

Windows AppContainer spawn API ([`plan_appcontainer`](#plan_appcontainer),
[`run_appcontainer`](#run_appcontainer)).

Windows AppContainer planning / launch helpers re-exported on every OS so
callers can depend on a stable `bookclerk_sandbox::spawn` path. Non-Windows
builds keep the planning helpers and return a clear error from launch/ACL
entry points.

| Item | Kind | Summary |
| --- | --- | --- |
| [`NamedPipeSecurity`](#namedpipesecurity) | struct | Owned `SECURITY_ATTRIBUTES` suitable for `CreateNamedPipe` to one guest. |
| [`AclGrant`](#aclgrant) | struct | RAII guard that revokes a temporary Package-SID ACE on drop. |
| [`AclJournalEntry`](#acljournalentry) | struct | One planned DACL mutation for a session's package SID. |
| [`AppContainerLaunch`](#appcontainerlaunch) | struct | Pure AppContainer launch plan (capabilities only — no profile side effects). |
| [`AppContainerSession`](#appcontainersession) | struct | Owned AppContainer profile for one isolated launch. |
| [`grant_path_access`](#grant_path_access) | function | Grant the Package SID access to `path` for one RPC / spawn allowlist entry. |
| [`is_os_managed_path`](#is_os_managed_path) | function | Is os managed path. |
| [`plan_acl_journal`](#plan_acl_journal) | function | Planned ACL mutations for one sibling, derived from `spec`. |
| [`plan_appcontainer`](#plan_appcontainer) | function | Map a confinement `Policy` into AppContainer network capabilities. |
| [`profile_name_for_label`](#profile_name_for_label) | function | Sanitize a policy label into a diagnostics-friendly stem (not a full moniker). |
| [`protect_host_journal_dir`](#protect_host_journal_dir) | function | Replace the DACL on `path` with a protected ACL for Administrators, SYSTEM, and the owner. Inheritance from the parent is blocked. |
| [`revoke_acl_journal`](#revoke_acl_journal) | function | Revoke every entry. A missing path is success. A second call is a no-op for SIDs that are already gone. |
| [`revoke_acl_journal_retain`](#revoke_acl_journal_retain) | function | Revoke every entry, including the ones after a failure. |
| [`run_appcontainer`](#run_appcontainer) | function | Spawn `program` inside an AppContainer derived from `policy`, proxy stdio, and return the child's exit code. |
| [`run_appcontainer_with_handoff`](#run_appcontainer_with_handoff) | function | `run_appcontainer` plus an optional host `crate::JailHandoff` of inherited handles. |
| [`run_unconfined_with_handoff`](#run_unconfined_with_handoff) | function | Launch without an AppContainer, inheriting the host-duplicated handoff handles. |
| [`set_test_fail_acl_revoke`](#set_test_fail_acl_revoke) | function | Fail one journal entry on this thread, then continue with the rest. |
| [`unique_profile_moniker`](#unique_profile_moniker) | function | Build a collision-resistant AppContainer profile moniker (≤ 64 characters). |

## Structs

### NamedPipeSecurity

```rust
pub struct NamedPipeSecurity {}
```

Owned `SECURITY_ATTRIBUTES` suitable for `CreateNamedPipe` to one guest.

Keep this value alive until the create call returns; the kernel copies the
descriptor onto the pipe object.

#### NamedPipeSecurity::for_app_container

```rust
pub fn for_app_container(package_sid: &str) -> Result<Self, SandboxError>
```

Build attributes that allow `package_sid` to open a duplex named pipe.

##### Errors

Returns [`SandboxError::Backend`](index.md#sandboxerror) when `package_sid` is not a Package
SID string, or when Win32 refuses to parse the SDDL.

#### NamedPipeSecurity::as_mut_ptr

```rust
pub fn as_mut_ptr(&mut self) -> *mut c_void
```

Raw pointer for Tokio `create_with_security_attributes_raw` / Win32.

Implements: `Drop`

### AclGrant

```rust
pub struct AclGrant {}
```

RAII guard that revokes a temporary Package-SID ACE on drop.

With a per-launch Package SID, `REVOKE_ACCESS` removes only this launch’s
trustee ACEs on the path (not another job’s grants).

#### AclGrant::path

```rust
pub fn path(&self) -> &Path
```

Path this grant covers.

#### AclGrant::is_active

```rust
pub fn is_active(&self) -> bool
```

Whether an ACE was actually written (false for ambient OS skips).

Implements: `Debug`, `Drop`

### AclJournalEntry

```rust
pub struct AclJournalEntry {
    pub path: PathBuf,
    pub package_sid: String,
    pub is_dir: bool,
    pub propagate: bool,
}
```

One planned DACL mutation for a session's package SID.

The host builds this from the spec it already handed to the jail. Revoke
matches only `package_sid`, so a second live session on the same path keeps
its grant.

**Fields**

- `path`: Filesystem path the jail was going to ACE.
- `package_sid`: Package SID trustee. Revoke removes only this trustee.
- `is_dir`: Directory at grant time. Selects the revoke inheritance flag.
- `propagate`: `true` for inheritable leaf grants (`SetSecurityInfo` on revoke).
  Ancestor traverse is `false` (`SetKernelObjectSecurity`).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AppContainerLaunch

```rust
pub struct AppContainerLaunch {
    pub label_stem: String,
    pub capability_names: Vec<&'static str>,
}
```

Pure AppContainer launch plan (capabilities only — no profile side effects).

**Fields**

- `label_stem`: Sanitized diagnostics stem derived from the policy label (not a moniker).
- `capability_names`: Named capability SIDs to grant (well-known capability names).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### AppContainerSession

```rust
pub struct AppContainerSession {
    /* private fields */
}
```

Owned AppContainer profile for one isolated launch.

When [`delete_on_drop`](#appcontainersessiondelete_on_drop) is true (the default for
[`Self::create`](#appcontainersessioncreate)), dropping the session deletes the profile after the guest
and its Job Object have exited. Host-owned sessions created for long-lived
plugins use [`Self::attach`](#appcontainersessionattach) so the jail does not delete the profile.

#### AppContainerSession::create

```rust
pub fn create(label: &str) -> Result<Self, SandboxError>
```

Create a fresh profile with a unique moniker derived from `label`.

##### Errors

Returns an error when the operation fails.

#### AppContainerSession::attach

```rust
pub fn attach(profile_name: &str) -> Result<Self, SandboxError>
```

Open (ensure) an existing profile without taking deletion ownership.

##### Errors

Returns an error when the operation fails.

#### AppContainerSession::profile_name

```rust
pub fn profile_name(&self) -> &str
```

Profile moniker passed to CreateAppContainerProfile.

#### AppContainerSession::package_sid

```rust
pub fn package_sid(&self) -> &str
```

Package SID as an SDDL string.

#### AppContainerSession::delete_on_drop

```rust
pub fn delete_on_drop(&self) -> bool
```

Whether [`Drop`](https://doc.rust-lang.org/stable/core/ops/drop/trait.Drop.html) will call DeleteAppContainerProfile.

#### AppContainerSession::disarm_delete

```rust
pub fn disarm_delete(&mut self)
```

Relinquish deletion ownership (e.g. transfer to another owner).

#### AppContainerSession::arm_delete

```rust
pub fn arm_delete(&mut self)
```

Take deletion ownership (DeleteAppContainerProfile on drop).

Implements: `Debug`, `Drop`

## Functions

### grant_path_access

```rust
pub fn grant_path_access(
    package_sid: &str,
    path: &Path,
    write: bool,
) -> Result<AclGrant, SandboxError>
```

Grant the Package SID access to `path` for one RPC / spawn allowlist entry.

When `write` is true the ACE includes generic write; otherwise read+execute.
The returned [`AclGrant`](#aclgrant) revokes the ACE on drop.

Ambient OS runtime paths: read grants are no-ops (OS ALL APPLICATION PACKAGES
already covers them); write grants fail closed without calling an ACL API.

#### Errors

Returns an error when the operation fails.

### is_os_managed_path

```rust
pub fn is_os_managed_path(_path: &Path) -> bool
```

Is os managed path.

### plan_acl_journal

```rust
pub fn plan_acl_journal(
    spec: &Spec,
    package_sid: &str,
    program: Option<&Path>,
    profile_dirs: &[PathBuf],
) -> Vec<AclJournalEntry>
```

Planned ACL mutations for one sibling, derived from `spec`.

Ambient OS paths are omitted. `profile_dirs` are extra inheritable write
grants (the AppContainer folder and `Temp`). Ancestor traverse entries use
`propagate = false`.

### plan_appcontainer

```rust
pub fn plan_appcontainer(policy: &Policy) -> AppContainerLaunch
```

Map a confinement [`Policy`](index.md#policy) into AppContainer network capabilities.

This is pure: it does **not** create or ensure an AppContainer profile.
Create a profile with [`AppContainerSession::create`](#appcontainersessioncreate) (or attach a host-owned
one) before [`run_appcontainer`](#run_appcontainer).

| [`NetPolicy`](index.md#netpolicy) | Capabilities |
| --- | --- |
| [`Deny`](index.md#netpolicy) | _(none)_ |
| [`Outbound`](index.md#netpolicy) | `internetClient` |
| [`OutboundListen`](index.md#netpolicy) | `internetClient`, `privateNetworkClientServer` |
| [`Full`](index.md#netpolicy) | `internetClient`, `internetClientServer`, `privateNetworkClientServer` |

### profile_name_for_label

```rust
pub fn profile_name_for_label(label: &str) -> String
```

Sanitize a policy label into a diagnostics-friendly stem (not a full moniker).

Prefer [`unique_profile_moniker`](#unique_profile_moniker) for CreateAppContainerProfile names.

### protect_host_journal_dir

```rust
pub fn protect_host_journal_dir(path: &Path) -> Result<(), SandboxError>
```

Replace the DACL on `path` with a protected ACL for Administrators, SYSTEM,
and the owner. Inheritance from the parent is blocked.

#### Errors

Returns [`SandboxError::Backend`](index.md#sandboxerror) when the security descriptor cannot be
built or applied. Non-Windows hosts succeed without changing permissions.

### revoke_acl_journal

```rust
pub fn revoke_acl_journal(entries: &[AclJournalEntry]) -> Result<(), SandboxError>
```

Revoke every entry. A missing path is success. A second call is a no-op
for SIDs that are already gone.

#### Errors

Returns [`SandboxError::Backend`](index.md#sandboxerror) when a present path cannot be updated.

### revoke_acl_journal_retain

```rust
pub fn revoke_acl_journal_retain(
    entries: &mut Vec<AclJournalEntry>,
) -> Result<(), SandboxError>
```

Revoke every entry, including the ones after a failure.

On success `entries` is empty. On failure it keeps only the entries whose
revoke failed. Later entries are still attempted, so one path that always
denies `WRITE_DAC` does not pin the rest of the journal. A missing path
counts as already revoked and is not retained.

#### Errors

Returns the first [`SandboxError::Backend`](index.md#sandboxerror). [`set_test_fail_acl_revoke`](#set_test_fail_acl_revoke)
forces one index to fail before any Win32 call. That hook is thread-local
and is not read from the environment.

### run_appcontainer

```rust
pub fn run_appcontainer(
    policy: &Policy,
    program: &Path,
    args: &[OsString],
    session: Option<&AppContainerSession>,
) -> Result<u32, SandboxError>
```

Spawn `program` inside an AppContainer derived from `policy`, proxy stdio,
and return the child's exit code.

When `session` is `None`, a unique profile is created for this launch and
deleted after the process tree exits. When `Some`, the caller owns the
profile (typical for long-lived plugin guests).

Stdio is proxied (not inherited) because Win32 AppContainer launch via
`CreateProcess` with extended attributes does not reliably inherit the
launcher's redirected pipes; the jail process sits in the middle and copies
bytes until the guest exits.

#### Errors

Returns [`SandboxError::Backend`](index.md#sandboxerror) when the profile, ACLs, or CreateProcess
step fails. On non-Windows hosts this always fails.

### run_appcontainer_with_handoff

```rust
pub fn run_appcontainer_with_handoff(
    policy: &Policy,
    program: &Path,
    args: &[OsString],
    session: Option<&AppContainerSession>,
    handoff: Option<&JailHandoff>,
) -> Result<u32, SandboxError>
```

[`run_appcontainer`](#run_appcontainer) plus an optional host [`crate::JailHandoff`](index.md#jailhandoff) of inherited handles.

#### Errors

Returns [`SandboxError::Backend`](index.md#sandboxerror) when the profile, ACLs, or CreateProcess
step fails. On non-Windows hosts this always fails.

### run_unconfined_with_handoff

```rust
pub fn run_unconfined_with_handoff(
    program: &Path,
    args: &[OsString],
    handoff: &JailHandoff,
) -> Result<u32, SandboxError>
```

Launch without an AppContainer, inheriting the host-duplicated handoff handles.

Used when `[plugins].isolation = off` on Windows: the jail is still the
sole `CreateProcess` so `PROC_THREAD_ATTRIBUTE_HANDLE_LIST` can name the
session links. There is no package SID and no ACL mutation.

#### Errors

Returns [`SandboxError::Backend`](index.md#sandboxerror) when CreateProcess fails. On non-Windows
hosts this always fails.

### set_test_fail_acl_revoke

```rust
pub fn set_test_fail_acl_revoke(index: Option<usize>)
```

Fail one journal entry on this thread, then continue with the rest.

`None` clears the hook. Production hosts do not call this. The value is
not taken from the environment.

### unique_profile_moniker

```rust
pub fn unique_profile_moniker(label: &str) -> String
```

Build a collision-resistant AppContainer profile moniker (≤ 64 characters).

Format: `bc.<sanitized-label-stem>.<16 hex random>`. Concurrent launches with
the same policy label always receive distinct Package SIDs.
