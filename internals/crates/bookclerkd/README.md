# bookclerkd

`bookclerkd` — long-running scan / acquire daemon with HTTP control plane.

The library holds the whole host. The `bookclerkd` binary runs it behind
the HTTP listeners ([`run`](#run)); the desktop window runs the same host with
no listener and hands it requests in-process ([`embedded`](embedded.md)).

| Item | Kind | Summary |
| --- | --- | --- |
| [`Args`](#args) | struct | `bookclerkd` CLI flags (files dir, config path, listen overrides). |
| [`OpenApiVersion`](#openapiversion) | enum | Which OpenAPI version `--print-openapi` writes. |
| [`run`](#run) | function | Runs the daemon behind its HTTP listeners until the process is asked to stop (Ctrl+C, SIGTERM) or to restart onto new bootstrap files. |

## Modules

- [`embedded`](embedded.md): The host embedded in a desktop window, with no listener.

## Structs

### Args

```rust
pub struct Args {
    pub bookclerk_files: Option<PathBuf>,
    pub config: Option<PathBuf>,
    pub listen: Option<String>,
    pub print_openapi: Option<OpenApiVersion>,
}
```

`bookclerkd` CLI flags (files dir, config path, listen overrides).

**Fields**

- `bookclerk_files`: Bookclerk files directory.
- `config`: Path to config.toml.
- `listen`: Override HTTP listen address(es), comma-separated
  (`127.0.0.1:8787,[::1]:8787`).
- `print_openapi`: Print the HTTP API's OpenAPI document (JSON) to stdout and exit,
  without reading any configuration. `3.1` (the default) is the
  document the daemon serves at `/api/openapi.json`; `3.0` is the same
  API restated for tools that only read OpenAPI 3.0.

Implements: `Args`, `CommandFactory`, `Debug`, `FromArgMatches`, `Parser`

## Enums

### OpenApiVersion

```rust
pub enum OpenApiVersion {
    V31,
    V30,
}
```

Which OpenAPI version `--print-openapi` writes.

**Variants**

- `V31`: OpenAPI 3.1, the document the daemon serves at `/api/openapi.json`.
- `V30`: OpenAPI 3.0.3, served at `/api/openapi-3.0.json`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`, `ValueEnum`

## Functions

### run

```rust
pub async fn run(args: Args) -> Result<()>
```

Runs the daemon behind its HTTP listeners until the process is asked to
stop (Ctrl+C, SIGTERM) or to restart onto new bootstrap files.

#### Arguments

* `args` - The parsed command line.

#### Errors

Returns an error when the configuration cannot be loaded, the host cannot
start (the control plane, an interrupted shared branch join, operator auth),
`daemon.listen` is invalid, or a restart cannot re-exec the binary.
