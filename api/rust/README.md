# Rust API reference

Generated from rustdoc JSON for the Bookclerk workspace's libraries.
To write a native plugin, use the [Rust plugin SDK](bookclerk_plugin_sdk/index.md).

## Plugin interface

The plugin ABI, the `plugin.toml` manifest and the plugin author CLI.

| Crate | Description |
| --- | --- |
| [`bookclerk_plugin_abi`](bookclerk_plugin_abi/index.md) | Authoritative Bookclerk plugin Workers RPC ABI (schema + shared types) |
| [`bookclerk_plugin_manifest`](bookclerk_plugin_manifest/index.md) | plugin.toml parse, validate, and canonical format (no host deps) |
| [`bookclerk_plugin_tools`](bookclerk_plugin_tools/index.md) | Bookclerk plugin author CLI (`bookclerk-plugin`: check / fmt / sync-embed / package / smoke) |

## Plugins

First-party plugins: databases, destinations, sources and integrations.

| Crate | Description |
| --- | --- |
| [`bookclerk_plugin_database_d1`](bookclerk_plugin_database_d1/index.md) | Optional Cloudflare D1 library database plugin (Workers RPC guest) |
| [`bookclerk_plugin_database_postgres`](bookclerk_plugin_database_postgres/index.md) | Optional PostgreSQL library database plugin (Workers RPC guest) |
| [`bookclerk_plugin_database_sqlite`](bookclerk_plugin_database_sqlite/index.md) | Platform SQLite library database plugin (Workers RPC guest) |
| [`bookclerk_plugin_destination_local`](bookclerk_plugin_destination_local/index.md) | Local filesystem output plugin: Workers RPC guest for Bookclerk |
| [`bookclerk_plugin_destination_s3`](bookclerk_plugin_destination_s3/index.md) | S3 / MinIO output plugin: Workers RPC guest for Bookclerk |
| [`bookclerk_plugin_integration_audiobookshelf`](bookclerk_plugin_integration_audiobookshelf/index.md) | Audiobookshelf integration plugin: in-process Integration + Workers RPC guest binary |
| [`bookclerk_plugin_source_audible`](bookclerk_plugin_source_audible/index.md) | Audible source plugin: in-process ContentSource + Workers RPC guest binary |
| [`bookclerk_plugin_source_chirp`](bookclerk_plugin_source_chirp/index.md) | Chirp source plugin: in-process ContentSource + Workers RPC guest binary |
| [`bookclerk_plugin_source_graphicaudio`](bookclerk_plugin_source_graphicaudio/index.md) | GraphicAudio source plugin: in-process ContentSource + Workers RPC guest binary |
| [`bookclerk_plugin_source_libro`](bookclerk_plugin_source_libro/index.md) | Libro.fm source plugin: in-process ContentSource + Workers RPC guest binary |

## Host internals

The crates the hosts are built from. Not a stable API.

| Crate | Description |
| --- | --- |
| [`bookclerk_acquire`](bookclerk_acquire/index.md) | End-to-end acquire pipeline: license, download, decrypt, store |
| [`bookclerk_config`](bookclerk_config/index.md) | Bookclerk settings, paths, and naming template config |
| [`bookclerk_db_exec`](bookclerk_db_exec/index.md) | Neutral SQL executor and transaction injects for database guests |
| [`bookclerk_db_guest`](bookclerk_db_guest/index.md) | First-party database guest session workers (SeaORM proxy, host IR envelopes) |
| [`bookclerk_discover`](bookclerk_discover/index.md) | Recommendations, embeddings, listening sync, and title request helpers |
| [`bookclerk_enrich`](bookclerk_enrich/index.md) | Source-agnostic Audible metadata enrichment (catalog + Audnexus + confidence scoring) |
| [`bookclerk_integrations`](bookclerk_integrations/index.md) | Outbound integration framework (traits, registry, portal, claim tickets) |
| [`bookclerk_library`](bookclerk_library/index.md) | Library database schema, migrations, and book records |
| [`bookclerk_media`](bookclerk_media/index.md) | Clear-media packaging: remux, metadata fix-up, and MP3 encode (no DRM) |
| [`bookclerk_migrate`](bookclerk_migrate/index.md) | Import/export classic Libation Files and native Bookclerk backups |
| [`bookclerk_mp4`](bookclerk_mp4/index.md) | ISO-BMFF container plumbing shared by media packaging and store plugins (no cryptography) |
| [`bookclerk_naming`](bookclerk_naming/index.md) | Port of Libation's NamingTemplate tag-template engine (Chardonnay parity) |
| [`bookclerk_node_client`](bookclerk_node_client/index.md) | Host-node client: enrollment, the pinned owner socket, heartbeats, claimed work, token write-back, wakes and resume |
| [`bookclerk_node_proto`](bookclerk_node_proto/index.md) | Host-node protocol: node API frames, handshake signatures, key checks, and sealed boxes |
| [`bookclerk_plugin_catalog`](bookclerk_plugin_catalog/index.md) | Registry-neutral plugin discovery, manifests, and secure install |
| [`bookclerk_plugin_host`](bookclerk_plugin_host/index.md) | External plugin host: discovery, spawn, consent, and workerd-fronted guests |
| [`bookclerk_sandbox`](bookclerk_sandbox/index.md) | Portable, unprivileged process confinement (Landlock/seccomp, Seatbelt, AppContainer) |
| [`bookclerk_search`](bookclerk_search/index.md) | Full-text library search (Tantivy; classic Lucene parity) |
| [`bookclerk_source`](bookclerk_source/index.md) | ContentSource trait and shared types for multi-source library providers |
| [`bookclerk_storage`](bookclerk_storage/index.md) | Pluggable local and S3 storage backends for acquired audiobooks |
| [`bookclerk_tray`](bookclerk_tray/index.md) | In-process system tray for bookclerkd (opens the web UI in a browser) |
| [`bookclerk_workerd`](bookclerk_workerd/index.md) | Jailed workerd isolate launcher for Bookclerk script plugins |

## Development and testing

Build tooling, test suites and examples.

| Crate | Description |
| --- | --- |
| [`bookclerk_dev`](bookclerk_dev/index.md) | Cargo alias dispatcher: build, stage, and run Bookclerk with external plugins |
| [`bookclerk_plugin_e2e`](bookclerk_plugin_e2e/index.md) | End-to-end suite against a staged first-party plugin installation |
| [`bookclerk_plugin_echo_workerd_rust`](bookclerk_plugin_echo_workerd_rust/index.md) | Echo integration — workerd Rust/Wasm guest (JSON dispatch + JS WorkerEntrypoint) |
