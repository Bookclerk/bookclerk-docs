**bookclerk_plugin_abi > json_bytes**

# Module: json_bytes

## Contents

**Modules**

- [`b64`](#b64) - `Vec<u8>` as base64 text.
- [`opt_b64`](#opt_b64) - `Option<Vec<u8>>` as base64 text; `null`/absent is `None`.

**Functions**

- [`decode_text`](#decode_text) - Decodes standard base64 text.
- [`encode`](#encode) - Encodes bytes as standard base64 text (bridge JSON projection).

---

## Module: b64

`Vec<u8>` as base64 text.



## bookclerk_plugin_abi::json_bytes::decode_text

*Function*

Decodes standard base64 text.

# Errors

Returns when the text is not valid base64.

```rust
fn decode_text(text: &str) -> Result<Vec<u8>, String>
```



## bookclerk_plugin_abi::json_bytes::encode

*Function*

Encodes bytes as standard base64 text (bridge JSON projection).

```rust
fn encode(bytes: &[u8]) -> String
```



## Module: opt_b64

`Option<Vec<u8>>` as base64 text; `null`/absent is `None`.



