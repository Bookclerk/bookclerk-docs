# bookclerk_acquire

Acquire orchestration: license → download → decrypt → metadata → storage.

Host-facing pipeline for fetching a storefront title, packaging audio, and
writing it to one or more output destinations. DRM decrypt happens inside
content-source plugins; this crate never sees license keys. See
`docs/media.md` for encode/remux details.

## Modules

### [`convert`](convert.md)

*1 function, 2 structs*

### [`destinations`](destinations.md)

*1 function, 2 structs*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`match_storage`](match_storage.md)

*2 functions, 2 structs*

### [`naming`](naming.md)

*1 struct, 10 functions*

### [`pipeline`](pipeline.md)

*2 constants, 3 structs, 8 functions*

### [`reconcile`](reconcile.md)

*3 structs, 4 functions*

### [`storage_scan`](storage_scan.md)

*1 function, 1 struct*

