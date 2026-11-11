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

Prefer product docs under `docs/plugins/README.md` for jail / capability rules.

| Item | Kind | Summary |
| --- | --- | --- |
| [`PROTOCOL_NAME`](#protocol_name) | constant | Logical ABI identifier for diagnostics (not a `plugin.toml` field). |
| [`MAX_RPC_LINE_BYTES`](#max_rpc_line_bytes) | constant | Maximum length of one framed bridge line in bytes (including newline). |

## Re-exports

- [`bookclerk_plugin_abi::Abridgement`](../../../internals/crates/bookclerk_plugin_abi/generated.md#abridgement)
- [`bookclerk_plugin_abi::AccountCredential`](../../../internals/crates/bookclerk_plugin_abi/generated.md#accountcredential)
- [`bookclerk_plugin_abi::AuthenticateUserParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#authenticateuserparams)
- [`bookclerk_plugin_abi::Brand`](../../../internals/crates/bookclerk_plugin_abi/generated.md#brand)
- [`bookclerk_plugin_abi::CatalogCapabilities`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogcapabilities)
- [`bookclerk_plugin_abi::CatalogDetail`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogdetail)
- [`bookclerk_plugin_abi::CatalogDetailParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogdetailparams)
- [`bookclerk_plugin_abi::CatalogField`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogfield)
- [`bookclerk_plugin_abi::CatalogHit`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cataloghit)
- [`bookclerk_plugin_abi::CatalogHits`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cataloghits)
- [`bookclerk_plugin_abi::CatalogSort`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogsort)
- [`bookclerk_plugin_abi::ChapterMarker`](../../../internals/crates/bookclerk_plugin_abi/generated.md#chaptermarker)
- [`bookclerk_plugin_abi::CliArg`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliarg)
- [`bookclerk_plugin_abi::CliArgKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliargkind)
- [`bookclerk_plugin_abi::CliArgSpec`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliargspec)
- [`bookclerk_plugin_abi::CliCommandSpec`](../../../internals/crates/bookclerk_plugin_abi/generated.md#clicommandspec)
- [`bookclerk_plugin_abi::CliInvokeParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliinvokeparams)
- [`bookclerk_plugin_abi::CliInvokeResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliinvokeresult)
- [`bookclerk_plugin_abi::CliSchema`](../../../internals/crates/bookclerk_plugin_abi/generated.md#clischema)
- [`bookclerk_plugin_abi::ConfigOption`](../../../internals/crates/bookclerk_plugin_abi/generated.md#configoption)
- [`bookclerk_plugin_abi::ConfigOptionValue`](../../../internals/crates/bookclerk_plugin_abi/generated.md#configoptionvalue)
- [`bookclerk_plugin_abi::Credit`](../../../internals/crates/bookclerk_plugin_abi/generated.md#credit)
- [`bookclerk_plugin_abi::CreditLevel`](../../../internals/crates/bookclerk_plugin_abi/generated.md#creditlevel)
- [`bookclerk_plugin_abi::CreditRole`](../../../internals/crates/bookclerk_plugin_abi/generated.md#creditrole)
- [`bookclerk_plugin_abi::EventPollResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#eventpollresult)
- [`bookclerk_plugin_abi::ExpandCandidatesParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#expandcandidatesparams)
- [`bookclerk_plugin_abi::ExpressionKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#expressionkind)
- [`bookclerk_plugin_abi::ExternalUser`](../../../internals/crates/bookclerk_plugin_abi/generated.md#externaluser)
- [`bookclerk_plugin_abi::FetchOptions`](../../../internals/crates/bookclerk_plugin_abi/generated.md#fetchoptions)
- [`bookclerk_plugin_abi::FetchTitleParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#fetchtitleparams)
- [`bookclerk_plugin_abi::Identifier`](../../../internals/crates/bookclerk_plugin_abi/generated.md#identifier)
- [`bookclerk_plugin_abi::IdentifierScheme`](../../../internals/crates/bookclerk_plugin_abi/generated.md#identifierscheme)
- [`bookclerk_plugin_abi::ListDealsParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#listdealsparams)
- [`bookclerk_plugin_abi::ListeningProgress`](../../../internals/crates/bookclerk_plugin_abi/generated.md#listeningprogress)
- [`bookclerk_plugin_abi::LoginCompleteParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#logincompleteparams)
- [`bookclerk_plugin_abi::LoginParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#loginparams)
- [`bookclerk_plugin_abi::LoginResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#loginresult)
- [`bookclerk_plugin_abi::LoginStartResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#loginstartresult)
- [`bookclerk_plugin_abi::MediaKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#mediakind)
- [`bookclerk_plugin_abi::OutputLocalContextDto`](../../../internals/crates/bookclerk_plugin_abi/kind.md#outputlocalcontextdto)
- [`bookclerk_plugin_abi::OutputS3ContextDto`](../../../internals/crates/bookclerk_plugin_abi/kind.md#outputs3contextdto)
- [`bookclerk_plugin_abi::PlainFetch`](../../../internals/crates/bookclerk_plugin_abi/generated.md#plainfetch)
- [`bookclerk_plugin_abi::PlainPart`](../../../internals/crates/bookclerk_plugin_abi/generated.md#plainpart)
- [`bookclerk_plugin_abi::PortalAuthMode`](../../../internals/crates/bookclerk_plugin_abi/generated.md#portalauthmode)
- [`bookclerk_plugin_abi::PurchaseHint`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehint)
- [`bookclerk_plugin_abi::PurchaseHintParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintparams)
- [`bookclerk_plugin_abi::PurchaseHintResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintresult)
- [`bookclerk_plugin_abi::PurchaseHintsParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintsparams)
- [`bookclerk_plugin_abi::PurchaseHintsResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintsresult)
- [`bookclerk_plugin_abi::RevokeAccountParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#revokeaccountparams)
- [`bookclerk_plugin_abi::S3CredentialsDto`](../../../internals/crates/bookclerk_plugin_abi/kind.md#s3credentialsdto)
- [`bookclerk_plugin_abi::ScanBook`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scanbook)
- [`bookclerk_plugin_abi::ScanLibraryParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scanlibraryparams)
- [`bookclerk_plugin_abi::ScanPage`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scanpage)
- [`bookclerk_plugin_abi::ScanParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scanparams)
- [`bookclerk_plugin_abi::SearchCatalogParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#searchcatalogparams)
- [`bookclerk_plugin_abi::SeriesEntry`](../../../internals/crates/bookclerk_plugin_abi/generated.md#seriesentry)
- [`bookclerk_plugin_abi::SourceAccount`](../../../internals/crates/bookclerk_plugin_abi/generated.md#sourceaccount)
- [`bookclerk_plugin_abi::SourceAccounts`](../../../internals/crates/bookclerk_plugin_abi/generated.md#sourceaccounts)
- [`bookclerk_plugin_abi::SyncListeningResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#synclisteningresult)
- [`bookclerk_plugin_abi::WorkKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#workkind)

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
