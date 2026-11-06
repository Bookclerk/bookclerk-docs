# bookclerk_plugin_manifest::id

Display-alias grammar shared by manifest parse, host registry, and SDKs.

The `id` is a presentation alias unique within one host plugin namespace
(`$BOOKCLERK_FILES_DIR`). It is not a durable security identity — that is the
provenance-qualified PluginKey. The grammar is **non-lossy**:
characters that would need rewriting (for example `/` → `_`) are rejected
instead of sanitized, so `a/b` and `a_b` cannot collide as aliases.

## Functions

### validate_plugin_id

```rust
pub fn validate_plugin_id(id: &str) -> Result<()>
```

Validates a plugin id against the strict install grammar.

Accepted form: `[a-z0-9_]{2,32}` with no leading or trailing `_`, no
consecutive `__`, and no leading/trailing whitespace (whitespace is not
trimmed — `" echo"` ≠ `"echo"`).

#### Arguments

* `id` - Candidate id string exactly as it appears in `plugin.toml` or
  registry input (do not trim beforehand if you need the whitespace check).

#### Returns

`Ok(())` when the id is valid.

#### Errors

Returns [`Error::Message`](error.md#error) describing the first failed rule (length,
charset, underscore placement, or whitespace).

#### Examples

```rust
use bookclerk_plugin_manifest::validate_plugin_id;

assert!(validate_plugin_id("sqlite").is_ok());
assert!(validate_plugin_id("my_store").is_ok());
assert!(validate_plugin_id("a/b").is_err());
assert!(validate_plugin_id("_bad").is_err());
```
