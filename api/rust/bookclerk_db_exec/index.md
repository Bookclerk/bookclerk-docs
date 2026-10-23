# bookclerk_db_exec

Neutral SQL executor / adapter SDK for Bookclerk database guests.

Hosts compile domain work into a typed [`bookclerk_plugin_abi::ExecuteRequest`](../bookclerk_plugin_abi/db_execute.md#bookclerk_plugin_abidb_executeexecuterequest)
of **canonical** Bookclerk SQL (`?` placeholders, host-desugared NULLS /
`NULLIF`). This crate owns physical lowering and native execution. It must
not import Bookclerk entities or host domain planners. Postgres adapters
lower host-schema packs and binding DDL types at execute
([`schema_sql_for_backend`](schema_postgres.md#bookclerk_db_execschema_postgresschema_sql_for_backend), [`lower_binding_ddl_execute_request`](schema_postgres.md#bookclerk_db_execschema_postgreslower_binding_ddl_execute_request));
[`lower_canonical_sql`](lower.md#bookclerk_db_execlowerlower_canonical_sql) stays DML/query helpers. Host crates must not call
those lowerers.

## Modules

### [`bookclerk_db_exec`](bookclerk_db_exec.md)

*2 functions, 3 modules*

### [`b64`](b64.md)

*2 functions*

### [`backup`](backup.md)

*9 functions*

### [`canonical`](canonical.md)

*3 functions*

### [`classify`](classify.md)

*1 enum, 3 functions*

### [`exec`](exec.md)

*2 enums, 2 structs, 8 functions*

### [`guest_receipt`](guest_receipt.md)

*11 functions, 8 constants*

### [`lower`](lower.md)

*5 functions*

### [`proxy_txn`](proxy_txn.md)

*2 enums, 2 structs, 27 functions*

### [`schema_postgres`](schema_postgres.md)

*14 functions*

### [`sql_v1`](sql_v1.md)

*1 enum, 42 functions, 69 constants*

### [`typed`](typed.md)

*18 functions*

