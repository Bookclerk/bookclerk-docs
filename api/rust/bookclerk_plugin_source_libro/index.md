# bookclerk_plugin_source_libro

Libro.fm content source: auth, library sync, and DRM-free download.

Uses the unofficial mobile API documented by community clients (see
`client` module). Audible ASIN enrichment (metadata confidence matching)
lives in `bookclerk-enrich` / CLI — this crate must not depend on
`bookclerk-audible` or `bookclerk-enrich`. Public explore catalog helpers
live in [`catalog`](catalog.md) (SDK HTTP, no enrich).

## Modules

### [`bookclerk_plugin_source_libro`](bookclerk_plugin_source_libro.md)

*3 modules*

### [`auth`](auth.md)

*1 struct*

### [`catalog`](catalog.md)

*4 functions*

### [`client`](client.md)

*1 enum, 11 structs, 8 constants*

### [`container`](container.md)

*1 enum*

### [`db`](db.md)

*4 functions*

### [`download`](download.md)

*2 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`guest`](guest.md)

*9 functions*

### [`source`](source.md)

*1 constant, 1 struct*

### [`sync`](sync.md)

*1 struct, 4 functions*

