# bookclerk_library::sql_plan::host_ir

Domain result-selection metadata kept beside a typed [`bookclerk_plugin_abi::ExecuteRequest`](../../bookclerk_plugin_abi/db_execute.md#executerequest).

The planner emits [`bookclerk_plugin_abi::ExecuteRequest`](../../bookclerk_plugin_abi/db_execute.md#executerequest) directly. This
module holds only the host-side indexes used to interpret
[`bookclerk_plugin_abi::ExecuteReply`](../../bookclerk_plugin_abi/db_execute.md#executereply) into a named [`crate::DbAtomicResult`](../index.md#dbatomicresult).

| Item | Kind | Summary |
| --- | --- | --- |
| [`AtomicSelection`](#atomicselection) | struct | Host indexes into a typed atomic batch (not part of the plugin ABI). |
| [`typed_rows_to_json`](#typed_rows_to_json) | function | Projects typed result rows onto named JSON objects for domain interpretation. |

## Structs

### AtomicSelection

```rust
pub struct AtomicSelection {
    pub outcome_index: u32,
    pub payload_index: Option<u32>,
    pub prior_receipt_index: Option<u32>,
    pub receipt_select_index: Option<u32>,
}
```

Host indexes into a typed atomic batch (not part of the plugin ABI).

**Fields**

- `outcome_index`: Index of the application-status `SELECT` when receipts are not used.
- `payload_index`: Index of a payload `SELECT` when the op returns a JSON record.
- `prior_receipt_index`: Index of the receipt `SELECT` immediately after prune (replay detect).
- `receipt_select_index`: Index of the final receipt `SELECT`.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Functions

### typed_rows_to_json

```rust
pub fn typed_rows_to_json(columns: &[DbColumn], rows: &[DbRow]) -> Vec<Value>
```

Projects typed result rows onto named JSON objects for domain interpretation.
