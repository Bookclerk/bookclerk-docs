**bookclerk_config > extras**

# Module: extras

## Contents

**Structs**

- [`LameConfig`](#lameconfig) - LAME encoder tuning (classic `Lame*` settings).
- [`ReplacementRule`](#replacementrule) - Character replacement rule (`ReplacementCharacters` in classic Settings.json).

**Enums**

- [`FileTimestampMode`](#filetimestampmode) - Which timestamp to apply after acquire (`CreationTime` / `LastWriteTime`).
- [`PathSanitizationMode`](#pathsanitizationmode) - How to sanitize storage path segments when `crate::OutputConfig::replacement_characters`

**Functions**

- [`apply_replacements`](#apply_replacements) - Apply replacement rules to a string (classic filename sanitization).
- [`classic_key_aliases`](#classic_key_aliases) - Classic Settings.json key → dotted config path for `config get` / `config set`.
- [`default_replacement_characters`](#default_replacement_characters) - Alias for [`windows_replacement_characters`](extras.md#bookclerk_configextraswindows_replacement_characters) (classic default name).
- [`key_matches_reconcile_pattern`](#key_matches_reconcile_pattern) - True when `key` matches a reconcile `pattern` built with
- [`parse_replacement_characters`](#parse_replacement_characters) - Parse classic `ReplacementCharacters` JSON.
- [`posix_replacement_characters`](#posix_replacement_characters) - Path-separator sanitization for POSIX local filesystems.
- [`reconciliation_wildcard_rules`](#reconciliation_wildcard_rules) - Replacement rules that turn every known sanitizable character into
- [`resolve_replacement_characters`](#resolve_replacement_characters) - Resolve the character-replacement map used when sanitising storage path segments.
- [`s3_replacement_characters`](#s3_replacement_characters) - Characters AWS recommends avoiding in S3 object keys, plus path separators.
- [`windows_replacement_characters`](#windows_replacement_characters) - NTFS-safe replacement map (classic Libation Windows defaults subset).

**Constants**

- [`RECONCILE_WILDCARD`](#reconcile_wildcard) - Sentinel placed in reconcile path *patterns* where a sanitizable character

---

## bookclerk_config::extras::FileTimestampMode

*Enum*

Which timestamp to apply after acquire (`CreationTime` / `LastWriteTime`).

**Variants:**
- `Now` - Use the wall-clock time at acquire completion (default).
- `Purchased` - Use the account purchase / added-to-library timestamp when known.
- `Published` - Use the publisher release date when known.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &FileTimestampMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> FileTimestampMode`
- **Default**
  - `fn default() -> FileTimestampMode`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::extras::LameConfig

*Struct*

LAME encoder tuning (classic `Lame*` settings).

**Fields:**
- `target: String` - `LameTarget`: `quality`, `bitrate`, `constant`, etc.
- `vbr_quality: u8` - `LameVBRQuality` (0–9, lower = better).
- `bitrate_kbps: u32` - `LameBitrate` in kbps when target is bitrate/constant.
- `mode: String` - `LameMode`: `default`, `mono`, `stereo`.
- `downsample_mono: bool` - `LameDownsampleMono`.
- `constant_bitrate: bool` - `LameConstantBitrate`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LameConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &LameConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_config::extras::PathSanitizationMode

*Enum*

How to sanitize storage path segments when `crate::OutputConfig::replacement_characters`
is empty (explicit rules always win).

**Variants:**
- `Auto` - Pick a profile from the storage backend and host OS.
- `Windows` - NTFS / classic Libation Windows map (`: * ? " < > | / \`).
- `Posix` - Path-separator only (`/` `\`) — suitable for Linux/macOS local disks.
- `S3` - AWS S3 “characters to avoid” plus path separators.
- `None` - Disable platform-specific sanitization; path separators (`/` `\`) are

**Traits:** Eq, Copy

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PathSanitizationMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> PathSanitizationMode`
- **Default**
  - `fn default() -> PathSanitizationMode`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::extras::RECONCILE_WILDCARD

*Constant*: `char`

Sentinel placed in reconcile path *patterns* where a sanitizable character
appeared in metadata. Matches any single character in an on-disk key.



## bookclerk_config::extras::ReplacementRule

*Struct*

Character replacement rule (`ReplacementCharacters` in classic Settings.json).

**Fields:**
- `find: String` - Substring to search for in each path segment / tag value.
- `replace: String` - Replacement text (may be empty to strip `find`).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ReplacementRule) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ReplacementRule`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_config::extras::apply_replacements

*Function*

Apply replacement rules to a string (classic filename sanitization).

```rust
fn apply_replacements(input: &str, rules: &[ReplacementRule]) -> String
```



## bookclerk_config::extras::classic_key_aliases

*Function*

Classic Settings.json key → dotted config path for `config get` / `config set`.

```rust
fn classic_key_aliases() -> std::collections::HashMap<&'static str, &'static str>
```



## bookclerk_config::extras::default_replacement_characters

*Function*

Alias for [`windows_replacement_characters`](extras.md#bookclerk_configextraswindows_replacement_characters) (classic default name).

```rust
fn default_replacement_characters() -> Vec<ReplacementRule>
```



## bookclerk_config::extras::key_matches_reconcile_pattern

*Function*

True when `key` matches a reconcile `pattern` built with
[`reconciliation_wildcard_rules`](extras.md#bookclerk_configextrasreconciliation_wildcard_rules) (wildcard = any single character).

```rust
fn key_matches_reconcile_pattern(pattern: &str, key: &str) -> bool
```



## bookclerk_config::extras::parse_replacement_characters

*Function*

Parse classic `ReplacementCharacters` JSON.

Supports both the classic Libation `Settings.json` shape
(`{ "Replacement": [ { "CharacterToReplace": ":", "ReplacementString": "_" }, … ] }`)
and a flat map shape (`{ ":": "_", … }`). An empty or unrecognised value falls
back to [`windows_replacement_characters`](extras.md#bookclerk_configextraswindows_replacement_characters) (classic migrate parity).

```rust
fn parse_replacement_characters(value: &serde_json::Value) -> Vec<ReplacementRule>
```



## bookclerk_config::extras::posix_replacement_characters

*Function*

Path-separator sanitization for POSIX local filesystems.

Keeps characters such as `:` that are legal on Linux/macOS but reserved on
Windows. `/` and `\` are still replaced so tag values cannot inject path
segments.

```rust
fn posix_replacement_characters() -> Vec<ReplacementRule>
```



## bookclerk_config::extras::reconciliation_wildcard_rules

*Function*

Replacement rules that turn every known sanitizable character into
[`RECONCILE_WILDCARD`](extras.md#bookclerk_configextrasreconcile_wildcard).

Used when searching for existing liberations: creation still follows the
active profile, but reconcile must match files produced under Windows,
POSIX, or S3 rules (or custom `replacement_characters`).

```rust
fn reconciliation_wildcard_rules(explicit: &[ReplacementRule]) -> Vec<ReplacementRule>
```



## bookclerk_config::extras::resolve_replacement_characters

*Function*

Resolve the character-replacement map used when sanitising storage path segments.

Explicit non-empty `replacement_characters` always wins. Otherwise
`path_sanitization` (with `storage_is_s3` for `PathSanitizationMode::Auto`)
selects a profile.

```rust
fn resolve_replacement_characters(explicit: &[ReplacementRule], mode: PathSanitizationMode, storage_is_s3: bool) -> Vec<ReplacementRule>
```



## bookclerk_config::extras::s3_replacement_characters

*Function*

Characters AWS recommends avoiding in S3 object keys, plus path separators.

See [Object key naming guidelines](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html).
Characters that only need URL encoding (e.g. `:`) are left intact.

```rust
fn s3_replacement_characters() -> Vec<ReplacementRule>
```



## bookclerk_config::extras::windows_replacement_characters

*Function*

NTFS-safe replacement map (classic Libation Windows defaults subset).

```rust
fn windows_replacement_characters() -> Vec<ReplacementRule>
```



