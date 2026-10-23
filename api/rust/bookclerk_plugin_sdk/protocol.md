# bookclerk_plugin_sdk::protocol

Shared role method names and typed method payloads.

Audience: guest authors who need the Cap'n Proto method parameter / result
structs (generated from `schema/plugin.capnp`) without depending on host
crates. Wire types live in `bookclerk_plugin_abi`; this module re-exports
them for a stable SDK import path.

## Trust boundary

External plugins are **untrusted** relative to the host. The host must never
hand them `library.db`, `master.key`, or the Bookclerk files-dir root. Plugins
receive only a scoped `plugin_data_dir` / `cache_dir`, and credentials are
host-mediated (login returns a blob the host seals; scan and fetch receive
that blob from the host). Scan returns book DTOs for the host to upsert.

Prefer product docs under `docs/plugins.md` for jail / capability rules.

| Item | Kind | Summary |
| --- | --- | --- |
| [`PROTOCOL_NAME`](#protocol_name) | constant | Logical ABI identifier for diagnostics (not a `plugin.toml` field). |
| [`MAX_RPC_LINE_BYTES`](#max_rpc_line_bytes) | constant | Maximum length of one framed bridge line in bytes (including newline). |

## Re-exports

- [`bookclerk_plugin_abi::Abridgement`](../bookclerk_plugin_abi/generated.md#abridgement)
- [`bookclerk_plugin_abi::AccountCredential`](../bookclerk_plugin_abi/generated.md#accountcredential)
- [`bookclerk_plugin_abi::AuthenticateUserParams`](../bookclerk_plugin_abi/generated.md#authenticateuserparams)
- [`bookclerk_plugin_abi::Brand`](../bookclerk_plugin_abi/generated.md#brand)
- [`bookclerk_plugin_abi::CatalogDetail`](../bookclerk_plugin_abi/generated.md#catalogdetail)
- [`bookclerk_plugin_abi::CatalogDetailParams`](../bookclerk_plugin_abi/generated.md#catalogdetailparams)
- [`bookclerk_plugin_abi::CatalogField`](../bookclerk_plugin_abi/generated.md#catalogfield)
- [`bookclerk_plugin_abi::CatalogHit`](../bookclerk_plugin_abi/generated.md#cataloghit)
- [`bookclerk_plugin_abi::CatalogHits`](../bookclerk_plugin_abi/generated.md#cataloghits)
- [`bookclerk_plugin_abi::CatalogSort`](../bookclerk_plugin_abi/generated.md#catalogsort)
- [`bookclerk_plugin_abi::ChapterMarker`](../bookclerk_plugin_abi/generated.md#chaptermarker)
- [`bookclerk_plugin_abi::CliArg`](../bookclerk_plugin_abi/generated.md#cliarg)
- [`bookclerk_plugin_abi::CliArgKind`](../bookclerk_plugin_abi/generated.md#cliargkind)
- [`bookclerk_plugin_abi::CliArgSpec`](../bookclerk_plugin_abi/generated.md#cliargspec)
- [`bookclerk_plugin_abi::CliCommandSpec`](../bookclerk_plugin_abi/generated.md#clicommandspec)
- [`bookclerk_plugin_abi::CliInvokeParams`](../bookclerk_plugin_abi/generated.md#cliinvokeparams)
- [`bookclerk_plugin_abi::CliInvokeResult`](../bookclerk_plugin_abi/generated.md#cliinvokeresult)
- [`bookclerk_plugin_abi::CliSchema`](../bookclerk_plugin_abi/generated.md#clischema)
- [`bookclerk_plugin_abi::ConfigOption`](../bookclerk_plugin_abi/generated.md#configoption)
- [`bookclerk_plugin_abi::ConfigOptionValue`](../bookclerk_plugin_abi/generated.md#configoptionvalue)
- [`bookclerk_plugin_abi::EventPollResult`](../bookclerk_plugin_abi/generated.md#eventpollresult)
- [`bookclerk_plugin_abi::ExpandCandidatesParams`](../bookclerk_plugin_abi/generated.md#expandcandidatesparams)
- [`bookclerk_plugin_abi::ExternalUser`](../bookclerk_plugin_abi/generated.md#externaluser)
- [`bookclerk_plugin_abi::FetchOptions`](../bookclerk_plugin_abi/generated.md#fetchoptions)
- [`bookclerk_plugin_abi::FetchTitleParams`](../bookclerk_plugin_abi/generated.md#fetchtitleparams)
- [`bookclerk_plugin_abi::ListDealsParams`](../bookclerk_plugin_abi/generated.md#listdealsparams)
- [`bookclerk_plugin_abi::ListeningProgress`](../bookclerk_plugin_abi/generated.md#listeningprogress)
- [`bookclerk_plugin_abi::LoginCompleteParams`](../bookclerk_plugin_abi/generated.md#logincompleteparams)
- [`bookclerk_plugin_abi::LoginParams`](../bookclerk_plugin_abi/generated.md#loginparams)
- [`bookclerk_plugin_abi::LoginResult`](../bookclerk_plugin_abi/generated.md#loginresult)
- [`bookclerk_plugin_abi::LoginStartResult`](../bookclerk_plugin_abi/generated.md#loginstartresult)
- [`bookclerk_plugin_abi::OutputLocalContextDto`](../bookclerk_plugin_abi/kind.md#outputlocalcontextdto)
- [`bookclerk_plugin_abi::OutputS3ContextDto`](../bookclerk_plugin_abi/kind.md#outputs3contextdto)
- [`bookclerk_plugin_abi::PlainFetch`](../bookclerk_plugin_abi/generated.md#plainfetch)
- [`bookclerk_plugin_abi::PlainPart`](../bookclerk_plugin_abi/generated.md#plainpart)
- [`bookclerk_plugin_abi::PortalAuthMode`](../bookclerk_plugin_abi/generated.md#portalauthmode)
- [`bookclerk_plugin_abi::PurchaseHint`](../bookclerk_plugin_abi/generated.md#purchasehint)
- [`bookclerk_plugin_abi::PurchaseHintParams`](../bookclerk_plugin_abi/generated.md#purchasehintparams)
- [`bookclerk_plugin_abi::PurchaseHintResult`](../bookclerk_plugin_abi/generated.md#purchasehintresult)
- [`bookclerk_plugin_abi::S3CredentialsDto`](../bookclerk_plugin_abi/kind.md#s3credentialsdto)
- [`bookclerk_plugin_abi::ScanBook`](../bookclerk_plugin_abi/generated.md#scanbook)
- [`bookclerk_plugin_abi::ScanLibraryParams`](../bookclerk_plugin_abi/generated.md#scanlibraryparams)
- [`bookclerk_plugin_abi::ScanParams`](../bookclerk_plugin_abi/generated.md#scanparams)
- [`bookclerk_plugin_abi::ScanSummary`](../bookclerk_plugin_abi/generated.md#scansummary)
- [`bookclerk_plugin_abi::SearchCatalogParams`](../bookclerk_plugin_abi/generated.md#searchcatalogparams)
- [`bookclerk_plugin_abi::SourceAccount`](../bookclerk_plugin_abi/generated.md#sourceaccount)
- [`bookclerk_plugin_abi::SourceAccounts`](../bookclerk_plugin_abi/generated.md#sourceaccounts)
- [`bookclerk_plugin_abi::SyncListeningResult`](../bookclerk_plugin_abi/generated.md#synclisteningresult)

## Constants

### PROTOCOL_NAME

```rust
pub const PROTOCOL_NAME: &str = "workers-rpc";
```

Logical ABI identifier for diagnostics (not a `plugin.toml` field).

Manifests advertise compatibility via `api_version` only.

### MAX_RPC_LINE_BYTES

```rust
pub const MAX_RPC_LINE_BYTES: usize = _;
```

Maximum length of one framed bridge line in bytes (including newline).

Used by the workerd bridge framing. Currently 16 MiB.
