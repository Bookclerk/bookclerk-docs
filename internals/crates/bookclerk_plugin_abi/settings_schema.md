# bookclerk_plugin_abi::settings_schema

Operator-facing plugin settings schema (`[[settings]]` in `plugin.toml`).

A plugin declares the keys an operator may set on one of its instances,
their types, and their defaults. The host validates instance documents
against this schema and renders it in Settings; it hard-codes nothing per
plugin. The same struct is the `plugin.toml` TOML shape and the JSON body
pinned in the library database at enrollment.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SettingValueSpec`](#settingvaluespec) | struct | One selectable value of an `SettingKind::Enum` setting. |
| [`SettingSpec`](#settingspec) | struct | One `[[settings]]` entry. |
| [`SettingsSchema`](#settingsschema) | struct | The ordered `[[settings]]` list of one plugin. |
| [`EffectiveConfig`](#effectiveconfig) | struct | Result of `SettingsSchema::effective_config`. |
| [`SettingKind`](#settingkind) | enum | Value type of one setting. |
| [`is_valid_setting_key`](#is_valid_setting_key) | function | Grammar for setting keys and secret names: `^a-z{0,63}$`. |
| [`SETTINGS_SCHEMA_VERSION`](#settings_schema_version) | constant | Schema revision of `SettingsSchema` as pinned in the database. |
| [`MAX_SETTINGS`](#max_settings) | constant | Maximum `[[settings]]` entries one manifest may declare. |
| [`MAX_SETTING_VALUES`](#max_setting_values) | constant | Maximum enum values one setting may declare. |
| [`MAX_SETTING_STRING_BYTES`](#max_setting_string_bytes) | constant | Maximum UTF-8 size of one string setting value. |

## Structs

### SettingValueSpec

```rust
pub struct SettingValueSpec {
    pub id: String,
    pub label: String,
}
```

One selectable value of an [`SettingKind::Enum`](#settingkind) setting.

**Fields**

- `id`: Stored value.
- `label`: Operator-facing label; defaults to `id`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SettingSpec

```rust
pub struct SettingSpec {
    pub key: String,
    pub label: String,
    pub kind: SettingKind,
    pub values: Vec<SettingValueSpec>,
    pub default: Option<Value>,
    pub required: bool,
    pub description: Option<String>,
}
```

One `[[settings]]` entry.

**Fields**

- `key`: Setting key (`^[a-z][a-z0-9_]{0,63}$`), unique within the plugin.
- `label`: Operator-facing label; defaults to `key`.
- `kind`: Value type.
- `values`: Allowed values for [`SettingKind::Enum`](#settingkind); empty otherwise.
- `default`: Default applied when the instance document omits the key. Not allowed
  for secrets.
- `required`: When true, a spawn without a value (document or default) fails.
- `description`: Optional help text.

#### SettingSpec::label_or_key

```rust
pub fn label_or_key(&self) -> &str
```

Label for UI, falling back to the key.

#### SettingSpec::is_secret

```rust
pub fn is_secret(&self) -> bool
```

True for [`SettingKind::Secret`](#settingkind).

#### SettingSpec::check_value

```rust
pub fn check_value(&self, value: &Value) -> Result<(), String>
```

Checks `value` against this entry's kind.

##### Errors

Returns a message naming the key when the value does not fit.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SettingsSchema

```rust
pub struct SettingsSchema {
    pub settings: Vec<SettingSpec>,
}
```

The ordered `[[settings]]` list of one plugin.

**Fields**

- `settings`: Entries in declaration order.

#### SettingsSchema::new

```rust
pub fn new(settings: Vec<SettingSpec>) -> Self
```

Wraps a declared list without validating it.

#### SettingsSchema::validate

```rust
pub fn validate(&self, config_binding: bool, secrets_binding: bool) -> Result<(), String>
```

Validates the declaration itself.

`config_binding` / `secrets_binding` say whether the manifest declares
`[vars]` / `[secrets]`; non-secret entries need the former and secret
entries the latter, because that is how values reach the guest.

##### Errors

Returns a message describing the first failed rule.

#### SettingsSchema::get

```rust
pub fn get(&self, key: &str) -> Option<&SettingSpec>
```

Entry for `key`.

#### SettingsSchema::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no entries are declared.

#### SettingsSchema::config_entries

```rust
pub fn config_entries(&self) -> impl Iterator<Item = &SettingSpec>
```

Non-secret entries, in order.

#### SettingsSchema::secret_entries

```rust
pub fn secret_entries(&self) -> impl Iterator<Item = &SettingSpec>
```

Secret entries, in order.

#### SettingsSchema::defaults

```rust
pub fn defaults(&self) -> BTreeMap<String, Value>
```

Defaults for non-secret entries that declare one.

#### SettingsSchema::config_options

```rust
pub fn config_options(&self) -> Vec<ConfigOption>
```

`describe().configOptions` projection: every non-secret entry, with
enum values listed and other kinds as free-form.

#### SettingsSchema::check_setting

```rust
pub fn check_setting(&self, key: &str, value: &Value) -> Result<(), String>
```

Checks one document setting against the schema.

##### Errors

Unknown keys, secrets given as plain values, and type mismatches are
rejected with a message naming the key.

#### SettingsSchema::check_secret_key

```rust
pub fn check_secret_key(&self, key: &str) -> Result<(), String>
```

Checks one secret ref key against the schema.

##### Errors

Returns a message when `key` is not a declared secret.

#### SettingsSchema::required_secret_keys

```rust
pub fn required_secret_keys(&self) -> impl Iterator<Item = &str>
```

Keys of `secret` entries marked `required`.

#### SettingsSchema::effective_config

```rust
pub fn effective_config(&self, settings: &BTreeMap<String, Value>) -> EffectiveConfig
```

Merges `settings` over the schema defaults and reports missing
required keys.

Keys the schema does not declare are dropped and returned in
`dropped`; declared keys whose value no longer fits the entry's kind
(a type or enum change after an upgrade) are left out of `config` and
returned in `invalid`, so the default applies instead. A required key
with neither a usable value nor a default is `missing`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EffectiveConfig

```rust
pub struct EffectiveConfig {
    pub config: BTreeMap<String, Value>,
    pub dropped: Vec<String>,
    pub invalid: Vec<String>,
    pub missing: Vec<String>,
}
```

Result of [`SettingsSchema::effective_config`](#settingsschemaeffective_config).

**Fields**

- `config`: Values delivered as `CONFIG`.
- `dropped`: Document keys the schema does not declare.
- `invalid`: Declared keys whose stored value does not fit the entry's kind.
- `missing`: Required keys with neither a usable value nor a default.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

## Enums

### SettingKind

```rust
pub enum SettingKind {
    String,
    Bool,
    Integer,
    Enum,
    Secret,
}
```

Value type of one setting.

**Variants**

- `String`: Free-form UTF-8 string (at most [`MAX_SETTING_STRING_BYTES`](#max_setting_string_bytes)).
- `Bool`: Boolean.
- `Integer`: Signed 64-bit integer.
- `Enum`: One of the declared [`SettingSpec::values`](#settingspec) ids.
- `Secret`: Sealed secret: delivered in `SECRETS`, never stored in a document.

#### SettingKind::as_str

```rust
pub fn as_str(self) -> &'static str
```

Wire / UI name (`string`, `bool`, `integer`, `enum`, `secret`).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

## Functions

### is_valid_setting_key

```rust
pub fn is_valid_setting_key(key: &str) -> bool
```

Grammar for setting keys and secret names: `^[a-z][a-z0-9_]{0,63}$`.

## Constants

### SETTINGS_SCHEMA_VERSION

```rust
pub const SETTINGS_SCHEMA_VERSION: u32 = 1;
```

Schema revision of [`SettingsSchema`](#settingsschema) as pinned in the database.

### MAX_SETTINGS

```rust
pub const MAX_SETTINGS: usize = 32;
```

Maximum `[[settings]]` entries one manifest may declare.

### MAX_SETTING_VALUES

```rust
pub const MAX_SETTING_VALUES: usize = 64;
```

Maximum enum values one setting may declare.

### MAX_SETTING_STRING_BYTES

```rust
pub const MAX_SETTING_STRING_BYTES: usize = 1024;
```

Maximum UTF-8 size of one string setting value.
