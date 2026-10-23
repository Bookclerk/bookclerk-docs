# bookclerk_storage

Storage backends for acquired audiobooks, covers, and PDFs.

# Audience

Host acquire / library code that writes through [`StorageBackend`](traits.md#bookclerk_storagetraitsstoragebackend). Guest
plugins do not depend on this crate.

Product narrative: destination sections in `docs/configuration.md`. Style:
`docs/code-documentation.md`.

## Modules

### [`bounded`](bounded.md)

*1 enum, 3 constants, 6 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`fanout`](fanout.md)

*1 struct*

### [`local`](local.md)

*1 struct*

### [`placement`](placement.md)

*1 function*

### [`s3`](s3.md)

*3 structs*

### [`traits`](traits.md)

*1 constant, 1 trait, 2 functions, 6 structs*

### [`transfer`](transfer.md)

*1 enum, 1 function, 1 struct*

