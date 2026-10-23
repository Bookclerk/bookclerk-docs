# bookclerk_config

Host configuration, files-dir layout, and logging for Bookclerk.

# Audience

Host binaries (`bookclerk`, `bookclerkd`) and in-process host crates — not
guest plugins. Parses `config.toml`, applies env overrides, resolves
`BOOKCLERK_FILES_DIR`, and installs tracing / diagnostics.

Product narrative: `docs/configuration.md`, `docs/diagnostics.md`. Style:
`docs/code-documentation.md`.

## Modules

### [`atomic_replace`](atomic_replace.md)

*1 function*

### [`cookie_flags`](cookie_flags.md)

*3 functions*

### [`database`](database.md)

*1 enum, 2 functions, 4 structs*

### [`desktop`](desktop.md)

*1 function*

### [`diagnostics`](diagnostics.md)

*1 function, 3 structs*

### [`error`](error.md)

*1 enum, 1 type alias*

### [`events`](events.md)

*1 struct*

### [`extras`](extras.md)

*1 constant, 10 functions, 2 enums, 2 structs*

### [`isolation`](isolation.md)

*1 enum*

### [`jobs`](jobs.md)

*2 structs*

### [`journal`](journal.md)

*1 enum, 1 struct, 2 functions*

### [`listen`](listen.md)

*1 struct*

### [`logging`](logging.md)

*1 enum, 2 structs, 3 functions*

### [`media`](media.md)

*1 struct*

### [`naming_profile`](naming_profile.md)

*1 enum, 2 structs*

### [`oidc_broker`](oidc_broker.md)

*1 enum, 2 structs, 6 functions*

### [`operator_auth`](operator_auth.md)

*1 enum, 3 functions*

### [`output`](output.md)

*1 function, 2 structs, 3 enums*

### [`overrides`](overrides.md)

*3 functions*

### [`path_limits`](path_limits.md)

*1 enum, 1 struct, 2 constants, 4 functions*

### [`paths`](paths.md)

*1 struct, 2 constants, 2 functions*

### [`pipeline_opts`](pipeline_opts.md)

*2 enums*

### [`platform`](platform.md)

*1 function*

### [`plugins`](plugins.md)

*2 functions, 4 structs*

### [`redact`](redact.md)

*1 constant, 11 functions*

### [`settings`](settings.md)

*1 function, 2 enums, 7 constants, 9 structs*

