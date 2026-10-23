**bookclerk_plugin_abi > kind**

# Module: kind

## Contents

**Structs**

- [`OutputLocalContextDto`](#outputlocalcontextdto) - Local filesystem destination knobs the host grants through
- [`OutputS3ContextDto`](#outputs3contextdto) - S3 destination knobs the host grants through `Bindings.config`.
- [`S3CredentialsDto`](#s3credentialsdto) - Static AWS-style credentials injected by the host (never read from guest env).

---

## bookclerk_plugin_abi::kind::OutputLocalContextDto

*Struct*

Local filesystem destination knobs the host grants through
`Bindings.config`.

**Fields:**
- `plugin_data_dir: String` - Scoped writable directory for this plugin only (`…/plugin-state/<PluginKey fs-id>/data`,
- `root: String` - Library output root (`[output.local].root`). Empty on the logical
- `prefix: String` - Key prefix under `root` (`[output.local].prefix`).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputLocalContextDto) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> OutputLocalContextDto`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::kind::OutputS3ContextDto

*Struct*

S3 destination knobs the host grants through `Bindings.config`.

**Fields:**
- `plugin_data_dir: String` - Scoped writable directory for this plugin only (`…/plugin-state/<PluginKey fs-id>/data`,
- `bucket: String` - Target bucket name.
- `prefix: String` - Key prefix under the bucket (`[output.s3].prefix`).
- `region: String` - AWS region (or compatible).
- `endpoint: Option<String>` - Optional custom endpoint (MinIO / path-style hosts); host may prepend
- `force_path_style: bool` - When true, force path-style addressing (wire `forcePathStyle`).
- `credentials: Option<S3CredentialsDto>` - When absent the guest may use the AWS SDK default provider chain

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputS3ContextDto) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> OutputS3ContextDto`



## bookclerk_plugin_abi::kind::S3CredentialsDto

*Struct*

Static AWS-style credentials injected by the host (never read from guest env).

[`Debug`](https://doc.rust-lang.org/stable/core/fmt/macros/derive.Debug.html) redacts secret fields. Present on [`OutputS3ContextDto`](kind.md#bookclerk_plugin_abikindoutputs3contextdto) when the
host has resolved keys from env or `encrypted_secrets`.

**Fields:**
- `access_key_id: String` - Access key id (wire `accessKeyId`).
- `secret_access_key: String` - Secret access key (wire `secretAccessKey`).
- `session_token: Option<String>` - Optional session token for temporary credentials (wire `sessionToken`).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> S3CredentialsDto`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &S3CredentialsDto) -> bool`



