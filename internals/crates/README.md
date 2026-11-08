# Rust crates

Generated from rustdoc JSON for the Bookclerk workspace's libraries.
To write a native plugin, use the [Rust plugin SDK](../../plugins/sdk/rust/README.md).

## Plugin interface

The plugin ABI, the `plugin.toml` manifest and the plugin author CLI.

| Crate | Description |
| --- | --- |
| [`bookclerk_plugin_abi`](bookclerk_plugin_abi/README.md) | Authoritative Bookclerk plugin Workers RPC ABI (schema + shared types) |
| [`bookclerk_plugin_manifest`](bookclerk_plugin_manifest/README.md) | plugin.toml parse, validate, and canonical format (no host deps) |
| [`bookclerk_plugin_tools`](bookclerk_plugin_tools/README.md) | Bookclerk plugin author CLI (`bookclerk-plugin`: check / fmt / sync-embed / package / smoke) |

## Plugins

First-party plugins: databases, destinations, sources and integrations.

| Crate | Description |
| --- | --- |
| [`bookclerk_plugin_database_d1`](bookclerk_plugin_database_d1/README.md) | Optional Cloudflare D1 library database plugin (Workers RPC guest) |
| [`bookclerk_plugin_database_postgres`](bookclerk_plugin_database_postgres/README.md) | Optional PostgreSQL library database plugin (Workers RPC guest) |
| [`bookclerk_plugin_database_sqlite`](bookclerk_plugin_database_sqlite/README.md) | Platform SQLite library database plugin (Workers RPC guest) |
| [`bookclerk_plugin_destination_local`](bookclerk_plugin_destination_local/README.md) | Local filesystem output plugin: Workers RPC guest for Bookclerk |
| [`bookclerk_plugin_destination_s3`](bookclerk_plugin_destination_s3/README.md) | S3 / MinIO output plugin: Workers RPC guest for Bookclerk |
| [`bookclerk_plugin_integration_audiobookshelf`](bookclerk_plugin_integration_audiobookshelf/README.md) | Audiobookshelf integration plugin: in-process Integration + Workers RPC guest binary |
| [`bookclerk_plugin_source_audible`](bookclerk_plugin_source_audible/README.md) | Audible storefront plugin guest (jailed native binary) |
| [`bookclerk_plugin_source_chirp`](bookclerk_plugin_source_chirp/README.md) | Chirp source plugin: native storefront guest binary |
| [`bookclerk_plugin_source_graphicaudio`](bookclerk_plugin_source_graphicaudio/README.md) | GraphicAudio source plugin: native storefront guest binary |
| [`bookclerk_plugin_source_libro`](bookclerk_plugin_source_libro/README.md) | Libro.fm source plugin: native storefront guest binary |

## Host internals

The crates the hosts are built from. Not a stable API.

| Crate | Description |
| --- | --- |
| [`bookclerk_acquire`](bookclerk_acquire/README.md) | End-to-end acquire pipeline: license, download, decrypt, store |
| [`bookclerk_audible_catalog`](bookclerk_audible_catalog/README.md) | Unauthenticated Audible catalog HTTP (search, genres, ratings, reviews) shared by the Audible guest and host enrichment |
| [`bookclerk_config`](bookclerk_config/README.md) | Bookclerk settings, paths, and naming template config |
| [`bookclerk_cover`](bookclerk_cover/README.md) | Cover image pipeline: sniff, decode under limits, resize and re-encode untrusted cover art as JPEG |
| [`bookclerk_db_exec`](bookclerk_db_exec/README.md) | Neutral SQL executor and transaction injects for database guests |
| [`bookclerk_db_guest`](bookclerk_db_guest/README.md) | First-party database guest session workers (SeaORM proxy, host IR envelopes) |
| [`bookclerk_demo_seed`](bookclerk_demo_seed/README.md) | Fills a fresh hosted-demo library with a second member, their books, wishlist and offline branch |
| [`bookclerk_discover`](bookclerk_discover/README.md) | Recommendations, embeddings, listening sync, and title request helpers |
| [`bookclerk_enrich`](bookclerk_enrich/README.md) | Source-agnostic Audible metadata enrichment (catalog search, metadata providers, confidence scoring) |
| [`bookclerk_integrations`](bookclerk_integrations/README.md) | Outbound integration framework (traits, registry, portal, claim tickets) |
| [`bookclerk_library`](bookclerk_library/README.md) | Library database schema, migrations, and book records |
| [`bookclerk_media`](bookclerk_media/README.md) | Clear-media packaging: remux, metadata fix-up, and MP3 encode (no DRM) |
| [`bookclerk_migrate`](bookclerk_migrate/README.md) | Import/export classic Libation Files and native Bookclerk backups |
| [`bookclerk_mp4`](bookclerk_mp4/README.md) | ISO-BMFF container plumbing shared by media packaging and store plugins (no cryptography) |
| [`bookclerk_naming`](bookclerk_naming/README.md) | Port of Libation's NamingTemplate tag-template engine (Chardonnay parity) |
| [`bookclerk_node_client`](bookclerk_node_client/README.md) | Host-node client: enrollment, the pinned owner socket, heartbeats, claimed work, token write-back, wakes and resume |
| [`bookclerk_node_proto`](bookclerk_node_proto/README.md) | Host-node protocol: node API frames, handshake signatures, key checks, and sealed boxes |
| [`bookclerk_plugin_catalog`](bookclerk_plugin_catalog/README.md) | Registry-neutral plugin discovery, manifests, and secure install |
| [`bookclerk_plugin_host`](bookclerk_plugin_host/README.md) | External plugin host: discovery, spawn, consent, and workerd-fronted guests |
| [`bookclerk_resolve`](bookclerk_resolve/README.md) | Sans-IO resolver that maps store listings to FRBR works, expressions and editions |
| [`bookclerk_sandbox`](bookclerk_sandbox/README.md) | Portable, unprivileged process confinement (Landlock/seccomp, Seatbelt, AppContainer) |
| [`bookclerk_search`](bookclerk_search/README.md) | Full-text library search (Tantivy; classic Lucene parity) |
| [`bookclerk_source`](bookclerk_source/README.md) | Host-side ContentSource trait and source registry; re-exports the storefront types guests share |
| [`bookclerk_storage`](bookclerk_storage/README.md) | Pluggable local and S3 storage backends for acquired audiobooks |
| [`bookclerk_storefront`](bookclerk_storefront/README.md) | Types storefront guests share with the host: DTOs, ABI conversions and listing helpers, with no database, ORM or OpenAPI dependencies |
| [`bookclerk_tls`](bookclerk_tls/README.md) | The one rustls crypto provider and HTTP client TLS settings for the Bookclerk hosts |
| [`bookclerk_tray`](bookclerk_tray/README.md) | In-process system tray for bookclerkd (opens the web UI in a browser) |
| [`bookclerk_workerd`](bookclerk_workerd/README.md) | Jailed workerd isolate launcher for Bookclerk script plugins |

## Development and testing

Build tooling, test suites and examples.

| Crate | Description |
| --- | --- |
| [`bookclerk_dev`](bookclerk_dev/README.md) | Cargo alias dispatcher: build, stage, and run Bookclerk with external plugins |
| [`bookclerk_plugin_e2e`](bookclerk_plugin_e2e/README.md) | End-to-end suite against a staged first-party plugin installation |
| [`bookclerk_plugin_echo_workerd_rust`](bookclerk_plugin_echo_workerd_rust/README.md) | Echo integration — workerd Rust/Wasm guest (JSON dispatch + JS WorkerEntrypoint) |
