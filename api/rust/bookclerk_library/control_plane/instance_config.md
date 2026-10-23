**bookclerk_library > control_plane > instance_config**

# Module: control_plane::instance_config

## Contents

**Structs**

- [`InstanceBindingGrant`](#instancebindinggrant) - Whether the plugin-instance config grant includes `config` or `secrets`.
- [`InstanceConfigDocument`](#instanceconfigdocument) - A parsed instance document plus its revision.
- [`InstanceSecretRefV1`](#instancesecretrefv1) - A name in `encrypted_secrets`. The ciphertext is not part of the document.
- [`PluginInstanceConfigV1`](#plugininstanceconfigv1) - Schema-1 plugin instance configuration.
- [`PrunedInstanceConfig`](#prunedinstanceconfig) - One instance document rewritten to fit the pinned schema.
- [`ResolvedInstanceBindings`](#resolvedinstancebindings) - `CONFIG` and `SECRETS` payloads for one instance.

**Enums**

- [`InstanceConfigReplace`](#instanceconfigreplace) - Outcome of [`replace_instance_config`](instance_config.md#bookclerk_librarycontrol_planeinstance_configreplace_instance_config).
- [`SettingValue`](#settingvalue) - One scalar setting. Nested objects and arrays are not representable.

**Functions**

- [`import_instance_config_if_absent`](#import_instance_config_if_absent) - Inserts the document when it is absent.
- [`load_instance_config`](#load_instance_config) - Loads and parses the instance config document.
- [`parse_setting_text`](#parse_setting_text) - Parses an operator-typed value for `key` according to the schema.
- [`reconcile_instance_configs_to_schema`](#reconcile_instance_configs_to_schema) - Rewrites every instance document of `plugin_key` to what `schema` still
- [`replace_instance_config`](#replace_instance_config) - Replaces the instance document when `expected_revision` is current.
- [`resolve_instance_bindings`](#resolve_instance_bindings) - Resolves `CONFIG` and `SECRETS` for `instance_id` against `schema`.
- [`seal_instance_secret`](#seal_instance_secret) - Seals one instance secret with the cluster DEK.
- [`set_secret_ref`](#set_secret_ref) - Adds a secret ref for `key` whose sealed row is named `key`, replacing an
- [`set_setting`](#set_setting) - Sets `key` to `value` in `settings`, or removes it when `value` is `None`.

**Constants**

- [`MAX_INSTANCE_SECRET_REFS`](#max_instance_secret_refs) - Maximum secret refs in one schema-1 document.
- [`MAX_INSTANCE_SETTINGS`](#max_instance_settings) - Maximum settings in one schema-1 document.
- [`MAX_INSTANCE_SETTING_BYTES`](#max_instance_setting_bytes) - Maximum UTF-8 size of one string setting.
- [`PLUGIN_INSTANCE_CONFIG_SCHEMA_VERSION`](#plugin_instance_config_schema_version) - Domain schema of [`PluginInstanceConfigV1`](instance_config.md#bookclerk_librarycontrol_planeinstance_configplugininstanceconfigv1).

---

## bookclerk_library::control_plane::instance_config::InstanceBindingGrant

*Struct*

Whether the plugin-instance config grant includes `config` or `secrets`.

Resolved from the plugin key's grant before this function runs.

**Fields:**
- `config: bool` - Grant includes the `config` binding.
- `secrets: bool` - Grant includes the `secrets` binding.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> InstanceBindingGrant`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &InstanceBindingGrant) -> bool`



## bookclerk_library::control_plane::instance_config::InstanceConfigDocument

*Struct*

A parsed instance document plus its revision.

**Fields:**
- `body: PluginInstanceConfigV1` - Parsed body.
- `revision: i64` - Compare-and-swap revision.
- `schema_version: i64` - Domain schema version. Schema 1 is the only version this binary applies.
- `instance_id: String` - Instance id (`scope_id`).

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &InstanceConfigDocument) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> InstanceConfigDocument`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::instance_config::InstanceConfigReplace

*Enum*

Outcome of [`replace_instance_config`](instance_config.md#bookclerk_librarycontrol_planeinstance_configreplace_instance_config).

**Variants:**
- `Applied(InstanceConfigDocument)` - The new document is committed.
- `Conflict{ current_revision: i64 }` - `expected_revision` was not current. No audit row was written.
- `Replayed{ revision: i64 }` - This operation id already committed at `revision`.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> InstanceConfigReplace`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &InstanceConfigReplace) -> bool`



## bookclerk_library::control_plane::instance_config::InstanceSecretRefV1

*Struct*

A name in `encrypted_secrets`. The ciphertext is not part of the document.

**Fields:**
- `key: String` - Key in the `SECRETS` object delivered to the guest.
- `name: String` - `encrypted_secrets.name` for this instance.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> InstanceSecretRefV1`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &InstanceSecretRefV1) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::control_plane::instance_config::MAX_INSTANCE_SECRET_REFS

*Constant*: `usize`

Maximum secret refs in one schema-1 document.



## bookclerk_library::control_plane::instance_config::MAX_INSTANCE_SETTINGS

*Constant*: `usize`

Maximum settings in one schema-1 document.



## bookclerk_library::control_plane::instance_config::MAX_INSTANCE_SETTING_BYTES

*Constant*: `usize`

Maximum UTF-8 size of one string setting.



## bookclerk_library::control_plane::instance_config::PLUGIN_INSTANCE_CONFIG_SCHEMA_VERSION

*Constant*: `i64`

Domain schema of [`PluginInstanceConfigV1`](instance_config.md#bookclerk_librarycontrol_planeinstance_configplugininstanceconfigv1).



## bookclerk_library::control_plane::instance_config::PluginInstanceConfigV1

*Struct*

Schema-1 plugin instance configuration.

**Fields:**
- `settings: std::collections::BTreeMap<String, SettingValue>` - Operator settings. Keys match `^[a-z][a-z0-9_]{0,63}$`.
- `secret_refs: Vec<InstanceSecretRefV1>` - Secret names. Values stay in `encrypted_secrets`.

**Methods:**

- `fn empty() -> Self` - Empty settings and no secret refs.
- `fn validate(self: &Self, schema: &SettingsSchema) -> Result<()>` - Rejects documents whose settings or secret refs the plugin does not
- `fn validate_shape(self: &Self) -> Result<()>` - Rejects documents this binary will not store regardless of schema:
- `fn to_json(self: &Self) -> Result<String>` - Canonical JSON body stored in `document_json`.
- `fn settings_json(self: &Self) -> Value` - Settings object delivered as the `CONFIG` payload.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginInstanceConfigV1) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginInstanceConfigV1`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::instance_config::PrunedInstanceConfig

*Struct*

One instance document rewritten to fit the pinned schema.

**Fields:**
- `instance_id: String` - Instance whose document changed.
- `removed_keys: Vec<String>` - Keys dropped from the document: settings the schema no longer
- `revision: i64` - Revision after the rewrite.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PrunedInstanceConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PrunedInstanceConfig) -> bool`



## bookclerk_library::control_plane::instance_config::ResolvedInstanceBindings

*Struct*

`CONFIG` and `SECRETS` payloads for one instance.

**Fields:**
- `config: bookclerk_plugin_abi::ExtensibleConfig` - Schema defaults merged with the document, as `application/json`.
- `secrets: bookclerk_plugin_abi::ExtensibleConfig` - Granted secret values as `application/json`, or an empty payload.
- `config_revision: i64` - Document revision applied by this resolution.
- `dropped_keys: Vec<String>` - Document keys left out of `config`: ones the schema no longer

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ResolvedInstanceBindings) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ResolvedInstanceBindings`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::instance_config::SettingValue

*Enum*

One scalar setting. Nested objects and arrays are not representable.

**Variants:**
- `Bool(bool)` - JSON boolean.
- `Number(serde_json::Number)` - Finite JSON number.
- `String(String)` - JSON string, at most 1024 bytes.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SettingValue) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SettingValue`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_library::control_plane::instance_config::import_instance_config_if_absent

*Function*

Inserts the document when it is absent.

An existing row is returned and the seed is not validated again.

# Errors

Returns an error when the actor may not import, the body is invalid, or the write fails.

```rust
fn import_instance_config_if_absent(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, instance_id: &super::instances::PluginInstanceId, schema: &bookclerk_plugin_abi::SettingsSchema, body: &PluginInstanceConfigV1, operation_id: &str) -> crate::error::Result<InstanceConfigDocument>
```



## bookclerk_library::control_plane::instance_config::load_instance_config

*Function*

Loads and parses the instance config document.

An unknown `schema_version` fails the read. The stored JSON is not rewritten.

# Errors

Returns an error when the row is missing, the schema is unknown, or the body is invalid.

```rust
fn load_instance_config(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId) -> crate::error::Result<InstanceConfigDocument>
```



## bookclerk_library::control_plane::instance_config::parse_setting_text

*Function*

Parses an operator-typed value for `key` according to the schema.

Enum and string settings take the text as given; `bool` accepts
`true`/`false`; `integer` a signed decimal. Secrets are rejected here:
seal them with [`seal_instance_secret`](instance_config.md#bookclerk_librarycontrol_planeinstance_configseal_instance_secret) and add a ref.

# Errors

Returns an error when `key` is unknown or `text` does not fit its kind.

```rust
fn parse_setting_text(schema: &bookclerk_plugin_abi::SettingsSchema, key: &str, text: &str) -> crate::error::Result<SettingValue>
```



## bookclerk_library::control_plane::instance_config::reconcile_instance_configs_to_schema

*Function*

Rewrites every instance document of `plugin_key` to what `schema` still
accepts: settings whose key is gone, became a secret, or whose value no
longer fits (a type or enum change), and secret refs whose key is gone or
became plain, are dropped.

Enrollment runs this on every pass, so a document that lost a race or
was written under an earlier schema is repaired on the next restart
rather than staying invalid. A document that already fits is not
rewritten; defaults for new keys are not written (`CONFIG` resolution
applies them at spawn). One instance's failure (a compare-and-swap
conflict because another node wrote the document meanwhile, or an
unreadable row) is logged and skipped so the others are still handled.

# Errors

Returns an error when the instance list cannot be read.

```rust
fn reconcile_instance_configs_to_schema(store: &crate::store::LibraryStore, plugin_key: &str, schema: &bookclerk_plugin_abi::SettingsSchema) -> crate::error::Result<Vec<PrunedInstanceConfig>>
```



## bookclerk_library::control_plane::instance_config::replace_instance_config

*Function*

Replaces the instance document when `expected_revision` is current.

# Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

```rust
fn replace_instance_config(store: &crate::store::LibraryStore, actor: &super::documents::ConfigActor, instance_id: &super::instances::PluginInstanceId, schema: &bookclerk_plugin_abi::SettingsSchema, expected_revision: i64, body: &PluginInstanceConfigV1, operation_id: &str) -> crate::error::Result<InstanceConfigReplace>
```



## bookclerk_library::control_plane::instance_config::resolve_instance_bindings

*Function*

Resolves `CONFIG` and `SECRETS` for `instance_id` against `schema`.

`CONFIG` is the schema's defaults overlaid with the document's settings.
Keys the schema does not declare, and values that no longer fit their
entry, are dropped and reported; a required key with neither a usable
value nor a default fails, as does a required secret with no ref. A
missing document, a dangling secret ref, a secret that fails to unseal, a
missing `config` grant for a plugin that declares settings, or secret refs
without a `secrets` grant fails resolution. An empty ref list with no
`secrets` grant yields the empty secrets payload.

# Errors

Returns an error when the document or a secret ref cannot be resolved.

```rust
fn resolve_instance_bindings(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId, grant: &InstanceBindingGrant, schema: &bookclerk_plugin_abi::SettingsSchema) -> crate::error::Result<ResolvedInstanceBindings>
```



## bookclerk_library::control_plane::instance_config::seal_instance_secret

*Function*

Seals one instance secret with the cluster DEK.

The plaintext is not written to `configuration_documents` or `configuration_audit`.

# Errors

Returns an error when the value is empty or the seal fails.

```rust
fn seal_instance_secret(store: &crate::store::LibraryStore, instance_id: &super::instances::PluginInstanceId, plugin_key: &str, name: &str, plaintext: &str) -> crate::error::Result<()>
```



## bookclerk_library::control_plane::instance_config::set_secret_ref

*Function*

Adds a secret ref for `key` whose sealed row is named `key`, replacing an
existing ref for the same key.

```rust
fn set_secret_ref(body: & mut PluginInstanceConfigV1, key: &str)
```



## bookclerk_library::control_plane::instance_config::set_setting

*Function*

Sets `key` to `value` in `settings`, or removes it when `value` is `None`.

The CLI and API use this for one-key edits; the full document still goes
through [`replace_instance_config`](instance_config.md#bookclerk_librarycontrol_planeinstance_configreplace_instance_config) with the pinned schema.

```rust
fn set_setting(body: & mut PluginInstanceConfigV1, key: &str, value: Option<SettingValue>)
```



