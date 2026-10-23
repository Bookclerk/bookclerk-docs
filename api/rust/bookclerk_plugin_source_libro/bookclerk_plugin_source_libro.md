**bookclerk_plugin_source_libro**

# Module: bookclerk_plugin_source_libro

## Contents

**Modules**

- [`catalog`](#catalog) - Public Libro.fm explore catalog helpers for Discover.
- [`db`](#db) - DB-backed credential storage for Libro.fm accounts.
- [`guest`](#guest) - Guest-process helpers for the external Libro.fm source plugin.

---

## Module: catalog

Public Libro.fm explore catalog helpers for Discover.

Mirrors former `bookclerk-discover` explore HTTP (no account required).
This crate intentionally does **not** depend on `bookclerk-enrich`.



## Module: db

DB-backed credential storage for Libro.fm accounts.

Credentials live in `encrypted_secrets` under `provider =` the plugin id
(`libro`), accessed only through [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope).



## Module: guest

Guest-process helpers for the external Libro.fm source plugin.

These APIs do **not** open the library DB — the host seals credentials and
upserts scan rows via `bookclerk_plugin_host::ExternalSource`. Conversions
between storefront types and the typed ABI live in [`bookclerk_source::abi`](../bookclerk_source/abi.md).



