# bookclerk_plugin_host

External plugin host for Bookclerk (`bookclerk-plugin-host`).

Discovers staged guests under `$BOOKCLERK_FILES_DIR/plugins/` and spawns
them over the Workers RPC ABI through the mandatory `bookclerk-workerd`
front door (`api_version = 3` Cap'n Proto on stdio).

Production hosts never link ordinary plugin implementation crates.
First-party storefronts (Audible, Libro.fm, …) are staged guests, the
same path third-party plugins use. Database adapters own physical
lowering behind the typed `databaseAdapter` capability (`openSession`,
`dropUnit`); production hosts do not link `bookclerk-plugin-database-*`
crates to perform product behavior.

External plugins are **untrusted** relative to the host: the host never
passes `library.db` / `master.key` / the files-dir root, clears
secret-bearing env on spawn, and mediates credentials + library upserts.
Operators must `bookclerk plugins approve` domains/bindings before enable;
the same covering grant is required again at every external spawn and at
privileged delivery points (`config` / `secrets` / `work_fs` / `oauth`).
Durable identity is provenance-qualified `PluginKey`;
the manifest `id` is a display / CLI alias.

Host binaries should depend on **this** crate for registration — not on
individual store crates.

# Guest SDK

Third-party Rust plugins should depend on [`bookclerk_plugin_sdk`](../bookclerk_plugin_sdk/bookclerk_plugin_sdk.md), not this
crate. This host crate re-exports the protocol types for in-tree convenience.

See `docs/plugins.md`, `docs/adr/plugin-workers-rpc-workerd.md`, and
`docs/plugin-registry.md`.

## Modules

### [`authority`](authority.md)

*1 constant, 1 type alias, 15 functions*

### [`callback_proxy`](callback_proxy.md)

*1 struct*

### [`consent`](consent.md)

*21 constants, 4 structs, 46 functions*

### [`crates_io`](crates_io.md)

*1 function*

### [`deployment`](deployment.md)

*1 constant, 1 enum, 1 trait, 2 functions, 4 structs*

### [`destinations`](destinations.md)

*2 functions*

### [`discover`](discover.md)

*1 struct, 18 functions*

### [`enrollment`](enrollment.md)

*1 struct, 4 functions*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`event_publisher`](event_publisher.md)

*2 structs*

### [`host::database`](host/database.md)

*2 structs, 7 functions*

### [`host::destination`](host/destination.md)

*2 structs*

### [`host::integration`](host/integration.md)

*1 struct*

### [`host::plugin_backups`](host/plugin_backups.md)

*2 functions*

### [`host::source`](host/source.md)

*1 struct*

### [`host_control_plane`](host_control_plane.md)

*3 structs, 5 functions*

### [`install`](install.md)

*1 enum, 15 functions, 2 constants, 4 structs*

### [`install_preflight`](install_preflight.md)

*4 functions*

### [`instance_bindings`](instance_bindings.md)

*1 struct, 3 functions*

### [`jail`](jail.md)

*2 functions*

### [`kv_binding`](kv_binding.md)

*1 constant, 2 structs*

### [`queue_producer`](queue_producer.md)

*2 structs*

### [`registry`](registry.md)

*3 constants, 3 functions, 3 structs*

### [`remote_consent`](remote_consent.md)

*1 struct, 2 enums, 8 functions*

### [`rpc_session`](rpc_session.md)

*1 function, 1 type alias, 2 constants, 5 structs*

### [`spawn_plan`](spawn_plan.md)

*2 constants, 2 enums, 2 structs*

### [`spawn_stdio`](spawn_stdio.md)

*4 constants, 4 functions*

