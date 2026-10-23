**bookclerk_plugin_abi > json_bytes > opt_b64**

# Module: json_bytes::opt_b64

## Contents

**Functions**

- [`deserialize`](#deserialize) - Deserializes base64 text, a byte array, or `null`.
- [`serialize`](#serialize) - Serializes optional bytes as base64 text or `null`.

---

## bookclerk_plugin_abi::json_bytes::opt_b64::deserialize

*Function*

Deserializes base64 text, a byte array, or `null`.

# Errors

Returns when the text is not valid base64.

```rust
fn deserialize<'de, D>(deserializer: D) -> Result<Option<Vec<u8>>, <D as >::Error>
```



## bookclerk_plugin_abi::json_bytes::opt_b64::serialize

*Function*

Serializes optional bytes as base64 text or `null`.

# Errors

Propagates serializer failures.

```rust
fn serialize<S>(bytes: &Option<Vec<u8>>, serializer: S) -> Result<<S as >::Ok, <S as >::Error>
```



