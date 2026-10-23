**bookclerk_plugin_host > spawn_plan**

# Module: spawn_plan

## Contents

**Structs**

- [`SpawnPlan`](#spawnplan) - Everything the spawn path needs to start one guest.
- [`WorkerdFrontDoor`](#workerdfrontdoor) - Resolved `bookclerk-workerd` launcher and the pinned `workerd` beside it.

**Enums**

- [`GuestRuntimeKind`](#guestruntimekind) - Process tree the jail will contain, as reported in [`crate::ExecutorIdentity`](rpc_session.md#bookclerk_plugin_hostrpc_sessionexecutoridentity).
- [`SpawnTransport`](#spawntransport) - Which transport a spawn uses to reach the guest.

**Constants**

- [`WORKERD_BIN_ENV`](#workerd_bin_env) - Override for the pinned Cloudflare `workerd` binary, honoured by
- [`WORKERD_LAUNCHER_ENV`](#workerd_launcher_env) - Override for the `bookclerk-workerd` launcher path (tests / packaging).

---

## bookclerk_plugin_host::spawn_plan::GuestRuntimeKind

*Enum*

Process tree the jail will contain, as reported in [`crate::ExecutorIdentity`](rpc_session.md#bookclerk_plugin_hostrpc_sessionexecutoridentity).

**Variants:**
- `Workerd` - `bookclerk-workerd` + the pinned `workerd` running author modules.
- `NativeBehindWorkerd` - `bookclerk-workerd` + `workerd` (control plane) + the native guest.
- `NativeDirect` - The native guest alone, spoken to directly (diagnostic transport).

**Methods:**

- `fn label(self: Self) -> &'static str` - Stable label used in executor identities and logs.
- `fn process_overhead(self: Self) -> u32` - Payload occupancy before guest-owned children.
- `fn fronted_by_workerd(self: Self) -> bool` - True when `bookclerk-workerd` is the program the jail execs.
- `fn for_manifest(runtime: PluginRuntimeKind, transport: SpawnTransport) -> Self` - Runtime kind a manifest resolves to under `transport`.

**Traits:** Copy, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &GuestRuntimeKind) -> bool`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Clone**
  - `fn clone(self: &Self) -> GuestRuntimeKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::spawn_plan::SpawnPlan

*Struct*

Everything the spawn path needs to start one guest.

**Fields:**
- `launcher: std::path::PathBuf` - Program the jail execs: `bookclerk-workerd`, or the native command on
- `args: Vec<String>` - Arguments after `launcher` (manifest `args`; diagnostic transport only).
- `native_backend: Option<std::path::PathBuf>` - Native executable the host spawns as the sibling guest. `None` for
- `workerd_bin: Option<std::path::PathBuf>` - Pinned `workerd` the launcher will spawn (must stay readable in the jail).
- `runtime: GuestRuntimeKind` - Process tree this plan produces.

**Methods:**

- `fn resolve(plugin: &DiscoveredPlugin, transport: SpawnTransport) -> Result<Self>` - Resolves the launch for `plugin` on `transport` using the host's front door.
- `fn resolve_with<impl FnOnce() -> Result<WorkerdFrontDoor>>(plugin: &DiscoveredPlugin, transport: SpawnTransport, locate: impl Trait) -> Result<Self>` - `Self::resolve` with an injected front-door locator (called only when
- `fn fronted_by_workerd(self: &Self) -> bool` - True when `bookclerk-workerd` is the program the jail execs.
- `fn executable_reads(self: &Self) -> Vec<PathBuf>` - Executable paths the jail must let the launcher tree read and exec.
- `fn gateway_executable_reads(self: &Self) -> Vec<PathBuf>` - Executables the gateway jail may read: `bookclerk-workerd` and `workerd`.
- `fn guest_executable_reads(self: &Self) -> Vec<PathBuf>` - Executables the native sibling jail may read (the backend only).

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> SpawnPlan`
- **PartialEq**
  - `fn eq(self: &Self, other: &SpawnPlan) -> bool`



## bookclerk_plugin_host::spawn_plan::SpawnTransport

*Enum*

Which transport a spawn uses to reach the guest.

**Variants:**
- `WorkerdFrontDoor` - Product path: `bookclerk-workerd` fronts every guest, native or isolate.
- `DirectNativeDiagnostic` - Host ↔ native Cap'n Proto on the guest's own stdio, no isolate.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SpawnTransport`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SpawnTransport) -> bool`
- **Default**
  - `fn default() -> SpawnTransport`



## bookclerk_plugin_host::spawn_plan::WORKERD_BIN_ENV

*Constant*: `&str`

Override for the pinned Cloudflare `workerd` binary, honoured by
`bookclerk-workerd` and forwarded to it by the host.



## bookclerk_plugin_host::spawn_plan::WORKERD_LAUNCHER_ENV

*Constant*: `&str`

Override for the `bookclerk-workerd` launcher path (tests / packaging).



## bookclerk_plugin_host::spawn_plan::WorkerdFrontDoor

*Struct*

Resolved `bookclerk-workerd` launcher and the pinned `workerd` beside it.

**Fields:**
- `launcher: std::path::PathBuf` - `bookclerk-workerd` executable the jail execs.
- `workerd_bin: std::path::PathBuf` - Pinned Cloudflare `workerd` the launcher spawns.

**Methods:**

- `fn locate() -> Result<Self>` - Locates the front door for this host process.
- `fn locate_in(dir: &Path) -> Result<Self>` - Locates the front door looking **only** in `dir` (no environment

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> WorkerdFrontDoor`
- **PartialEq**
  - `fn eq(self: &Self, other: &WorkerdFrontDoor) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



