**bookclerk_media > mp3**

# Module: mp3

## Contents

**Functions**

- [`write_output_within_budget`](#write_output_within_budget) - Appends `data` only when the result stays within `max`.

---

## bookclerk_media::mp3::write_output_within_budget

*Function*

Appends `data` only when the result stays within `max`.

The failing call writes nothing, so the file length never passes the budget.
`max == None` does not impose a byte ceiling.

# Errors

Returns `MediaError::Native` when `written + data.len()` would exceed
`max`. Returns `MediaError::Io` when the accepted bytes cannot be written.

```rust
fn write_output_within_budget<impl Write>(file: & mut impl Trait, data: &[u8], written: & mut u64, max: Option<u64>) -> crate::error::Result<()>
```



