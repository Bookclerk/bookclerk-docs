**bookclerk_config > paths**

# Module: paths

## Contents

**Structs**

- [`Paths`](#paths) - Resolved filesystem locations for Bookclerk data.

**Functions**

- [`resolve_config_path`](#resolve_config_path) - Resolve config file path: `BOOKCLERK_CONFIG` → `{files_dir}/config.toml`.
- [`resolve_files_dir`](#resolve_files_dir) - Resolve the Bookclerk files directory.

**Constants**

- [`BOOKCLERK_CONFIG_ENV`](#bookclerk_config_env) - Override for the config file path (`BOOKCLERK_CONFIG`).
- [`BOOKCLERK_FILES_DIR_ENV`](#bookclerk_files_dir_env) - Environment variable for the Bookclerk data directory.

---

## bookclerk_config::paths::BOOKCLERK_CONFIG_ENV

*Constant*: `&str`

Override for the config file path (`BOOKCLERK_CONFIG`).



## bookclerk_config::paths::BOOKCLERK_FILES_DIR_ENV

*Constant*: `&str`

Environment variable for the Bookclerk data directory.



## bookclerk_config::paths::Paths

*Struct*

Resolved filesystem locations for Bookclerk data.

**Fields:**
- `files_dir: std::path::PathBuf` - Root data directory (accounts, DB, settings).
- `config_file: std::path::PathBuf` - Resolved config.toml path (may be overridden by `--config` / `BOOKCLERK_CONFIG`).
- `library_db: std::path::PathBuf` - SQLite library database path.
- `cache_dir: std::path::PathBuf` - Temporary download / decrypt scratch space.
- `log_dir: std::path::PathBuf` - Reserved path (legacy). Logging goes to stderr + journald; Bookclerk does
- `search_index_dir: std::path::PathBuf` - Tantivy full-text search index (classic Lucene `SearchEngine`).
- `models_dir: std::path::PathBuf` - Downloaded ONNX embedding models (`discovery.embeddings_enabled`).

**Methods:**

- `fn from_files_dir(files_dir: PathBuf) -> Self` - Build path set from an explicit files directory.
- `fn ensure_dirs(self: &Self) -> std::io::Result<()>` - Ensure directories that Bookclerk expects to exist are present.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> Paths`
- **PartialEq**
  - `fn eq(self: &Self, other: &Paths) -> bool`



## bookclerk_config::paths::resolve_config_path

*Function*

Resolve config file path: `BOOKCLERK_CONFIG` → `{files_dir}/config.toml`.

```rust
fn resolve_config_path(files_dir: &std::path::Path) -> std::path::PathBuf
```



## bookclerk_config::paths::resolve_files_dir

*Function*

Resolve the Bookclerk files directory.

Precedence:
1. Explicit `--bookclerk-files` / caller-provided path
2. `BOOKCLERK_FILES_DIR`
3. XDG data dir: `$XDG_DATA_HOME/bookclerk` (or platform equivalent)
4. Fallback: `./BookclerkFiles` (cwd-relative, Docker-friendly)

```rust
fn resolve_files_dir(cli_override: Option<std::path::PathBuf>) -> std::path::PathBuf
```



