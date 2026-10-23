# bookclerk\_plugin\_sdk.path\_guard

Path containment helpers for authoring tools and sparse workerd launchers.

#### resolve\_under

```python
def resolve_under(root: Path | str, *parts: str | Path) -> Path
```

Join ``parts`` under ``root`` and require the result stay inside ``root``.

Lexical ``abspath`` / ``normpath`` + ``relpath`` / ``startswith`` barrier
(no ``realpath`` before the check — that is itself a path sink under local
threat modeling). Callers that write under a plugin tree must also use
:func:`refuse_symlink_path` so a ``.bookclerk -> /outside`` link cannot
redirect generated output.

**Arguments**:

- `root` - Trusted directory (user-selected plugin/output root, or cache).
- `*parts` - Relative segments to join, or one absolute path to validate
  under ``root``.
  

**Returns**:

  Absolute normalized path under ``root``.
  

**Raises**:

- `ValueError` - When a part contains ``..`` / NUL or the result escapes
  ``root``.

#### ensure\_dir\_under

```python
def ensure_dir_under(root: Path | str, rel: str | Path) -> Path
```

Create ``root`` / ``rel`` after resolve + ``startswith`` (check before mkdir).

#### write\_file\_under

```python
def write_file_under(root: Path | str, name: str,
                     contents: str | bytes) -> Path
```

Write ``contents`` to ``root`` / ``name`` after resolve + ``startswith``.

Assumes ``root`` already exists (session/cache dirs are created first).
Refuses a symlink at the destination, including a leaf link out of ``root``.
Filenames may contain ``..`` (``edition..2.js``); ``.`` and ``..`` components
are rejected.

#### copy\_file\_under

```python
def copy_file_under(root: Path | str, name: str, src: Path | str) -> Path
```

Copy ``src`` to ``root`` / ``name`` after resolve + ``startswith`` on dest.

#### refuse\_symlink\_existing\_components

```python
def refuse_symlink_existing_components(trusted_root: Path | str,
                                       path: Path | str) -> Path
```

Like :func:`refuse_symlink_path`, but any trailing missing components are OK.

Use before ``os.makedirs(..., exist_ok=True)`` so a symlink intermediate
cannot redirect recursive directory creation outside ``trusted_root``.

#### refuse\_symlink\_path

```python
def refuse_symlink_path(trusted_root: Path | str, path: Path | str) -> Path
```

Require ``path`` under ``trusted_root`` with no symlink suffix components.

Resolves the trusted root identity once (a symlinked operator/plugin root is
allowed). Only components *below* that identity are inspected for symlinks.
Missing final components are allowed (for create); intermediate missing
parents raise.

**Arguments**:

- `trusted_root` - Original operator/plugin root (not a promoted child).
- `path` - Candidate path previously produced by :func:`resolve_under`.
  

**Returns**:

  The validated ``path`` as a :class:`~pathlib.Path`.
  

**Raises**:

- `ValueError` - When a suffix component is a symlink or escapes ``trusted_root``.
- `FileNotFoundError` - When an intermediate parent is missing.

#### cli\_user\_path

```python
def cli_user_path(raw: str | Path) -> Path
```

Accept a CLI path argument (any user-chosen location; no cwd jail).

Authoring CLIs take absolute or relative paths without forcing containment
under ``Path.cwd()``. Reject NUL only; callers apply
:func:`resolve_under` for manifest-derived children beneath the opened
plugin/output root.

**Arguments**:

- `raw` - Operator-selected path (absolute, relative, or ``~``-prefixed).
  

**Returns**:

  Normalized absolute path.
  

**Raises**:

- `ValueError` - When the path contains an interior NUL.

#### copy\_tree\_no\_symlinks

```python
def copy_tree_no_symlinks(src: Path, dst: Path) -> None
```

Copy a directory tree, refusing any symlink entries.

Unlike ``shutil.copytree`` (which follows links by default), this never
embeds outside bytes through ``modules/leak -> /outside``.

**Arguments**:

- `src` - Source directory (must not itself be a symlink).
- `dst` - Destination directory to create.
  

**Raises**:

- `ValueError` - When a symlink or unsupported file type is encountered.
- `OSError` - On filesystem failures.

