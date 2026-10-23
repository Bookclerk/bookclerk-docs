**bookclerk_plugin_source_graphicaudio**

# Module: bookclerk_plugin_source_graphicaudio

## Contents

**Modules**

- [`db`](#db) - DB-backed credential storage for GraphicAudio accounts.
- [`guest`](#guest) - Guest-process helpers for the external GraphicAudio source plugin.

---

## Module: db

DB-backed credential storage for GraphicAudio accounts.

Credentials live in `encrypted_secrets` under `provider =` the plugin id
(`graphicaudio`), accessed only through [`SourceScope`](../bookclerk_library/scope.md#bookclerk_libraryscopesourcescope).



## Module: guest

Guest-process helpers for the external GraphicAudio source plugin.

These APIs do **not** open the library DB — the host seals credentials and
upserts scan rows via `bookclerk_plugin_host::ExternalSource`. Conversions
between storefront types and the typed ABI live in [`bookclerk_source::abi`](../bookclerk_source/abi.md).



