**bookclerk_library > sql_plan > host_ir**

# Module: sql_plan::host_ir

## Contents

**Structs**

- [`AtomicSelection`](#atomicselection) - Host indexes into a typed atomic batch (not part of the plugin ABI).

**Functions**

- [`typed_rows_to_json`](#typed_rows_to_json) - Projects typed result rows onto named JSON objects for domain interpretation.

---

## bookclerk_library::sql_plan::host_ir::AtomicSelection

*Struct*

Host indexes into a typed atomic batch (not part of the plugin ABI).

**Fields:**
- `outcome_index: u32` - Index of the application-status `SELECT` when receipts are not used.
- `payload_index: Option<u32>` - Index of a payload `SELECT` when the op returns a JSON record.
- `prior_receipt_index: Option<u32>` - Index of the receipt `SELECT` immediately after prune (replay detect).
- `receipt_select_index: Option<u32>` - Index of the final receipt `SELECT`.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AtomicSelection) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> AtomicSelection`
- **Default**
  - `fn default() -> AtomicSelection`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::sql_plan::host_ir::typed_rows_to_json

*Function*

Projects typed result rows onto named JSON objects for domain interpretation.

```rust
fn typed_rows_to_json(columns: &[bookclerk_plugin_abi::DbColumn], rows: &[bookclerk_plugin_abi::DbRow]) -> Vec<serde_json::Value>
```



