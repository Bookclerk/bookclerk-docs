**bookclerk_plugin_source_audible**

# Module: bookclerk_plugin_source_audible

## Contents

**Modules**

- [`catalog`](#catalog) - Public Audible catalog helpers for Discover (`search_catalog` / expand / purchase).
- [`db`](#db) - DB-backed credential storage for Audible accounts.

---

## Module: catalog

Public Audible catalog helpers for Discover (`search_catalog` / expand / purchase).

Uses [`bookclerk_enrich`](../bookclerk_enrich/index.md) public catalog HTTP — no account required.



## Module: db

DB-backed credential storage for Audible accounts.

Credentials live in `encrypted_secrets` under `provider =` the plugin id
(`audible`), accessed only through [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope) — the same boundary as
third-party source plugins.

Each credential is stored as `format = "sealed-v1"` wrapping a
`Protection::Plain` audible-rs envelope. The outer XChaCha20-Poly1305 seal
(process DEK from `master.key`) provides at-rest protection; audible-rs
inner encryption is intentionally bypassed so key derivation happens once
at startup rather than on every credential access.



