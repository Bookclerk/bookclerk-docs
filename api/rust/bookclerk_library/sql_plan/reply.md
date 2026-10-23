**bookclerk_library > sql_plan > reply**

# Module: sql_plan::reply

## Contents

**Functions**

- [`validate_execute_reply`](#validate_execute_reply) - Rejects a guest [`ExecuteReply`](../../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply) that does not echo the sent request or

---

## bookclerk_library::sql_plan::reply::validate_execute_reply

*Function*

Rejects a guest [`ExecuteReply`](../../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecutereply) that does not echo the sent request or
exceeds negotiated caps.

A mismatch after the guest reports success is treated as
`LibraryError::Unavailable` so callers retry the same `operationId`
rather than interpreting a truncated or mismatched envelope.

# Errors

Returns `LibraryError::Unavailable` when the echo, statement count,
result selection shape, or byte/row bounds do not match the request.

```rust
fn validate_execute_reply(req: &bookclerk_plugin_abi::ExecuteRequest, reply: &bookclerk_plugin_abi::ExecuteReply, caps: &bookclerk_plugin_abi::DbCapabilities) -> crate::error::Result<()>
```



