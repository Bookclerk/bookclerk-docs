**bookclerk_plugin_abi > db_rpc**

# Module: db_rpc

## Contents

**Functions**

- [`canonical_execute_request_hash`](#canonical_execute_request_hash) - SHA-256 hex of the Cap'n-encoded request with transport metadata cleared.
- [`decode_db_value_bytes`](#decode_db_value_bytes) - Decodes a standalone Cap'n `DbValue` message.
- [`decode_execute_request_bytes`](#decode_execute_request_bytes) - Decodes a standalone Cap'n `ExecuteRequest` message.
- [`decode_execute_result_reply_bytes`](#decode_execute_result_reply_bytes) - Decodes a standalone Cap'n `ExecuteAtomicReply` message.
- [`encoded_db_value_bytes`](#encoded_db_value_bytes) - Encodes a standalone Cap'n `DbValue` message (unpacked stream).
- [`encoded_execute_reply_bytes`](#encoded_execute_reply_bytes) - Encodes a standalone Cap'n `ExecuteReply` message (unpacked stream).
- [`encoded_execute_request_bytes`](#encoded_execute_request_bytes) - Encodes a standalone Cap'n `ExecuteRequest` message (unpacked stream).
- [`encoded_execute_result_reply_bytes`](#encoded_execute_result_reply_bytes) - Encodes a standalone Cap'n `ExecuteAtomicReply` (`ok` or `err`).
- [`encoded_statement_result_bytes`](#encoded_statement_result_bytes) - Encodes a standalone Cap'n `StatementResult` message (unpacked stream).

---

## bookclerk_plugin_abi::db_rpc::canonical_execute_request_hash

*Function*

SHA-256 hex of the Cap'n-encoded request with transport metadata cleared.

`operationId`, `requestHash`, and `deadlineUnixMs` are not part of the
digest: a retry of the same mutation may mint a remaining deadline without
hash-conflicting. The trusted host stamps `requestHash` after validation
so guests cannot collide unrelated mutations on one idempotency key.

# Errors

Returns `PluginError::internal` when the message cannot be serialized.

```rust
fn canonical_execute_request_hash(req: &crate::ExecuteRequest) -> crate::Result<String>
```



## bookclerk_plugin_abi::db_rpc::decode_db_value_bytes

*Function*

Decodes a standalone Cap'n `DbValue` message.

# Errors

Returns when the buffer is not a valid unpacked `DbValue`.

```rust
fn decode_db_value_bytes(bytes: &[u8]) -> crate::Result<crate::DbValue>
```



## bookclerk_plugin_abi::db_rpc::decode_execute_request_bytes

*Function*

Decodes a standalone Cap'n `ExecuteRequest` message.

# Errors

Returns when the buffer is not a valid unpacked `ExecuteRequest`.

```rust
fn decode_execute_request_bytes(bytes: &[u8]) -> crate::Result<crate::ExecuteRequest>
```



## bookclerk_plugin_abi::db_rpc::decode_execute_result_reply_bytes

*Function*

Decodes a standalone Cap'n `ExecuteAtomicReply` message.

# Errors

Returns the guest [`PluginError`](error.md#bookclerk_plugin_abierrorpluginerror) on `err`, or a decode failure on `ok`.

```rust
fn decode_execute_result_reply_bytes(bytes: &[u8]) -> crate::Result<crate::ExecuteReply>
```



## bookclerk_plugin_abi::db_rpc::encoded_db_value_bytes

*Function*

Encodes a standalone Cap'n `DbValue` message (unpacked stream).

# Errors

Returns `PluginError::invalid_params` when a `DbValue::Text` contains
U+0000, or `PluginError::internal` when the message cannot be serialized.

```rust
fn encoded_db_value_bytes(v: &crate::DbValue) -> crate::Result<Vec<u8>>
```



## bookclerk_plugin_abi::db_rpc::encoded_execute_reply_bytes

*Function*

Encodes a standalone Cap'n `ExecuteReply` message (unpacked stream).

# Errors

Returns `PluginError::invalid_params` when a TEXT cell contains U+0000,
or `PluginError::internal` when the message cannot be serialized.

```rust
fn encoded_execute_reply_bytes(reply: &crate::ExecuteReply) -> crate::Result<Vec<u8>>
```



## bookclerk_plugin_abi::db_rpc::encoded_execute_request_bytes

*Function*

Encodes a standalone Cap'n `ExecuteRequest` message (unpacked stream).

# Errors

Returns `PluginError::invalid_params` when SQL or TEXT binds contain
U+0000, or `PluginError::internal` when the message cannot be serialized.

```rust
fn encoded_execute_request_bytes(req: &crate::ExecuteRequest) -> crate::Result<Vec<u8>>
```



## bookclerk_plugin_abi::db_rpc::encoded_execute_result_reply_bytes

*Function*

Encodes a standalone Cap'n `ExecuteAtomicReply` (`ok` or `err`).

# Errors

Returns `PluginError::internal` when the message cannot be serialized.

```rust
fn encoded_execute_result_reply_bytes(outcome: crate::Result<crate::ExecuteReply>) -> crate::Result<Vec<u8>>
```



## bookclerk_plugin_abi::db_rpc::encoded_statement_result_bytes

*Function*

Encodes a standalone Cap'n `StatementResult` message (unpacked stream).

Used to enforce negotiated `maxResultBytes` against the actual wire size of
one statement before COMMIT.

# Errors

Returns `PluginError::invalid_params` when a TEXT cell contains U+0000,
or `PluginError::internal` when the message cannot be serialized.

```rust
fn encoded_statement_result_bytes(stmt: &crate::StatementResult) -> crate::Result<Vec<u8>>
```



