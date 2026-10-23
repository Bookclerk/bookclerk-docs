**bookclerk_plugin_abi > host_roles**

# Module: host_roles

## Contents

**Traits**

- [`AdapterTransaction`](#adaptertransaction) - Host-only interactive transaction on an adapter connection (SeaORM proxy).
- [`HostAdapterDatabaseSession`](#hostadapterdatabasesession) - Host-only view of an open adapter session (`begin` for interactive txn).

---

## bookclerk_plugin_abi::host_roles::AdapterTransaction

*Trait*

Host-only interactive transaction on an adapter connection (SeaORM proxy).

**Methods:**

- `execute`: Typed statements on this open transaction (no second `BEGIN`).
- `commit`: Commit.
- `rollback`: Rollback.
- `export_identity`: Identity high-water from adapter catalogs on this transaction.
- `import_identity`: Restore identity high-water into adapter catalogs on this transaction.
- `list_user_relations`: User-visible relation names visible to this transaction.
- `prepare_unit_restore`: Prepare this restore transaction (deferred FK checks).
- `drop_user_relations`: Drop named user relations on this transaction.
- `assert_restore_constraints`: Fail closed when this restore transaction still has FK violations.



## bookclerk_plugin_abi::host_roles::HostAdapterDatabaseSession

*Trait*

Host-only view of an open adapter session (`begin` for interactive txn).

**Methods:**

- `begin`: Opens a host-internal interactive transaction (SeaORM proxy).
- `execute`: Typed atomic execute on this host-private session.



