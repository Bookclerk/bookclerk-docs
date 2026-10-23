**bookclerk_plugin_database_d1**

# Module: bookclerk_plugin_database_d1

## Contents

**Modules**

- [`atomic`](#atomic) - Generic D1 HTTP `batch()` executor for host-authored SQL plans.
- [`d1`](#d1) - Cloudflare D1 HTTP API proxy for SeaORM.

---

## Module: atomic

Generic D1 HTTP `batch()` executor for host-authored SQL plans.

The guest does not parse Bookclerk operation names. Production and tests
use `D1Proxy::run_typed_atomic` (`ExecuteRequest`).



## Module: d1

Cloudflare D1 HTTP API proxy for SeaORM.

D1 HTTP requests do not keep an interactive SQL transaction open.
Autocommit statements use the documented `{ "sql", "params" }` REST body.
Interactive `BEGIN` is rejected: Time Travel is not a substitute for rollback, and
mid-transaction SeaORM reads cannot be satisfied without committing.
Atomic library operations use `D1Proxy::run_typed_atomic`, which sends
`{ "batch": [...] }` (a real SQL transaction) with control flow
encoded in SQL and a durable `operationId` receipt.



