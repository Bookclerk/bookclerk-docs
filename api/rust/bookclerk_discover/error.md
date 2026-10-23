**bookclerk_discover > error**

# Module: error

## Contents

**Enums**

- [`DiscoverError`](#discovererror) - Errors from embeddings, catalog fan-out, and recommendation pipelines.

---

## bookclerk_discover::error::DiscoverError

*Enum*

Errors from embeddings, catalog fan-out, and recommendation pipelines.

**Variants:**
- `Library(bookclerk_library::LibraryError)` - Error propagated from [`bookclerk_library`].
- `Enrich(bookclerk_enrich::EnrichError)` - Error propagated from [`bookclerk_enrich`].
- `Http(reqwest::Error)` - Outbound HTTP failure talking to a storefront or Open Library.
- `Embed(String)` - Embedding model load / inference failure.
- `Message(String)` - Operator-facing error text with no structured code.



