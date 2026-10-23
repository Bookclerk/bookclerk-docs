# bookclerk_jail

Applies a confinement policy to itself, then becomes the program it was
asked to run.

```text
BOOKCLERK_JAIL_SPEC='{"label":"plugin:libro", …}' bookclerk-jail /path/to/guest
```

# Why a separate process

Landlock and Seatbelt restrictions are inherited across `exec` and cannot be
relaxed afterwards, so a parent that confines itself and then `exec`s another
binary hands that binary a jail it has no way to refuse. That is the only
arrangement that works for a plugin, because the plugin binary *is* the
untrusted part: a guest asked to confine itself would simply not.

Doing it in the host instead is not an option. Both backends allocate, which
is unsafe after `fork` in a threaded process, and Landlock's `restrict_self`
binds the calling thread rather than the process — a runtime's worker threads
would stay unconfined. This binary is single-threaded, so it has neither
problem.

# Windows

Windows has no self-confinement primitive. The launcher therefore
`CreateProcess`es the guest into an AppContainer (via
[`bookclerk_sandbox::spawn::run_appcontainer`](../bookclerk_sandbox/platform/windows_spawn.md#bookclerk_sandboxplatformwindows_spawnrun_appcontainer)) and proxies stdio until the
guest exits, instead of confining itself and `exec`ing.

## Modules

### [`fds`](fds.md)

*1 function*

