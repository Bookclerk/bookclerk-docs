**bookclerk_sandbox > platform > windows_pipe**

# Module: platform::windows_pipe

## Contents

**Structs**

- [`NamedPipeSecurity`](#namedpipesecurity) - Owned `SECURITY_ATTRIBUTES` suitable for `CreateNamedPipe` to one guest.

---

## bookclerk_sandbox::platform::windows_pipe::NamedPipeSecurity

*Struct*

Owned `SECURITY_ATTRIBUTES` suitable for `CreateNamedPipe` to one guest.

Keep this value alive until the create call returns; the kernel copies the
descriptor onto the pipe object.

**Methods:**

- `fn for_app_container(package_sid: &str) -> Result<Self, SandboxError>` - Build attributes that allow `package_sid` to open a duplex named pipe.
- `fn as_mut_ptr(self: & mut Self) -> *mut std::ffi::c_void` - Raw pointer for Tokio `create_with_security_attributes_raw` / Win32.

**Trait Implementations:**

- **Drop**
  - `fn drop(self: & mut Self)`



