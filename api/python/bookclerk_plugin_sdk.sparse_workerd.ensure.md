# bookclerk\_plugin\_sdk.sparse\_workerd.ensure

Download / refresh the pinned Cloudflare ``workerd`` binary (mirrors ensure.rs).

Resolves the platform asset from ``workerd-pin.json``, verifies sha256, and
caches under the operator-selected directory (``BOOKCLERK_WORKERD_CACHE`` or
``~/.cache/bookclerk/workerd``).

#### package\_root

```python
def package_root() -> Path
```

Return the ``bookclerk_plugin_sdk`` package directory.

**Returns**:

  Absolute path containing ``workerd.py``, ``bridge/``, and the pin file.

#### load\_pin

```python
def load_pin(root: Path | None = None) -> dict[str, Any]
```

Load the pinned workerd release metadata.

**Arguments**:

- `root` - Package root containing ``workerd-pin.json`` (defaults to this SDK).
  

**Returns**:

  Parsed pin dictionary (release tag, assets, version stamp).
  

**Raises**:

- `FileNotFoundError` - If the pin file is missing.
- `json.JSONDecodeError` - If the pin file is not valid JSON.

#### platform\_key

```python
def platform_key(system: str | None = None,
                 machine: str | None = None) -> str | None
```

Map OS/arch to a pin asset key such as ``linux-x86_64``.

**Arguments**:

- `system` - OS name override (defaults to ``platform.system()``).
- `machine` - CPU arch override (defaults to ``platform.machine()``).
  

**Returns**:

  Pin key string, or ``None`` when the platform is unsupported.

#### binary\_name

```python
def binary_name() -> str
```

Return the on-disk workerd binary filename for this OS.

**Returns**:

  ``workerd.exe`` on Windows, otherwise ``workerd``.

#### download\_url

```python
def download_url(pin: dict[str, Any], artifact: str) -> str
```

Build the GitHub release download URL for a workerd artifact.

**Arguments**:

- `pin` - Loaded pin metadata containing ``release_tag``.
- `artifact` - Asset filename from the pin ``assets`` table.
  

**Returns**:

  Absolute HTTPS URL to the Cloudflare workerd release asset.

#### default\_cache\_dir

```python
def default_cache_dir() -> Path
```

Resolve the workerd binary cache directory.

Honors ``BOOKCLERK_WORKERD_CACHE`` as the selected install root (any
writable location, including paths outside ``$HOME`` and this package).
Otherwise uses ``~/.cache/bookclerk/workerd``. Derived files stay under
that root.

**Returns**:

  Path to the cache directory (may not exist yet).

#### validate\_fetch\_url

```python
def validate_fetch_url(url: str) -> str
```

Validate a URL before ``urlopen`` / download (request-forgery guard).

Allows ``https:`` anywhere, or ``http:`` only to loopback hosts.

**Arguments**:

- `url` - Absolute URL string.
  

**Returns**:

  The same URL when it passes scheme/host checks.
  

**Raises**:

- `ValueError` - When the URL is invalid or uses a disallowed scheme/host.

#### validate\_spawn\_executable

```python
def validate_spawn_executable(bin_path: Path | str,
                              trusted_root: Path | str | None = None) -> Path
```

Validate a workerd (or helper) binary path before ``subprocess`` spawn.

Requires an absolute path with no NUL bytes. When ``trusted_root`` is set,
requires the binary to stay under that root via :func:`resolve_under`.

**Arguments**:

- `bin_path` - Candidate executable path.
- `trusted_root` - Optional directory the binary must remain under.
  

**Returns**:

  Absolute validated :class:`~pathlib.Path`.
  

**Raises**:

- `ValueError` - When the path is relative, contains NUL, or escapes
  ``trusted_root``.

#### ensure\_workerd

```python
def ensure_workerd(cache_dir: Path | None = None,
                   root: Path | None = None) -> Path
```

Ensure a pinned ``workerd`` binary is available locally.

Reuses ``BOOKCLERK_WORKERD_BIN`` or a cached binary when that file exists
and the version stamp or ``--version`` output matches the pin. The selected
cache directory is the trusted install root (not this package directory).
A stamp without a binary is a cache miss.

**Arguments**:

- `cache_dir` - Override cache directory (defaults to :func:`default_cache_dir`).
- `root` - SDK root for loading ``workerd-pin.json``.
  

**Returns**:

  Path to an executable workerd binary matching the pin.
  

**Raises**:

- `RuntimeError` - If no asset exists for this platform or the download hash
  mismatches.
- `OSError` - If the binary cannot be written or executed.
- `ValueError` - If a resolved binary path fails spawn validation.

