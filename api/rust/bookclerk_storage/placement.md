**bookclerk_storage > placement**

# Module: placement

## Contents

**Functions**

- [`node_placement_id`](#node_placement_id) - This node's id, or a process-lifetime id when the node identity cannot be

---

## bookclerk_storage::placement::node_placement_id

*Function*

This node's id, or a process-lifetime id when the node identity cannot be
read.

`BOOKCLERK_PLACEMENT_ID` overrides it for tests. Otherwise the value is
the `node_id` in `$BOOKCLERK_FILES_DIR/node-identity.json`, which the
control plane writes when the node first opens the library. When that is
not available the id lasts for this process only, so a scan is not
adopted by another process that cannot prove it is the same node.

```rust
fn node_placement_id() -> String
```



