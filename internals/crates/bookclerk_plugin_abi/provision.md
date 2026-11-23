# bookclerk_plugin_abi::provision

The checks the host runs on what a plugin's guided setup returns.

`Provisioning.provision` hands back settings to store and credentials to
seal. Both cross a trust boundary, so the host checks them with
[`check_provisioned`](#check_provisioned) before anything is written:

- every key is one the manifest lists in `[provisioning] outputs`, as a
  setting or as a secret, and no key comes twice;
- settings are a flat JSON object of strings, numbers and booleans;
- credentials are non-empty single-line text within
  [`MAX_PROVISION_SECRET_BYTES`](README.md#max_provision_secret_bytes), at most [`MAX_PROVISION_SECRETS`](README.md#max_provision_secrets) of
  them;
- the summary is a few short printable lines;
- neither a returned credential nor the operator's grant shows up in a
  setting value or a summary line, since those are stored and shown in
  the clear.

The host then checks each value against the pinned settings schema. The
fuzz target `provision_reply` drives the same code.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ProvisionOutputs`](#provisionoutputs) | struct | The keys a plugin's guided setup may write (`[provisioning] outputs`), split by whether the setting is declared `type = "secret"`. |
| [`ProvisionedCredential`](#provisionedcredential) | struct | A credential a guided setup returned, held for sealing. `Debug` never prints it. |
| [`CheckedProvision`](#checkedprovision) | struct | What a guided setup returned, after `check_provisioned`. |
| [`ProvisionRefusal`](#provisionrefusal) | enum | Why `check_provisioned` refused a reply. The text names keys and limits, never a value. |
| [`check_provisioned`](#check_provisioned) | function | Checks a plugin's `Provisioned` against its declared outputs. |
| [`decode_provision_reply_bytes`](#decode_provision_reply_bytes) | function | Decodes a `ProvisionReply` from an unpacked Cap'n Proto message, the form a plugin's answer crosses the plugin boundary in. |
| [`encode_provision_reply_bytes`](#encode_provision_reply_bytes) | function | Encodes a `ProvisionReply`, the inverse of `decode_provision_reply_bytes`. |
| [`MAX_PROVISION_SETTING_VALUE_BYTES`](#max_provision_setting_value_bytes) | constant | Longest string a guided setup may store as one setting, in bytes. |

## Structs

### ProvisionOutputs

```rust
pub struct ProvisionOutputs {
    pub settings: BTreeSet<String>,
    pub secrets: BTreeSet<String>,
}
```

The keys a plugin's guided setup may write (`[provisioning] outputs`),
split by whether the setting is declared `type = "secret"`.

**Fields**

- `settings`: Non-secret setting keys, stored in the instance's settings document.
- `secrets`: `secret` setting keys, sealed into the secret store.

Implements: `Clone`, `Debug`, `Default`, `Eq`, `PartialEq`

### ProvisionedCredential

```rust
pub struct ProvisionedCredential(/* private */);
```

A credential a guided setup returned, held for sealing. `Debug` never
prints it.

#### ProvisionedCredential::expose

```rust
pub fn expose(&self) -> &str
```

The credential's text, for sealing.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### CheckedProvision

```rust
pub struct CheckedProvision {
    pub settings: BTreeMap<String, Value>,
    pub secrets: BTreeMap<String, ProvisionedCredential>,
    pub summary: Vec<String>,
}
```

What a guided setup returned, after [`check_provisioned`](#check_provisioned).

**Fields**

- `settings`: Settings to store, keyed by setting key: strings, numbers or
  booleans.
- `secrets`: Credentials to seal, keyed by `secret` setting key.
- `summary`: Lines for the setup screen.

Implements: `Clone`, `Debug`, `Default`, `PartialEq`

## Enums

### ProvisionRefusal

```rust
pub enum ProvisionRefusal {
    SettingsTooLong,
    SettingsNotObject,
    UndeclaredSetting(String),
    BadSettingValue(String),
    TooManySecrets,
    UndeclaredSecret(String),
    RepeatedSecret(String),
    BadSecretValue(String),
    TooManySummaryLines,
    BadSummaryLine(usize),
    CredentialLeak,
}
```

Why [`check_provisioned`](#check_provisioned) refused a reply. The text names keys and
limits, never a value.

**Variants**

- `SettingsTooLong`: `settingsJson` is longer than [`MAX_PROVISION_SETTINGS_BYTES`](README.md#max_provision_settings_bytes).
- `SettingsNotObject`: `settingsJson` is not a JSON object, or repeats a key.
- `UndeclaredSetting`: A setting key is not a non-secret key in `[provisioning] outputs`.
- `BadSettingValue`: A setting value is not a string, number or boolean, or is a string
  that is too long or holds a control character.
- `TooManySecrets`: More than [`MAX_PROVISION_SECRETS`](README.md#max_provision_secrets) credentials.
- `UndeclaredSecret`: A credential key is not a secret key in `[provisioning] outputs`.
- `RepeatedSecret`: A credential key came twice.
- `BadSecretValue`: A credential is empty, too long, holds a control character or has
  space at either end.
- `TooManySummaryLines`: More than [`MAX_PROVISION_SUMMARY_LINES`](README.md#max_provision_summary_lines) summary lines.
- `BadSummaryLine`: A summary line is too long or holds a control character.
- `CredentialLeak`: A credential or the grant appears in a setting value or a summary
  line, which would store or show it in the clear.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<ProvisionRefusal>`, `PartialEq`

## Functions

### check_provisioned

```rust
pub fn check_provisioned(
    provisioned: &Provisioned,
    outputs: &ProvisionOutputs,
    grant_token: Option<&str>,
) -> Result<CheckedProvision, ProvisionRefusal>
```

Checks a plugin's [`Provisioned`](generated.md#provisioned) against its declared outputs.

#### Arguments

* `provisioned` - What `Provisioning.provision` returned.
* `outputs` - The manifest's `[provisioning] outputs`, split into
  settings and secrets.
* `grant_token` - The token the operator gave this run, which must not
  be stored in the clear; `None` when there was none.

#### Returns

The settings, credentials and summary, each checked.

#### Errors

Returns the first [`ProvisionRefusal`](#provisionrefusal) that applies.

### decode_provision_reply_bytes

```rust
pub fn decode_provision_reply_bytes(
    bytes: &[u8],
) -> Result<Result<Provisioned, PluginError>>
```

Decodes a `ProvisionReply` from an unpacked Cap'n Proto message, the
form a plugin's answer crosses the plugin boundary in.

#### Arguments

* `bytes` - The message bytes.

#### Returns

The reply: what to store, or the plugin's typed error.

#### Errors

Returns the Cap'n Proto error the reader reports.

### encode_provision_reply_bytes

```rust
pub fn encode_provision_reply_bytes(
    reply: &Result<Provisioned, PluginError>,
) -> Result<Vec<u8>>
```

Encodes a `ProvisionReply`, the inverse of
[`decode_provision_reply_bytes`](#decode_provision_reply_bytes).

#### Arguments

* `reply` - The reply to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

## Constants

### MAX_PROVISION_SETTING_VALUE_BYTES

```rust
pub const MAX_PROVISION_SETTING_VALUE_BYTES: usize = crate::MAX_SETTING_STRING_BYTES;
```

Longest string a guided setup may store as one setting, in bytes.
