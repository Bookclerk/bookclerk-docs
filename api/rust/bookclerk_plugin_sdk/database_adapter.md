**bookclerk_plugin_sdk > database_adapter**

# Module: database_adapter

## Contents

**Modules**

- [`errors`](#errors) - Engine error classification for SQLite and PostgreSQL guests.
- [`migrate`](#migrate) - Generic SQL helpers for database guests.

---

## Module: errors

Engine error classification for SQLite and PostgreSQL guests.

Classifies by SQLSTATE / rusqlite codes first. English substrings such as
`"unique"` are not the primary signal. COMMIT-time and connection-loss
failures are `unavailable` so the host retries the same `operationId`.



## Module: migrate

Generic SQL helpers for database guests.

Guests may run host-provided typed statements. They must not select
Bookclerk schema versions or import `bookclerk_library::migrations`.
Schema packs arrive as already-separate statements — do not split on `;`.



