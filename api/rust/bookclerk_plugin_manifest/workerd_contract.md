# bookclerk_plugin_manifest::workerd_contract

Author compatibility date, flag allowlist, and module-extension rules.

These match the pinned local `workerd` binary. [`crate::PluginManifest::validate`](types.md#pluginmanifestvalidate)
and the TypeScript / Python author checkers enforce the same calendar, flag,
and module rules. A `compatibility_date` newer than the pin stays legal:
check and load warn, then run at the pin, because workerd refuses a date
past the one baked into the binary. `[[kv_namespaces]]` and `[queues]` are
host bindings served at `open`, so nothing here constrains them.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AppliedCompatibilityDate`](#appliedcompatibilitydate) | struct | Date written into the workerd config, and a fallback warning when clamped. |
| [`validate_author_compatibility_date`](#validate_author_compatibility_date) | function | Rejects a `compatibility_date` that is not a real `YYYY-MM-DD`. |
| [`apply_author_compatibility_date`](#apply_author_compatibility_date) | function | Resolves an author `compatibility_date` against this Bookclerk release. |
| [`validate_author_compatibility_flags`](#validate_author_compatibility_flags) | function | Rejects author flags outside `PYTHON_COMPATIBILITY_FLAGS`, a partial Python pair, Python flags without a Python module, or flags missing when `python` is true. |
| [`validate_module_declarations`](#validate_module_declarations) | function | Checks `main_module` and each `[[modules]]` row against the extensions the directory walk embeds. |
| [`module_load_key`](#module_load_key) | function | Relative key a modules-directory walk uses for a `[[modules]]` path or name. |
| [`workerd_module_is_embedded`](#workerd_module_is_embedded) | function | True when `path` ends in an extension the modules walk embeds. |
| [`WORKERD_PIN_COMPAT_DATE`](#workerd_pin_compat_date) | constant | Newest `compatibility_date` this pin's `workerd` binary can honor. |
| [`PYTHON_COMPATIBILITY_FLAGS`](#python_compatibility_flags) | constant | Author `compatibility_flags` this pin accepts. |

## Structs

### AppliedCompatibilityDate

```rust
pub struct AppliedCompatibilityDate {
    pub applied: String,
    pub warning: Option<String>,
}
```

Date written into the workerd config, and a fallback warning when clamped.

Bookclerk host surfaces (events, jobs, and later bindings) must follow
[`applied`](#appliedcompatibilitydate). That is the date workerd actually runs, so a
release gates its own behavior changes on the same calendar the isolate uses.

**Fields**

- `applied`: Author date when it is on or before the pin, otherwise [`WORKERD_PIN_COMPAT_DATE`](#workerd_pin_compat_date).
- `warning`: Wrangler-style warning when [`applied`](#appliedcompatibilitydate) was clamped.
  
  `None` when the author date is equal to or older than the pin.

## Functions

### validate_author_compatibility_date

```rust
pub fn validate_author_compatibility_date(date: &str) -> Result<()>
```

Rejects a `compatibility_date` that is not a real `YYYY-MM-DD`.

A date newer than [`WORKERD_PIN_COMPAT_DATE`](#workerd_pin_compat_date) is still valid. Load falls
back to the pin and warns; see [`apply_author_compatibility_date`](#apply_author_compatibility_date).

#### Arguments

* `date` - Author `workerd.compatibility_date` string, untrimmed.

#### Errors

Returns [`Error::Message`](error.md#error) when the string is not a real calendar date.

### apply_author_compatibility_date

```rust
pub fn apply_author_compatibility_date(date: &str) -> Result<AppliedCompatibilityDate>
```

Resolves an author `compatibility_date` against this Bookclerk release.

workerd enables each compatibility flag whose default-on date is on or
before the date it is given, and it refuses to start when that date is
newer than `supported-compatibility-date.txt` in the binary. Wrangler
works around that refusal by warning and starting at the newest date the
installed runtime supports. This function does the same with
[`WORKERD_PIN_COMPAT_DATE`](#workerd_pin_compat_date).

#### Arguments

* `date` - Author `workerd.compatibility_date` string, untrimmed.

#### Errors

Returns [`Error::Message`](error.md#error) when the string is not a real calendar date.

### validate_author_compatibility_flags

```rust
pub fn validate_author_compatibility_flags(flags: &[String], python: bool) -> Result<()>
```

Rejects author flags outside [`PYTHON_COMPATIBILITY_FLAGS`](#python_compatibility_flags), a partial
Python pair, Python flags without a Python module, or flags missing when
`python` is true.

#### Arguments

* `flags` - `workerd.compatibility_flags` in author order.
* `python` - True when the manifest declares a Python module. A `.py`
  file found only by the modules walk is not enough; callers reject that
  separately and must not pass it as `python`. The flag names themselves
  are not evidence.

#### Errors

Returns [`Error::Message`](error.md#error) for the first violated rule. Does not insert
missing flags.

### validate_module_declarations

```rust
pub fn validate_module_declarations(
    main_module: &str,
    modules: &[ModuleSpec],
) -> Result<()>
```

Checks `main_module` and each `[[modules]]` row against the extensions the
directory walk embeds.

A type that disagrees with the extension is rejected. `.txt`, `.md`,
`.ts`, and any other extension the walk skips fail with "not implemented
yet" (expected authoring parity, not a permanent ban). This does not look
at the filesystem; callers that have a modules directory also require each
row to be in the walk's load set.

#### Arguments

* `main_module` - `[workerd].main_module`.
* `modules` - `[[modules]]` rows.

#### Errors

Returns [`Error::Message`](error.md#error) for the first row or main file that fails.

### module_load_key

```rust
pub fn module_load_key(modules_dir: &str, raw: &str) -> String
```

Relative key a modules-directory walk uses for a `[[modules]]` path or name.

Drops `.` segments, then strips a leading `modules_dir/` prefix, so
`./modules/index.js`, `modules/./index.js`, and `index.js` all match a walk
of `modules_dir = "modules"`. `..` segments are left in place.

#### Arguments

* `modules_dir` - `[workerd].modules_dir`.
* `raw` - Author `path` or `name`.

#### Returns

Slash-separated relative key, or an empty string when `raw` is empty.

### workerd_module_is_embedded

```rust
pub fn workerd_module_is_embedded(path: &str) -> bool
```

True when `path` ends in an extension the modules walk embeds.

#### Arguments

* `path` - Module path or filename.

## Constants

### WORKERD_PIN_COMPAT_DATE

```rust
pub const WORKERD_PIN_COMPAT_DATE: &str = "2026-10-21";
```

Newest `compatibility_date` this pin's `workerd` binary can honor.

Kept equal to `BUNDLED_WORKERD_COMPAT_DATE` / `workerd-pin.json`
`bundled_compat_date`. Older calendar dates are passed through. A newer
date warns and falls back to this value when the isolate is configured.

### PYTHON_COMPATIBILITY_FLAGS

```rust
pub const PYTHON_COMPATIBILITY_FLAGS: &[&str] = _;
```

Author `compatibility_flags` this pin accepts.

Both are required together when the guest is Python, and forbidden
otherwise. Host-only flags such as egress `experimental` are not in this
list.
