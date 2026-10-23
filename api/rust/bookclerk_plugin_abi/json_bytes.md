# bookclerk_plugin_abi::json_bytes

Serde helpers projecting Cap'n Proto `Data` fields as base64 text in JSON.

The typed ABI travels as Cap'n Proto; JSON projections exist only for
transport-private bridges (the workerd HTTP bridge), operator
diagnostics, and tests. Bytes serialize as standard base64 (padded) and
deserialize from either base64 text or a JSON number array, so hand-written
fixtures stay readable.

| Item | Kind | Summary |
| --- | --- | --- |
| [`encode`](#encode) | function | Encodes bytes as standard base64 text (bridge JSON projection). |
| [`decode_text`](#decode_text) | function | Decodes standard base64 text. |

## Modules

- [`b64`](json_bytes/b64.md): `Vec<u8>` as base64 text.
- [`opt_b64`](json_bytes/opt_b64.md): `Option<Vec<u8>>` as base64 text; `null`/absent is `None`.

## Functions

### encode

```rust
pub fn encode(bytes: &[u8]) -> String
```

Encodes bytes as standard base64 text (bridge JSON projection).

### decode_text

```rust
pub fn decode_text(text: &str) -> Result<Vec<u8>, String>
```

Decodes standard base64 text.

#### Errors

Returns when the text is not valid base64.
