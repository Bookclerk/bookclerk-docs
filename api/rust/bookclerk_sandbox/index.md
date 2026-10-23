# bookclerk_sandbox

Portable, unprivileged process confinement.

# Audience

Host binaries that confine media workers and plugin guests (`bookclerk-jail`,
`bookclerk-media-worker`). Guest plugins never link this crate; the host
imposes the jail. See `docs/plugins.md#the-guest-jail` and `docs/media.md`.

Bookclerk confines code by what it may touch, not by which uid it runs as.
Every backend here works without root, without setuid helpers, and without
user namespaces or bind mounts:

- **Linux** — Landlock filesystem allowlist plus a seccomp-bpf deny list.
- **macOS** — Seatbelt (`sandbox_init`) with a deny-default SBPL profile.
- **Windows** — AppContainer applied at `CreateProcess`; see
  `spawn`. A process cannot confine *itself* on Windows, so
  `Policy::confine_current_process` reports the filesystem layer as
  `LayerStatus::Unsupported` there. Children are confined by
  [`spawn::run_appcontainer`](platform/windows_spawn.md#bookclerk_sandboxplatformwindows_spawnrun_appcontainer) (used by `bookclerk-jail`).

Callers pick the failure mode with [`Enforcement`](bookclerk_sandbox.md#bookclerk_sandboxenforcement). `Required` turns a
backend that cannot enforce into an error, which is what production paths
use so a missing jail can never pass silently.

## Modules

### [`bookclerk_sandbox`](bookclerk_sandbox.md)

*1 module, 3 constants, 4 structs, 5 enums, 6 functions*

### [`capacity`](capacity.md)

*1 constant, 1 enum, 14 functions, 2 structs*

### [`guest_ipc`](guest_ipc.md)

*1 constant, 2 functions*

### [`link`](link.md)

*12 constants, 2 enums, 2 functions, 5 structs*

### [`platform::linux`](platform/linux.md)

*1 constant, 2 functions*

### [`platform::windows_pipe`](platform/windows_pipe.md)

*1 struct*

### [`platform::windows_spawn`](platform/windows_spawn.md)

*14 functions, 4 structs*

### [`spawn_diag`](spawn_diag.md)

*3 constants, 9 functions*

### [`spawn_path`](spawn_path.md)

*1 enum, 8 functions*

### [`spec`](spec.md)

*1 struct, 3 constants*

