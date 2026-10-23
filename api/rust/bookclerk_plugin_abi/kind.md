# bookclerk_plugin_abi::kind

Output-plugin configuration DTOs carried as `application/json` inside
[`crate::ExtensibleConfig`](index.md#extensibleconfig) (`Bindings.config` for the `storage` entrypoint).

Field names serialize as **camelCase** on the wire. Every method payload
is a typed Cap'n Proto struct generated into [`crate::generated`](generated.md); only the
operator-configured destination knobs remain JSON because they are
plugin-private configuration, not ABI method parameters.

| Item | Kind | Summary |
| --- | --- | --- |
| [`S3CredentialsDto`](#s3credentialsdto) | struct | Static AWS-style credentials injected by the host (never read from guest env). |
| [`OutputS3ContextDto`](#outputs3contextdto) | struct | S3 destination knobs the host grants through `Bindings.config`. |
| [`OutputLocalContextDto`](#outputlocalcontextdto) | struct | Local filesystem destination knobs the host grants through `Bindings.config`. |

## Structs

### S3CredentialsDto

```rust
pub struct S3CredentialsDto {
    pub access_key_id: String,
    pub secret_access_key: String,
    pub session_token: Option<String>,
}
```

Static AWS-style credentials injected by the host (never read from guest env).

[`Debug`](https://doc.rust-lang.org/stable/core/fmt/macros/derive.Debug.html) redacts secret fields. Present on [`OutputS3ContextDto`](#outputs3contextdto) when the
host has resolved keys from env or `encrypted_secrets`.

**Fields**

- `access_key_id`: Access key id (wire `accessKeyId`).
- `secret_access_key`: Secret access key (wire `secretAccessKey`).
- `session_token`: Optional session token for temporary credentials (wire `sessionToken`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OutputS3ContextDto

```rust
pub struct OutputS3ContextDto {
    pub plugin_data_dir: String,
    pub bucket: String,
    pub prefix: String,
    pub region: String,
    pub endpoint: Option<String>,
    pub force_path_style: bool,
    pub credentials: Option<S3CredentialsDto>,
}
```

S3 destination knobs the host grants through `Bindings.config`.

**Fields**

- `plugin_data_dir`: Scoped writable directory for this plugin only (`…/plugin-state/<PluginKey fs-id>/data`,
  wire `pluginDataDir`). Empty on the logical ABI (jail layout is
  transport-private).
- `bucket`: Target bucket name.
- `prefix`: Key prefix under the bucket (`[output.s3].prefix`).
- `region`: AWS region (or compatible).
- `endpoint`: Optional custom endpoint (MinIO / path-style hosts); host may prepend
  `https://` for bare hostnames.
- `force_path_style`: When true, force path-style addressing (wire `forcePathStyle`).
- `credentials`: When absent the guest may use the AWS SDK default provider chain
  (unconfined dev only).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OutputLocalContextDto

```rust
pub struct OutputLocalContextDto {
    pub plugin_data_dir: String,
    pub root: String,
    pub prefix: String,
}
```

Local filesystem destination knobs the host grants through
`Bindings.config`.

**Fields**

- `plugin_data_dir`: Scoped writable directory for this plugin only (`…/plugin-state/<PluginKey fs-id>/data`,
  wire `pluginDataDir`). Empty on the logical ABI (jail layout is
  transport-private).
- `root`: Library output root (`[output.local].root`). Empty on the logical
  ABI; native guests read `BOOKCLERK_OUTPUT_LOCAL_ROOT` instead.
- `prefix`: Key prefix under `root` (`[output.local].prefix`).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`
