**bookclerk_plugin_manifest > workerd_contract**

# Module: workerd_contract

## Contents

**Structs**

- [`AppliedCompatibilityDate`](#appliedcompatibilitydate) - Date written into the workerd config, and a fallback warning when clamped.

**Functions**

- [`apply_author_compatibility_date`](#apply_author_compatibility_date) - Resolves an author `compatibility_date` against this Bookclerk release.
- [`module_load_key`](#module_load_key) - Relative key a modules-directory walk uses for a `[[modules]]` path or name.
- [`validate_author_compatibility_date`](#validate_author_compatibility_date) - Rejects a `compatibility_date` that is not a real `YYYY-MM-DD`.
- [`validate_author_compatibility_flags`](#validate_author_compatibility_flags) - Rejects author flags outside [`PYTHON_COMPATIBILITY_FLAGS`](workerd_contract.md#bookclerk_plugin_manifestworkerd_contractpython_compatibility_flags), a partial
- [`validate_module_declarations`](#validate_module_declarations) - Checks `main_module` and each `[[modules]]` row against the extensions the
- [`workerd_module_is_embedded`](#workerd_module_is_embedded) - True when `path` ends in an extension the modules walk embeds.

**Constants**

- [`PYTHON_COMPATIBILITY_FLAGS`](#python_compatibility_flags) - Author `compatibility_flags` this pin accepts.
- [`WORKERD_PIN_COMPAT_DATE`](#workerd_pin_compat_date) - Newest `compatibility_date` this pin's `workerd` binary can honor.

---

## bookclerk_plugin_manifest::workerd_contract::AppliedCompatibilityDate

*Struct*

Date written into the workerd config, and a fallback warning when clamped.

Bookclerk host surfaces (events, jobs, and later bindings) must follow
`applied`. That is the date workerd actually runs, so a
release gates its own behavior changes on the same calendar the isolate uses.

**Fields:**
- `applied: String` - Author date when it is on or before the pin, otherwise [`WORKERD_PIN_COMPAT_DATE`](workerd_contract.md#bookclerk_plugin_manifestworkerd_contractworkerd_pin_compat_date).
- `warning: Option<String>` - Wrangler-style warning when `applied` was clamped.



## bookclerk_plugin_manifest::workerd_contract::PYTHON_COMPATIBILITY_FLAGS

*Constant*: `&[&str]`

Author `compatibility_flags` this pin accepts.

Both are required together when the guest is Python, and forbidden
otherwise. Host-only flags such as egress `experimental` are not in this
list.



## bookclerk_plugin_manifest::workerd_contract::WORKERD_PIN_COMPAT_DATE

*Constant*: `&str`

Newest `compatibility_date` this pin's `workerd` binary can honor.

Kept equal to `BUNDLED_WORKERD_COMPAT_DATE` / `workerd-pin.json`
`bundled_compat_date`. Older calendar dates are passed through. A newer
date warns and falls back to this value when the isolate is configured.



## bookclerk_plugin_manifest::workerd_contract::apply_author_compatibility_date

*Function*

Resolves an author `compatibility_date` against this Bookclerk release.

workerd enables each compatibility flag whose default-on date is on or
before the date it is given, and it refuses to start when that date is
newer than `supported-compatibility-date.txt` in the binary. Wrangler
works around that refusal by warning and starting at the newest date the
installed runtime supports. This function does the same with
[`WORKERD_PIN_COMPAT_DATE`](workerd_contract.md#bookclerk_plugin_manifestworkerd_contractworkerd_pin_compat_date).

# Arguments

* `date` - Author `workerd.compatibility_date` string, untrimmed.

# Errors

Returns `Error::Message` when the string is not a real calendar date.

```rust
fn apply_author_compatibility_date(date: &str) -> crate::error::Result<AppliedCompatibilityDate>
```



## bookclerk_plugin_manifest::workerd_contract::module_load_key

*Function*

Relative key a modules-directory walk uses for a `[[modules]]` path or name.

Drops `.` segments, then strips a leading `modules_dir/` prefix, so
`./modules/index.js`, `modules/./index.js`, and `index.js` all match a walk
of `modules_dir = "modules"`. `..` segments are left in place.

# Arguments

* `modules_dir` - `[workerd].modules_dir`.
* `raw` - Author `path` or `name`.

# Returns

Slash-separated relative key, or an empty string when `raw` is empty.

```rust
fn module_load_key(modules_dir: &str, raw: &str) -> String
```



## bookclerk_plugin_manifest::workerd_contract::validate_author_compatibility_date

*Function*

Rejects a `compatibility_date` that is not a real `YYYY-MM-DD`.

A date newer than [`WORKERD_PIN_COMPAT_DATE`](workerd_contract.md#bookclerk_plugin_manifestworkerd_contractworkerd_pin_compat_date) is still valid. Load falls
back to the pin and warns; see [`apply_author_compatibility_date`](workerd_contract.md#bookclerk_plugin_manifestworkerd_contractapply_author_compatibility_date).

# Arguments

* `date` - Author `workerd.compatibility_date` string, untrimmed.

# Errors

Returns `Error::Message` when the string is not a real calendar date.

```rust
fn validate_author_compatibility_date(date: &str) -> crate::error::Result<()>
```



## bookclerk_plugin_manifest::workerd_contract::validate_author_compatibility_flags

*Function*

Rejects author flags outside [`PYTHON_COMPATIBILITY_FLAGS`](workerd_contract.md#bookclerk_plugin_manifestworkerd_contractpython_compatibility_flags), a partial
Python pair, Python flags without a Python module, or flags missing when
`python` is true.

# Arguments

* `flags` - `workerd.compatibility_flags` in author order.
* `python` - True when the manifest declares a Python module. A `.py`
  file found only by the modules walk is not enough; callers reject that
  separately and must not pass it as `python`. The flag names themselves
  are not evidence.

# Errors

Returns `Error::Message` for the first violated rule. Does not insert
missing flags.

```rust
fn validate_author_compatibility_flags(flags: &[String], python: bool) -> crate::error::Result<()>
```



## bookclerk_plugin_manifest::workerd_contract::validate_module_declarations

*Function*

Checks `main_module` and each `[[modules]]` row against the extensions the
directory walk embeds.

A type that disagrees with the extension is rejected. `.txt`, `.md`,
`.ts`, and any other extension the walk skips fail with "not implemented
yet" (expected authoring parity, not a permanent ban). This does not look
at the filesystem; callers that have a modules directory also require each
row to be in the walk's load set.

# Arguments

* `main_module` - `[workerd].main_module`.
* `modules` - `[[modules]]` rows.

# Errors

Returns `Error::Message` for the first row or main file that fails.

```rust
fn validate_module_declarations(main_module: &str, modules: &[crate::types::ModuleSpec]) -> crate::error::Result<()>
```



## bookclerk_plugin_manifest::workerd_contract::workerd_module_is_embedded

*Function*

True when `path` ends in an extension the modules walk embeds.

# Arguments

* `path` - Module path or filename.

```rust
fn workerd_module_is_embedded(path: &str) -> bool
```



