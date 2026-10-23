**bookclerk_plugin_sdk > protocol**

# Module: protocol

## Contents

**Modules**

- [`methods`](#methods) - Role capability method name constants (camelCase wire strings).

**Constants**

- [`MAX_RPC_LINE_BYTES`](#max_rpc_line_bytes) - Maximum length of one framed bridge line in bytes (including newline).
- [`PROTOCOL_NAME`](#protocol_name) - Logical ABI identifier for diagnostics (not a `plugin.toml` field).

---

## bookclerk_plugin_sdk::protocol::MAX_RPC_LINE_BYTES

*Constant*: `usize`

Maximum length of one framed bridge line in bytes (including newline).

Used by the workerd bridge framing. Currently 16 MiB.



## bookclerk_plugin_sdk::protocol::PROTOCOL_NAME

*Constant*: `&str`

Logical ABI identifier for diagnostics (not a `plugin.toml` field).

Manifests advertise compatibility via `api_version` only.



## Module: methods

Role capability method name constants (camelCase wire strings).

Re-exports `bookclerk_plugin_abi::methods::names` so guests can reference
capability names such as `methods::LOGIN` without depending on the ABI
crate path directly.



