**bookclerk_plugin_abi > settings_schema**

# Module: settings_schema

## Contents

**Structs**

- [`EffectiveConfig`](#effectiveconfig) - Result of `SettingsSchema::effective_config`.
- [`SettingSpec`](#settingspec) - One `[[settings]]` entry.
- [`SettingValueSpec`](#settingvaluespec) - One selectable value of an `SettingKind::Enum` setting.
- [`SettingsSchema`](#settingsschema) - The ordered `[[settings]]` list of one plugin.

**Enums**

- [`SettingKind`](#settingkind) - Value type of one setting.

**Functions**

- [`is_valid_setting_key`](#is_valid_setting_key) - Grammar for setting keys and secret names: `^[a-z][a-z0-9_]{0,63}$`.

**Constants**

- [`MAX_SETTINGS`](#max_settings) - Maximum `[[settings]]` entries one manifest may declare.
- [`MAX_SETTING_STRING_BYTES`](#max_setting_string_bytes) - Maximum UTF-8 size of one string setting value.
- [`MAX_SETTING_VALUES`](#max_setting_values) - Maximum enum values one setting may declare.
- [`SETTINGS_SCHEMA_VERSION`](#settings_schema_version) - Schema revision of [`SettingsSchema`](settings_schema.md#bookclerk_plugin_abisettings_schemasettingsschema) as pinned in the database.

---

## bookclerk_plugin_abi::settings_schema::EffectiveConfig

*Struct*

Result of `SettingsSchema::effective_config`.

**Fields:**
- `config: std::collections::BTreeMap<String, serde_json::Value>` - Values delivered as `CONFIG`.
- `dropped: Vec<String>` - Document keys the schema does not declare.
- `invalid: Vec<String>` - Declared keys whose stored value does not fit the entry's kind.
- `missing: Vec<String>` - Required keys with neither a usable value nor a default.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &EffectiveConfig) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> EffectiveConfig`
- **Clone**
  - `fn clone(self: &Self) -> EffectiveConfig`



## bookclerk_plugin_abi::settings_schema::MAX_SETTINGS

*Constant*: `usize`

Maximum `[[settings]]` entries one manifest may declare.



## bookclerk_plugin_abi::settings_schema::MAX_SETTING_STRING_BYTES

*Constant*: `usize`

Maximum UTF-8 size of one string setting value.



## bookclerk_plugin_abi::settings_schema::MAX_SETTING_VALUES

*Constant*: `usize`

Maximum enum values one setting may declare.



## bookclerk_plugin_abi::settings_schema::SETTINGS_SCHEMA_VERSION

*Constant*: `u32`

Schema revision of [`SettingsSchema`](settings_schema.md#bookclerk_plugin_abisettings_schemasettingsschema) as pinned in the database.



## bookclerk_plugin_abi::settings_schema::SettingKind

*Enum*

Value type of one setting.

**Variants:**
- `String` - Free-form UTF-8 string (at most [`MAX_SETTING_STRING_BYTES`](settings_schema.md#bookclerk_plugin_abisettings_schemamax_setting_string_bytes)).
- `Bool` - Boolean.
- `Integer` - Signed 64-bit integer.
- `Enum` - One of the declared `SettingSpec::values` ids.
- `Secret` - Sealed secret: delivered in `SECRETS`, never stored in a document.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Wire / UI name (`string`, `bool`, `integer`, `enum`, `secret`).

**Traits:** Copy, Eq

**Trait Implementations:**

- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Clone**
  - `fn clone(self: &Self) -> SettingKind`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SettingKind) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::settings_schema::SettingSpec

*Struct*

One `[[settings]]` entry.

**Fields:**
- `key: String` - Setting key (`^[a-z][a-z0-9_]{0,63}$`), unique within the plugin.
- `label: String` - Operator-facing label; defaults to `key`.
- `kind: SettingKind` - Value type.
- `values: Vec<SettingValueSpec>` - Allowed values for `SettingKind::Enum`; empty otherwise.
- `default: Option<serde_json::Value>` - Default applied when the instance document omits the key. Not allowed
- `required: bool` - When true, a spawn without a value (document or default) fails.
- `description: Option<String>` - Optional help text.

**Methods:**

- `fn label_or_key(self: &Self) -> &str` - Label for UI, falling back to the key.
- `fn is_secret(self: &Self) -> bool` - True for `SettingKind::Secret`.
- `fn check_value(self: &Self, value: &Value) -> Result<(), String>` - Checks `value` against this entry's kind.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SettingSpec`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SettingSpec) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::settings_schema::SettingValueSpec

*Struct*

One selectable value of an `SettingKind::Enum` setting.

**Fields:**
- `id: String` - Stored value.
- `label: String` - Operator-facing label; defaults to `id`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SettingValueSpec) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SettingValueSpec`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::settings_schema::SettingsSchema

*Struct*

The ordered `[[settings]]` list of one plugin.

**Fields:**
- `settings: Vec<SettingSpec>` - Entries in declaration order.

**Methods:**

- `fn new(settings: Vec<SettingSpec>) -> Self` - Wraps a declared list without validating it.
- `fn validate(self: &Self, config_binding: bool, secrets_binding: bool) -> Result<(), String>` - Validates the declaration itself.
- `fn get(self: &Self, key: &str) -> Option<&SettingSpec>` - Entry for `key`.
- `fn is_empty(self: &Self) -> bool` - True when no entries are declared.
- `fn config_entries(self: &Self) -> impl Trait` - Non-secret entries, in order.
- `fn secret_entries(self: &Self) -> impl Trait` - Secret entries, in order.
- `fn defaults(self: &Self) -> BTreeMap<String, Value>` - Defaults for non-secret entries that declare one.
- `fn config_options(self: &Self) -> Vec<ConfigOption>` - `describe().configOptions` projection: every non-secret entry, with
- `fn check_setting(self: &Self, key: &str, value: &Value) -> Result<(), String>` - Checks one document setting against the schema.
- `fn check_secret_key(self: &Self, key: &str) -> Result<(), String>` - Checks one secret ref key against the schema.
- `fn required_secret_keys(self: &Self) -> impl Trait` - Keys of `secret` entries marked `required`.
- `fn effective_config(self: &Self, settings: &BTreeMap<String, Value>) -> EffectiveConfig` - Merges `settings` over the schema defaults and reports missing

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> SettingsSchema`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SettingsSchema`
- **PartialEq**
  - `fn eq(self: &Self, other: &SettingsSchema) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::settings_schema::is_valid_setting_key

*Function*

Grammar for setting keys and secret names: `^[a-z][a-z0-9_]{0,63}$`.

```rust
fn is_valid_setting_key(key: &str) -> bool
```



