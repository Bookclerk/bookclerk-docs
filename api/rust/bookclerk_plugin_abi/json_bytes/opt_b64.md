# bookclerk_plugin_abi::json_bytes::opt_b64

`Option<Vec<u8>>` as base64 text; `null`/absent is `None`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`serialize`](#serialize) | function | Serializes optional bytes as base64 text or `null`. |
| [`deserialize`](#deserialize) | function | Deserializes base64 text, a byte array, or `null`. |

## Functions

### serialize

```rust
pub fn serialize<S: Serializer>(
    bytes: &Option<Vec<u8>>,
    serializer: S,
) -> Result<<S as >::Ok, <S as >::Error>
```

Serializes optional bytes as base64 text or `null`.

#### Errors

Propagates serializer failures.

### deserialize

```rust
pub fn deserialize<'de, D: Deserializer<'de>>(
    deserializer: D,
) -> Result<Option<Vec<u8>>, <D as >::Error>
```

Deserializes base64 text, a byte array, or `null`.

#### Errors

Returns when the text is not valid base64.
