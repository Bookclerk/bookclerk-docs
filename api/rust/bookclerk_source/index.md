# bookclerk_source

Shared traits and DTOs for Bookclerk content-source plugins.

# Audience

Host registration code and first-party / third-party **source plugin**
authors implementing [`ContentSource`](traits.md#bookclerk_sourcetraitscontentsource). Native guests and the plugin host
map these types onto the typed Cap'n Proto ABI through [`abi`](abi.md); workerd
guests use the generated TypeScript / Python SDK types instead.

Product narrative: `docs/plugins.md`. Style: `docs/code-documentation.md`.

## Modules

### [`bookclerk_source`](bookclerk_source.md)

*1 module*

### [`abi`](abi.md)

*15 functions, 2 constants*

### [`brand`](brand.md)

*1 struct*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`language`](language.md)

*3 functions*

### [`media`](media.md)

*4 functions*

### [`options`](options.md)

*1 struct*

### [`registry`](registry.md)

*1 struct*

### [`traits`](traits.md)

*1 enum, 1 trait, 2 functions*

### [`types`](types.md)

*1 type alias, 15 structs, 4 enums*

