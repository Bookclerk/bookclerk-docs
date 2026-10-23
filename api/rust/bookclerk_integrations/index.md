# bookclerk_integrations

Outbound integrations and SPA portal APIs.

Host binaries should depend on [`Integration`](traits.md#bookclerk_integrationstraitsintegration) / [`IntegrationRegistry`](registry.md#bookclerk_integrationsregistryintegrationregistry) only.
First-party adapters (e.g. Audiobookshelf) live in
`bookclerk-plugin-integration-*` crates and run as staged external guests.

## Modules

### [`brand`](brand.md)

*1 struct*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`factory`](factory.md)

*2 functions*

### [`hooks`](hooks.md)

*1 function*

### [`listening`](listening.md)

*2 functions*

### [`portal::routes`](portal/routes.md)

*1 struct, 4 functions*

### [`registry`](registry.md)

*1 constant, 1 struct*

### [`tickets`](tickets.md)

*1 struct, 8 functions*

### [`traits`](traits.md)

*1 trait, 4 structs*

### [`types`](types.md)

*5 structs*

