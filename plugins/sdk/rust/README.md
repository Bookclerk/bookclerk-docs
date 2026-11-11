# Rust plugin SDK

This page shows how to write a Bookclerk plugin in Rust with
`bookclerk-plugin-sdk` (`api_version = 3`). A Rust plugin is a native
guest, a binary the host runs in its own jail, or a workerd guest when
compiled to Wasm with the SDK's `workerd` glue. Depend on this crate
alone, never on `bookclerk-plugin-host`, which pulls in the host's
internals.

## Install

```toml
[dependencies]
bookclerk-plugin-sdk = { git = "https://github.com/Bookclerk/bookclerk", package = "bookclerk-plugin-sdk" }
# Database guests also need:
# bookclerk-plugin-sdk = { …, features = ["db"] }
```

| Feature | Purpose |
| --- | --- |
| *(default)* | The Cap'n Proto guest runner, fetch and upload path helpers, the callback tunnel, and the ABI re-exports |
| `db` | SeaORM to Workers RPC DTO helpers for database guests |
| `http` | Native HTTPS through the workerd socket proxy |

## Author model

A native guest implements `PluginWorker` (`describe`, and
`open(invocation, bindings) -> Entrypoints`) and calls `serve`, the
stdin and stdout Cap'n Proto runner. The entrypoints it returns are the
same named entrypoints the other SDKs export as classes: storefront,
storage, remote library, database adapter, CLI, OIDC, metadata and
covers. The
crate root re-exports the DTOs and helpers a guest needs; prefer
`use bookclerk_plugin_sdk::…` over reaching into submodules.

Identity (`apiVersion`, `id`, capabilities) comes from `plugin.toml`;
`describe` only refines `displayName` and similar fields.

A `[[databases]]` binding is taken from the `open` bindings with
`DatabaseBinding::take_named_from_bindings` and used as on D1
(`prepare`, `bind`, `all`, `first`, `batch`). Its `limits()` returns the
session's `DbCapabilities`; `all()` reads through the session's streamed
budget when it advertises one, `iterate()` walks the same read one row at
a time without holding the whole result, and `chunk()` splits a long statement list
into batches that each fit one request. Any cap that is hit is a
`PluginError` with `PluginErrorCode::LimitExceeded` and a typed `limit()`;
see [Working within database limits](../../database-limits.md). A guest's
tests open the same binding with `bookclerk_db_testkit::BindingDatabase`
under `bookclerk_db_testkit::selected_profile()`, so `bookclerk-plugin test .`
runs them under every profile (see
[Testing against the limits](../../database-limits.md#testing-against-the-limits)).
[`examples/plugins-echo-native-rust`](../../../../examples/plugins-echo-native-rust)
does all of this end to end in its `notes` table.

## The socket proxy

A native guest has no ambient network. It receives an inherited link
(`fd:<n>` on Unix, `handle:<n>` on Windows) in `BOOKCLERK_SOCKET_PROXY`
and multiplexes CONNECT streams over it with the `mux` module; `net`
opens mediated TCP sockets, the equivalent of Workers `connect()`, and
the `http` feature puts an HTTPS client on top. Only hosts the plugin's
`[capabilities.network]` grants name are reachable.

Older SDKs that only opened a pathname or named-pipe proxy cannot reach
the host and fail closed; rebuild guests against this SDK. Pathname,
`abstract:` and `\\.\pipe\` forms remain for tests.

## Author tools

The `bookclerk-plugin` author CLI (`check`, `fmt`, `sync-embed`,
`package`, `smoke` and `test`) lives in the separate `bookclerk-plugin-tools`
crate, so a guest that depends on this SDK never links
`bookclerk-workerd`. Install it once, then run it from the plugin's
directory:

```bash
cargo install --git https://github.com/Bookclerk/bookclerk bookclerk-plugin-tools
cargo install --git https://github.com/Bookclerk/bookclerk bookclerk-db-testkit  # the kit `test` serves
bookclerk-plugin check .
bookclerk-plugin smoke .
bookclerk-plugin test .    # the plugin's suite under every database profile
```

Inside a Bookclerk checkout, `cargo plugin -- check <dir>` is an alias
for the same binary (`.cargo/config.toml`).

## Next steps

- [plugin.toml](../../manifest.md): declare the entrypoints the guest
  serves.
- [Entrypoints, events and the ABI](../../abi.md): every method and
  event in detail.
- [`bookclerk_plugin_abi`](../../../internals/crates/bookclerk_plugin_abi/README.md)
  and [`bookclerk_plugin_manifest`](../../../internals/crates/bookclerk_plugin_manifest/README.md):
  the ABI and manifest crates the SDK re-exports.
- [Try the Echo example](../../README.md#try-the-echo-example): complete
  native and workerd Rust plugins under `examples/`.

## Reference

The sections below are generated from the crate's source: the crate
root, with its items and re-exports, and one page per public module.

Guest-only Bookclerk plugin SDK.

For third-party Rust plugin authors of native and workerd plugins
(`api_version = 3`). Depend on this crate from a standalone plugin
workspace, not on the host `bookclerk-plugin-host` crate. The Rust SDK
guide (`docs/plugins/sdk/rust/README.md`, published with this reference)
covers the author model, the socket proxy and the author CLI; the entry
points are:

| Need | Entry point |
| --- | --- |
| Native guest (`runtime = "native"`) | [`PluginWorker`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginworker) / [`serve`](#serve) (`api_version = 3`) |
| Fetch / upload work paths | [`fetch_work_dir`](#fetch_work_dir), [`upload_file_path`](#upload_file_path) |
| Mediated TCP sockets | [`connect_socket`](net.md#connect) / [`net`](net.md) (Workers `connect()` equivalent) |
| Native HTTPS through the socket proxy | feature `http` → `http::Client` |
| OAuth callback without guest listen | [`callback_tunnel`](callback_tunnel.md) |
| Workerd / Wasm guests | [`workerd`](workerd.md) + npm `@bookclerk/plugin-sdk` |
| ABI DTOs / method names | [`protocol`](protocol.md) (re-exports `bookclerk-plugin-abi`) |
| Database guest session / atomic execution | feature `db` → [`database_adapter`](database_adapter/README.md) |
| Author CLI (`check` / `fmt` / `sync-embed` / `package` / `smoke`) | separate crate `bookclerk-plugin-tools` (`cargo plugin`) |

Crate-root `pub use` items are the stable import surface for guests
(describe-metadata DTOs, [`PluginError`](../../../internals/crates/bookclerk_plugin_abi/error.md#pluginerror), fetch path helpers, tunnel
types). Prefer `use bookclerk_plugin_sdk::…` over reaching into
submodules unless you need module-level docs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ChunkCaps`](#chunkcaps) | struct | Caps `DatabaseBinding::chunk` splits a batch by when the session's limits are not to be used. |
| [`D1ExecResult`](#d1execresult) | struct | Cloudflare `D1ExecResult`. |
| [`D1Meta`](#d1meta) | struct | Cloudflare `D1Result::meta` shape. |
| [`D1Result`](#d1result) | struct | Cloudflare `D1Result` projection. |
| [`DatabaseBinding`](#databasebinding) | struct | Host-mediated typed SQL surface for plugin guests. |
| [`DatabaseBindingOptions`](#databasebindingoptions) | struct | Options for `DatabaseBinding`. |
| [`PreparedStatement`](#preparedstatement) | struct | Cloudflare-style prepared statement. |
| [`RetryToken`](#retrytoken) | struct | Explicit retry identity: reuse both `operationId` and `requestHash`. |
| [`RowStream`](#rowstream) | struct | Rows of one read, handed out one at a time (`PreparedStatement::iterate`). |
| [`FetchWorkDir`](#fetchworkdir) | struct | An open fetch work directory received from the host for one `fetchTitle` call. |
| [`UploadFile`](#uploadfile) | struct | An open upload file received from the host for one `putFile` call. |
| [`SdkError`](#sdkerror) | enum | Errors from the guest stdio loop and SDK helpers (not plugin business logic). |
| [`db_value_from_sea`](#db_value_from_sea) | function | Fallible SeaORM → `bookclerk_plugin_abi::DbValue` bridge. |
| [`proxy_rows_from_typed`](#proxy_rows_from_typed) | function | Builds SeaORM proxy rows from a typed statement result. |
| [`chunk_indexes`](#chunk_indexes) | function | Greedy split of `typed` into index runs that each fit one request. |
| [`execute_reply_to_d1_results`](#execute_reply_to_d1_results) | function | Maps a typed execute reply to Cloudflare `D1Result` per statement. |
| [`statement_result_to_d1_result`](#statement_result_to_d1_result) | function | Maps one typed statement result to Cloudflare `D1Result`. |
| [`fetch_work_dir`](#fetch_work_dir) | function | Resolves the directory a `fetchTitle` call should write into. |
| [`upload_file_path`](#upload_file_path) | function | Resolves the local file path for a `putFile` call. |
| [`decode_json`](#decode_json) | function | Deserialize a role-method JSON argument. |
| [`encode_json`](#encode_json) | function | Serialize a role-method JSON result. |
| [`encode_atomic_result`](#encode_atomic_result) | function | Encodes an atomic result; an oversized scalar after commit is `unavailable`. |
| [`page_rows`](#page_rows) | function | Page a JSON array of rows. `limit == 0` means `MAX_LIST_PAGE`. |
| [`manifest_capabilities`](#manifest_capabilities) | function | Parses `plugin.toml` source and returns the `PluginCapabilities` a guest must advertise from `describe()`. |
| [`fd_proc_path`](#fd_proc_path) | function | Builds the OS path that refers to an open descriptor by number. |
| [`recv_passed_fd`](#recv_passed_fd) | function | Receives one file descriptor from the host side channel (SCM_RIGHTS). |
| [`serve`](#serve) | function | Serves a `PluginWorker` on stdin/stdout (Cap'n Proto RPC). |
| [`Result`](#result) | type alias | Result alias for guest helpers that fail with `SdkError`. |
| [`PLUGIN_FD_CHANNEL`](#plugin_fd_channel) | constant | Fixed descriptor number the host leaves open for the plugin side channel. |
| [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env) | constant | Environment variable the host *would* set to the decimal form of `PLUGIN_FD_CHANNEL`. |

## Re-exports

- [`callback_tunnel::TunnelGuest`](callback_tunnel.md#tunnelguest)
- [`callback_tunnel::TunnelHost`](callback_tunnel.md#tunnelhost)
- [`callback_tunnel::TunnelStream`](callback_tunnel.md#tunnelstream)
- [`net::connect` as `connect_socket`](net.md#connect)
- [`net::nested_native_jail_requested`](net.md#nested_native_jail_requested)
- [`net::ConnectOptions`](net.md#connectoptions)
- [`net::PluginSocket`](net.md#pluginsocket)
- [`net::SecureTransport`](net.md#securetransport)
- [`net::SocketAddress`](net.md#socketaddress)
- [`net::GUEST_IPC_DIR_ENV`](net.md#guest_ipc_dir_env)
- [`net::NESTED_NATIVE_JAIL_ENV`](net.md#nested_native_jail_env)
- [`net::OWN_USER_AGENT_HOSTS_ENV`](net.md#own_user_agent_hosts_env)
- [`net::SESSION_CHALLENGE_ENV`](net.md#session_challenge_env)
- [`net::SESSION_CHALLENGE_LEN`](net.md#session_challenge_len)
- [`net::SOCKET_PROXY_ABSTRACT_PREFIX`](net.md#socket_proxy_abstract_prefix)
- [`net::SOCKET_PROXY_ENV`](net.md#socket_proxy_env)
- [`net::SOCKET_PROXY_WRITE_ENV`](net.md#socket_proxy_write_env)
- [`net::TEST_EXTRA_CANDIDATE_ENV`](net.md#test_extra_candidate_env)
- [`net::TEST_EXTRA_CANDIDATE_WRITE_ENV`](net.md#test_extra_candidate_write_env)
- [`net::TEST_EXTRA_ENDPOINT_ENV`](net.md#test_extra_endpoint_env)
- [`net::TEST_EXTRA_ENDPOINT_WRITE_ENV`](net.md#test_extra_endpoint_write_env)
- [`net::USER_AGENT_ENV`](net.md#user_agent_env)
- [`protocol::Abridgement`](../../../internals/crates/bookclerk_plugin_abi/generated.md#abridgement)
- [`protocol::AccountCredential`](../../../internals/crates/bookclerk_plugin_abi/generated.md#accountcredential)
- [`protocol::AuthenticateUserParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#authenticateuserparams)
- [`protocol::Brand`](../../../internals/crates/bookclerk_plugin_abi/generated.md#brand)
- [`protocol::CatalogCapabilities`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogcapabilities)
- [`protocol::CatalogDetail`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogdetail)
- [`protocol::CatalogDetailParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogdetailparams)
- [`protocol::CatalogField`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogfield)
- [`protocol::CatalogHit`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cataloghit)
- [`protocol::CatalogHits`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cataloghits)
- [`protocol::CatalogSort`](../../../internals/crates/bookclerk_plugin_abi/generated.md#catalogsort)
- [`protocol::ChapterMarker`](../../../internals/crates/bookclerk_plugin_abi/generated.md#chaptermarker)
- [`protocol::CliArg`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliarg)
- [`protocol::CliArgKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliargkind)
- [`protocol::CliArgSpec`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliargspec)
- [`protocol::CliCommandSpec`](../../../internals/crates/bookclerk_plugin_abi/generated.md#clicommandspec)
- [`protocol::CliInvokeParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliinvokeparams)
- [`protocol::CliInvokeResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#cliinvokeresult)
- [`protocol::CliSchema`](../../../internals/crates/bookclerk_plugin_abi/generated.md#clischema)
- [`protocol::ConfigOption`](../../../internals/crates/bookclerk_plugin_abi/generated.md#configoption)
- [`protocol::ConfigOptionValue`](../../../internals/crates/bookclerk_plugin_abi/generated.md#configoptionvalue)
- [`protocol::Credit`](../../../internals/crates/bookclerk_plugin_abi/generated.md#credit)
- [`protocol::CreditLevel`](../../../internals/crates/bookclerk_plugin_abi/generated.md#creditlevel)
- [`protocol::CreditRole`](../../../internals/crates/bookclerk_plugin_abi/generated.md#creditrole)
- [`protocol::EventPollResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#eventpollresult)
- [`protocol::ExpandCandidatesParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#expandcandidatesparams)
- [`protocol::ExpressionKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#expressionkind)
- [`protocol::ExternalUser`](../../../internals/crates/bookclerk_plugin_abi/generated.md#externaluser)
- [`protocol::FetchOptions`](../../../internals/crates/bookclerk_plugin_abi/generated.md#fetchoptions)
- [`protocol::FetchTitleParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#fetchtitleparams)
- [`protocol::Identifier`](../../../internals/crates/bookclerk_plugin_abi/generated.md#identifier)
- [`protocol::IdentifierScheme`](../../../internals/crates/bookclerk_plugin_abi/generated.md#identifierscheme)
- [`protocol::ListDealsParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#listdealsparams)
- [`protocol::ListeningProgress`](../../../internals/crates/bookclerk_plugin_abi/generated.md#listeningprogress)
- [`protocol::LoginCompleteParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#logincompleteparams)
- [`protocol::LoginParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#loginparams)
- [`protocol::LoginResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#loginresult)
- [`protocol::LoginStartResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#loginstartresult)
- [`protocol::MediaKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#mediakind)
- [`protocol::OutputLocalContextDto`](../../../internals/crates/bookclerk_plugin_abi/kind.md#outputlocalcontextdto)
- [`protocol::OutputS3ContextDto`](../../../internals/crates/bookclerk_plugin_abi/kind.md#outputs3contextdto)
- [`protocol::PlainFetch`](../../../internals/crates/bookclerk_plugin_abi/generated.md#plainfetch)
- [`protocol::PlainPart`](../../../internals/crates/bookclerk_plugin_abi/generated.md#plainpart)
- [`protocol::PortalAuthMode`](../../../internals/crates/bookclerk_plugin_abi/generated.md#portalauthmode)
- [`protocol::PurchaseHint`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehint)
- [`protocol::PurchaseHintParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintparams)
- [`protocol::PurchaseHintResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintresult)
- [`protocol::PurchaseHintsParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintsparams)
- [`protocol::PurchaseHintsResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#purchasehintsresult)
- [`protocol::RevokeAccountParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#revokeaccountparams)
- [`protocol::S3CredentialsDto`](../../../internals/crates/bookclerk_plugin_abi/kind.md#s3credentialsdto)
- [`protocol::ScanBook`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scanbook)
- [`protocol::ScanLibraryParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scanlibraryparams)
- [`protocol::ScanParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scanparams)
- [`protocol::ScanSummary`](../../../internals/crates/bookclerk_plugin_abi/generated.md#scansummary)
- [`protocol::SearchCatalogParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#searchcatalogparams)
- [`protocol::SeriesEntry`](../../../internals/crates/bookclerk_plugin_abi/generated.md#seriesentry)
- [`protocol::SourceAccount`](../../../internals/crates/bookclerk_plugin_abi/generated.md#sourceaccount)
- [`protocol::SourceAccounts`](../../../internals/crates/bookclerk_plugin_abi/generated.md#sourceaccounts)
- [`protocol::SyncListeningResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#synclisteningresult)
- [`protocol::WorkKind`](../../../internals/crates/bookclerk_plugin_abi/generated.md#workkind)
- [`protocol::MAX_RPC_LINE_BYTES`](protocol.md#max_rpc_line_bytes)
- [`protocol::PROTOCOL_NAME`](protocol.md#protocol_name)
- [`bookclerk_plugin_abi::byte_source_from_async_read`](../../../internals/crates/bookclerk_plugin_abi/README.md#byte_source_from_async_read)
- [`bookclerk_plugin_abi::canonical_execute_request_hash`](../../../internals/crates/bookclerk_plugin_abi/README.md#canonical_execute_request_hash)
- [`bookclerk_plugin_abi::connect_plugin`](../../../internals/crates/bookclerk_plugin_abi/README.md#connect_plugin)
- [`bookclerk_plugin_abi::database_adapter_config_from_bindings`](../../../internals/crates/bookclerk_plugin_abi/db.md#database_adapter_config_from_bindings)
- [`bookclerk_plugin_abi::decode_db_value_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#decode_db_value_bytes)
- [`bookclerk_plugin_abi::decode_execute_request_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#decode_execute_request_bytes)
- [`bookclerk_plugin_abi::decode_execute_result_reply_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#decode_execute_result_reply_bytes)
- [`bookclerk_plugin_abi::encoded_db_value_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#encoded_db_value_bytes)
- [`bookclerk_plugin_abi::encoded_execute_reply_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#encoded_execute_reply_bytes)
- [`bookclerk_plugin_abi::encoded_execute_request_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#encoded_execute_request_bytes)
- [`bookclerk_plugin_abi::encoded_execute_result_reply_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#encoded_execute_result_reply_bytes)
- [`bookclerk_plugin_abi::encoded_statement_result_bytes`](../../../internals/crates/bookclerk_plugin_abi/README.md#encoded_statement_result_bytes)
- [`bookclerk_plugin_abi::negotiate_rpc_features`](../../../internals/crates/bookclerk_plugin_abi/README.md#negotiate_rpc_features)
- [`bookclerk_plugin_abi::pull_byte_source_to_writer`](../../../internals/crates/bookclerk_plugin_abi/README.md#pull_byte_source_to_writer)
- [`bookclerk_plugin_abi::require_plugin_migration_registration`](../../../internals/crates/bookclerk_plugin_abi/README.md#require_plugin_migration_registration)
- [`bookclerk_plugin_abi::serve_plugin`](../../../internals/crates/bookclerk_plugin_abi/README.md#serve_plugin)
- [`bookclerk_plugin_abi::serve_plugin_stdio`](../../../internals/crates/bookclerk_plugin_abi/README.md#serve_plugin_stdio)
- [`bookclerk_plugin_abi::sql_payload_bytes`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#sql_payload_bytes)
- [`bookclerk_plugin_abi::sql_payload_exceeds`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#sql_payload_exceeds)
- [`bookclerk_plugin_abi::stream_copy_keys`](../../../internals/crates/bookclerk_plugin_abi/README.md#stream_copy_keys)
- [`bookclerk_plugin_abi::AdapterBackupOps`](../../../internals/crates/bookclerk_plugin_abi/backup_ops.md#adapterbackupops)
- [`bookclerk_plugin_abi::AdapterDatabaseSession`](../../../internals/crates/bookclerk_plugin_abi/README.md#adapterdatabasesession)
- [`bookclerk_plugin_abi::AdapterExecuteRequest`](../../../internals/crates/bookclerk_plugin_abi/README.md#adapterexecuterequest)
- [`bookclerk_plugin_abi::BindingValues`](../../../internals/crates/bookclerk_plugin_abi/README.md#bindingvalues)
- [`bookclerk_plugin_abi::Bindings`](../../../internals/crates/bookclerk_plugin_abi/README.md#bindings)
- [`bookclerk_plugin_abi::ByteRange`](../../../internals/crates/bookclerk_plugin_abi/README.md#byterange)
- [`bookclerk_plugin_abi::Cancellation`](../../../internals/crates/bookclerk_plugin_abi/README.md#cancellation)
- [`bookclerk_plugin_abi::ContentSource`](../../../internals/crates/bookclerk_plugin_abi/README.md#contentsource)
- [`bookclerk_plugin_abi::ContentSourceClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#contentsourceclient)
- [`bookclerk_plugin_abi::CopyResult`](../../../internals/crates/bookclerk_plugin_abi/README.md#copyresult)
- [`bookclerk_plugin_abi::CoverFetchParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#coverfetchparams)
- [`bookclerk_plugin_abi::CoverFetcher`](../../../internals/crates/bookclerk_plugin_abi/README.md#coverfetcher)
- [`bookclerk_plugin_abi::CoverFetcherClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#coverfetcherclient)
- [`bookclerk_plugin_abi::CoverImage`](../../../internals/crates/bookclerk_plugin_abi/generated.md#coverimage)
- [`bookclerk_plugin_abi::CoverImages`](../../../internals/crates/bookclerk_plugin_abi/generated.md#coverimages)
- [`bookclerk_plugin_abi::Database`](../../../internals/crates/bookclerk_plugin_abi/README.md#database)
- [`bookclerk_plugin_abi::DatabaseAdapterConfig`](../../../internals/crates/bookclerk_plugin_abi/generated.md#databaseadapterconfig)
- [`bookclerk_plugin_abi::DatabaseClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#databaseclient)
- [`bookclerk_plugin_abi::DbBootstrap`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbbootstrap)
- [`bookclerk_plugin_abi::DbCapabilities`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbcapabilities)
- [`bookclerk_plugin_abi::DbColumn`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbcolumn)
- [`bookclerk_plugin_abi::DbIdentityHighWater`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbidentityhighwater)
- [`bookclerk_plugin_abi::DbPlanStatementKind`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbplanstatementkind)
- [`bookclerk_plugin_abi::DbResultSelection`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbresultselection)
- [`bookclerk_plugin_abi::DbRow`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbrow)
- [`bookclerk_plugin_abi::DbTiming`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#dbtiming)
- [`bookclerk_plugin_abi::DbType`](../../../internals/crates/bookclerk_plugin_abi/db_value.md#dbtype)
- [`bookclerk_plugin_abi::DbValue`](../../../internals/crates/bookclerk_plugin_abi/db_value.md#dbvalue)
- [`bookclerk_plugin_abi::Destination`](../../../internals/crates/bookclerk_plugin_abi/README.md#destination)
- [`bookclerk_plugin_abi::DestinationClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#destinationclient)
- [`bookclerk_plugin_abi::DestinationServer`](../../../internals/crates/bookclerk_plugin_abi/README.md#destinationserver)
- [`bookclerk_plugin_abi::DiagnoseResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#diagnoseresult)
- [`bookclerk_plugin_abi::DomainEvent`](../../../internals/crates/bookclerk_plugin_abi/README.md#domainevent)
- [`bookclerk_plugin_abi::Entrypoints`](../../../internals/crates/bookclerk_plugin_abi/README.md#entrypoints)
- [`bookclerk_plugin_abi::EventConsumer`](../../../internals/crates/bookclerk_plugin_abi/README.md#eventconsumer)
- [`bookclerk_plugin_abi::EventConsumerClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#eventconsumerclient)
- [`bookclerk_plugin_abi::EventPublisher`](../../../internals/crates/bookclerk_plugin_abi/README.md#eventpublisher)
- [`bookclerk_plugin_abi::EventPublisherClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#eventpublisherclient)
- [`bookclerk_plugin_abi::EventResult`](../../../internals/crates/bookclerk_plugin_abi/README.md#eventresult)
- [`bookclerk_plugin_abi::ExecuteReply`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#executereply)
- [`bookclerk_plugin_abi::ExecuteRequest`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#executerequest)
- [`bookclerk_plugin_abi::ExecuteStream`](../../../internals/crates/bookclerk_plugin_abi/db_stream.md#executestream)
- [`bookclerk_plugin_abi::ExecuteStreamLimits`](../../../internals/crates/bookclerk_plugin_abi/db_stream.md#executestreamlimits)
- [`bookclerk_plugin_abi::ExtensibleConfig`](../../../internals/crates/bookclerk_plugin_abi/README.md#extensibleconfig)
- [`bookclerk_plugin_abi::GuestDatabase`](../../../internals/crates/bookclerk_plugin_abi/README.md#guestdatabase)
- [`bookclerk_plugin_abi::HealthOk`](../../../internals/crates/bookclerk_plugin_abi/README.md#healthok)
- [`bookclerk_plugin_abi::HostBindings`](../../../internals/crates/bookclerk_plugin_abi/README.md#hostbindings)
- [`bookclerk_plugin_abi::Invocation`](../../../internals/crates/bookclerk_plugin_abi/generated.md#invocation)
- [`bookclerk_plugin_abi::IsolationReq`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#isolationreq)
- [`bookclerk_plugin_abi::JobCheckpoint`](../../../internals/crates/bookclerk_plugin_abi/README.md#jobcheckpoint)
- [`bookclerk_plugin_abi::JobController`](../../../internals/crates/bookclerk_plugin_abi/README.md#jobcontroller)
- [`bookclerk_plugin_abi::JobInvocation`](../../../internals/crates/bookclerk_plugin_abi/README.md#jobinvocation)
- [`bookclerk_plugin_abi::JobInvocationLease`](../../../internals/crates/bookclerk_plugin_abi/README.md#jobinvocationlease)
- [`bookclerk_plugin_abi::JobOutcome`](../../../internals/crates/bookclerk_plugin_abi/README.md#joboutcome)
- [`bookclerk_plugin_abi::JobRunner`](../../../internals/crates/bookclerk_plugin_abi/README.md#jobrunner)
- [`bookclerk_plugin_abi::JobRunnerClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#jobrunnerclient)
- [`bookclerk_plugin_abi::KvDeleteOk`](../../../internals/crates/bookclerk_plugin_abi/generated.md#kvdeleteok)
- [`bookclerk_plugin_abi::KvGetOk`](../../../internals/crates/bookclerk_plugin_abi/generated.md#kvgetok)
- [`bookclerk_plugin_abi::KvListKey`](../../../internals/crates/bookclerk_plugin_abi/generated.md#kvlistkey)
- [`bookclerk_plugin_abi::KvListOk`](../../../internals/crates/bookclerk_plugin_abi/generated.md#kvlistok)
- [`bookclerk_plugin_abi::KvListOptions`](../../../internals/crates/bookclerk_plugin_abi/generated.md#kvlistoptions)
- [`bookclerk_plugin_abi::KvNamespace`](../../../internals/crates/bookclerk_plugin_abi/README.md#kvnamespace)
- [`bookclerk_plugin_abi::KvNamespaceClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#kvnamespaceclient)
- [`bookclerk_plugin_abi::KvPutOk`](../../../internals/crates/bookclerk_plugin_abi/generated.md#kvputok)
- [`bookclerk_plugin_abi::KvPutOptions`](../../../internals/crates/bookclerk_plugin_abi/generated.md#kvputoptions)
- [`bookclerk_plugin_abi::LimitExceeded`](../../../internals/crates/bookclerk_plugin_abi/error.md#limitexceeded)
- [`bookclerk_plugin_abi::ListOptions`](../../../internals/crates/bookclerk_plugin_abi/README.md#listoptions)
- [`bookclerk_plugin_abi::ListPage`](../../../internals/crates/bookclerk_plugin_abi/README.md#listpage)
- [`bookclerk_plugin_abi::MetadataChapter`](../../../internals/crates/bookclerk_plugin_abi/generated.md#metadatachapter)
- [`bookclerk_plugin_abi::MetadataChapters`](../../../internals/crates/bookclerk_plugin_abi/generated.md#metadatachapters)
- [`bookclerk_plugin_abi::MetadataChaptersParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#metadatachaptersparams)
- [`bookclerk_plugin_abi::MetadataLookupParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#metadatalookupparams)
- [`bookclerk_plugin_abi::MetadataProvider`](../../../internals/crates/bookclerk_plugin_abi/README.md#metadataprovider)
- [`bookclerk_plugin_abi::MetadataProviderClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#metadataproviderclient)
- [`bookclerk_plugin_abi::MetadataRecord`](../../../internals/crates/bookclerk_plugin_abi/generated.md#metadatarecord)
- [`bookclerk_plugin_abi::MetadataSearchParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#metadatasearchparams)
- [`bookclerk_plugin_abi::NamedKvBinding`](../../../internals/crates/bookclerk_plugin_abi/README.md#namedkvbinding)
- [`bookclerk_plugin_abi::NamedQueueBinding`](../../../internals/crates/bookclerk_plugin_abi/README.md#namedqueuebinding)
- [`bookclerk_plugin_abi::NeverCancel`](../../../internals/crates/bookclerk_plugin_abi/README.md#nevercancel)
- [`bookclerk_plugin_abi::ObjectInfo`](../../../internals/crates/bookclerk_plugin_abi/README.md#objectinfo)
- [`bookclerk_plugin_abi::ObjectMetadata`](../../../internals/crates/bookclerk_plugin_abi/README.md#objectmetadata)
- [`bookclerk_plugin_abi::Oidc`](../../../internals/crates/bookclerk_plugin_abi/README.md#oidc)
- [`bookclerk_plugin_abi::OidcClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#oidcclient)
- [`bookclerk_plugin_abi::OidcClientTemplate`](../../../internals/crates/bookclerk_plugin_abi/README.md#oidcclienttemplate)
- [`bookclerk_plugin_abi::OpenedEntrypoints`](../../../internals/crates/bookclerk_plugin_abi/README.md#openedentrypoints)
- [`bookclerk_plugin_abi::PluginCli`](../../../internals/crates/bookclerk_plugin_abi/README.md#plugincli)
- [`bookclerk_plugin_abi::PluginCliClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#plugincliclient)
- [`bookclerk_plugin_abi::PluginClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginclient)
- [`bookclerk_plugin_abi::PluginDescribe`](../../../internals/crates/bookclerk_plugin_abi/README.md#plugindescribe)
- [`bookclerk_plugin_abi::PluginError`](../../../internals/crates/bookclerk_plugin_abi/error.md#pluginerror)
- [`bookclerk_plugin_abi::PluginErrorCode`](../../../internals/crates/bookclerk_plugin_abi/error.md#pluginerrorcode)
- [`bookclerk_plugin_abi::PluginEvent`](../../../internals/crates/bookclerk_plugin_abi/generated.md#pluginevent)
- [`bookclerk_plugin_abi::PluginMigration`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginmigration)
- [`bookclerk_plugin_abi::PluginMigrationOp`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginmigrationop)
- [`bookclerk_plugin_abi::PluginServer`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginserver)
- [`bookclerk_plugin_abi::PluginWorker`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginworker)
- [`bookclerk_plugin_abi::ProgressSink`](../../../internals/crates/bookclerk_plugin_abi/README.md#progresssink)
- [`bookclerk_plugin_abi::PublishOk`](../../../internals/crates/bookclerk_plugin_abi/generated.md#publishok)
- [`bookclerk_plugin_abi::PutResult`](../../../internals/crates/bookclerk_plugin_abi/README.md#putresult)
- [`bookclerk_plugin_abi::QueryPage`](../../../internals/crates/bookclerk_plugin_abi/README.md#querypage)
- [`bookclerk_plugin_abi::QueueBatch`](../../../internals/crates/bookclerk_plugin_abi/generated.md#queuebatch)
- [`bookclerk_plugin_abi::QueueConsumer`](../../../internals/crates/bookclerk_plugin_abi/README.md#queueconsumer)
- [`bookclerk_plugin_abi::QueueConsumerClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#queueconsumerclient)
- [`bookclerk_plugin_abi::QueueMessage`](../../../internals/crates/bookclerk_plugin_abi/generated.md#queuemessage)
- [`bookclerk_plugin_abi::QueueProducer`](../../../internals/crates/bookclerk_plugin_abi/README.md#queueproducer)
- [`bookclerk_plugin_abi::QueueResult`](../../../internals/crates/bookclerk_plugin_abi/generated.md#queueresult)
- [`bookclerk_plugin_abi::QueueRetry`](../../../internals/crates/bookclerk_plugin_abi/generated.md#queueretry)
- [`bookclerk_plugin_abi::QueueSendMessage`](../../../internals/crates/bookclerk_plugin_abi/generated.md#queuesendmessage)
- [`bookclerk_plugin_abi::QueueSendOk`](../../../internals/crates/bookclerk_plugin_abi/generated.md#queuesendok)
- [`bookclerk_plugin_abi::ReadResult`](../../../internals/crates/bookclerk_plugin_abi/README.md#readresult)
- [`bookclerk_plugin_abi::RemoteLibrary`](../../../internals/crates/bookclerk_plugin_abi/README.md#remotelibrary)
- [`bookclerk_plugin_abi::RemoteLibraryClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#remotelibraryclient)
- [`bookclerk_plugin_abi::ScalarLimits`](../../../internals/crates/bookclerk_plugin_abi/README.md#scalarlimits)
- [`bookclerk_plugin_abi::Source`](../../../internals/crates/bookclerk_plugin_abi/README.md#source)
- [`bookclerk_plugin_abi::SourceClient`](../../../internals/crates/bookclerk_plugin_abi/README.md#sourceclient)
- [`bookclerk_plugin_abi::SourceServer`](../../../internals/crates/bookclerk_plugin_abi/README.md#sourceserver)
- [`bookclerk_plugin_abi::StatementResult`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#statementresult)
- [`bookclerk_plugin_abi::StreamCopyHandler`](../../../internals/crates/bookclerk_plugin_abi/README.md#streamcopyhandler)
- [`bookclerk_plugin_abi::StreamCopySpec`](../../../internals/crates/bookclerk_plugin_abi/README.md#streamcopyspec)
- [`bookclerk_plugin_abi::TypedDbStatement`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#typeddbstatement)
- [`bookclerk_plugin_abi::WriteOptions`](../../../internals/crates/bookclerk_plugin_abi/README.md#writeoptions)
- [`bookclerk_plugin_abi::FEATURE_SCALAR_LIMITS`](../../../internals/crates/bookclerk_plugin_abi/README.md#feature_scalar_limits)
- [`bookclerk_plugin_abi::FEATURE_STORAGE_COPY`](../../../internals/crates/bookclerk_plugin_abi/README.md#feature_storage_copy)
- [`bookclerk_plugin_abi::FEATURE_STREAMS`](../../../internals/crates/bookclerk_plugin_abi/README.md#feature_streams)
- [`bookclerk_plugin_abi::MAX_KV_KEY_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_kv_key_bytes)
- [`bookclerk_plugin_abi::MAX_KV_METADATA_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_kv_metadata_bytes)
- [`bookclerk_plugin_abi::MAX_KV_VALUE_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_kv_value_bytes)
- [`bookclerk_plugin_abi::MAX_LIST_PAGE`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_list_page)
- [`bookclerk_plugin_abi::MAX_PLUGIN_MIGRATION_OPS`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_plugin_migration_ops)
- [`bookclerk_plugin_abi::MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_plugin_migration_registration_bytes)
- [`bookclerk_plugin_abi::MAX_PLUGIN_MIGRATION_TOTAL_OPS`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_plugin_migration_total_ops)
- [`bookclerk_plugin_abi::MAX_SCALAR_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_scalar_bytes)
- [`bookclerk_plugin_abi::MAX_STREAM_WINDOW_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_stream_window_bytes)
- [`bookclerk_plugin_abi::PRODUCT_API_VERSION`](../../../internals/crates/bookclerk_plugin_abi/README.md#product_api_version)
- [`bookclerk_plugin_abi::SQL_CONTRACT_VERSION`](../../../internals/crates/bookclerk_plugin_abi/db_execute.md#sql_contract_version)

## Modules

- [`callback_tunnel`](callback_tunnel.md): Multiplexed byte tunnel for host-owned OAuth callback listeners.
- [`database_adapter`](database_adapter/README.md): Database adapter helpers for third-party database plugin authors.
- [`http`](http/README.md): Native guest HTTP through the workerd socket proxy.
- [`mux`](mux.md): Bidirectional multiplexed byte streams with per-stream credit windows.
- [`net`](net.md): Mediated TCP sockets for native guests (Workers `connect()` equivalent).
- [`protocol`](protocol.md): Shared role method names and typed method payloads.
- [`workerd`](workerd.md): Workerd guest bridge — same `crate::PluginWorker` contract as native.
- [`methods`](methods.md): Role capability method name constants (camelCase wire strings).

## Structs

### ChunkCaps

```rust
pub struct ChunkCaps {
    pub max_statements: u32,
    pub max_request_bytes: u32,
}
```

Caps [`DatabaseBinding::chunk`](#databasebindingchunk) splits a batch by when the session's
limits are not to be used.

**Fields**

- `max_statements`: Statements per request (`0` = no cap).
- `max_request_bytes`: Encoded request bytes per request (`0` = no cap).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### D1ExecResult

```rust
pub struct D1ExecResult {
    pub count: u32,
    pub duration: f64,
}
```

Cloudflare [`D1ExecResult`](https://developers.cloudflare.com/d1/worker-api/return-object/).

**Fields**

- `count`: Number of statements executed (one per non-empty `exec` line).
- `duration`: Total duration in milliseconds.

Implements: `Clone`, `Debug`, `PartialEq`

### D1Meta

```rust
pub struct D1Meta {
    pub duration: f64,
    pub changes: u64,
    pub last_row_id: i64,
    pub changed_db: bool,
    pub rows_read: u32,
    pub rows_written: u64,
}
```

Cloudflare [`D1Result::meta`](https://developers.cloudflare.com/d1/worker-api/return-object/) shape.

**Fields**

- `duration`: Engine-reported SQL duration in milliseconds.
- `changes`: Rows changed by the statement (`rowsAffected`).
- `last_row_id`: Last inserted row id when the adapter exposes it (else `0`).
- `changed_db`: `true` when `changes > 0`.
- `rows_read`: Rows returned to the guest for this statement.
- `rows_written`: Rows written (`changes` for DML).

Implements: `Clone`, `Debug`, `PartialEq`

### D1Result

```rust
pub struct D1Result {
    pub success: bool,
    pub results: Option<Vec<HashMap<String, DbValue>>>,
    pub meta: D1Meta,
}
```

Cloudflare [`D1Result`](https://developers.cloudflare.com/d1/worker-api/return-object/) projection.

**Fields**

- `success`: Always `true` on success; errors are returned as `Result::Err`.
- `results`: Row maps for selects; `None` when not applicable (DML).
- `meta`: Cloudflare-shaped timing and change metadata.

Implements: `Clone`, `Debug`, `PartialEq`

### DatabaseBinding

```rust
pub struct DatabaseBinding {
    /* private fields */
}
```

Host-mediated typed SQL surface for plugin guests.

Row-returning reads go through `GuestDatabase.executeStreamed` whenever
the session advertises a streamed budget (`limits()`, fetched once per
binding), so a result past the inline caps still arrives whole; sessions
that do not stream, or answer `unsupported`, are read inline.

#### DatabaseBinding::from_session

```rust
pub fn from_session(session: Arc<dyn GuestDatabase>) -> Self
```

Wraps a host-granted [`GuestDatabase`](../../../internals/crates/bookclerk_plugin_abi/README.md#guestdatabase).

#### DatabaseBinding::from_session_with

```rust
pub fn from_session_with(
    session: Arc<dyn GuestDatabase>,
    options: DatabaseBindingOptions,
) -> Self
```

Wraps a host-granted session with request-budget knobs.

#### DatabaseBinding::take_named_from_bindings

```rust
pub fn take_named_from_bindings(bindings: &mut Bindings, name: &str) -> Option<Self>
```

Takes the named plugin database binding `name` from the `open`
[`Bindings`](../../../internals/crates/bookclerk_plugin_abi/README.md#bindings).

Named bindings come from `plugin.toml` `[[databases]]` after operator
approval: each is an isolated plugin-owned database (full DML plus
idempotent `CREATE`/`DROP` `TABLE`/`INDEX` with `IF [NOT] EXISTS`),
physically separate from the Bookclerk library and from every other
plugin. Production invocations never inject the host library.

#### DatabaseBinding::limits

```rust
pub async fn limits(&self) -> Result<DbCapabilities>
```

The negotiated limits of the granted session (`DbCapabilities`).

Use it to size a batch or a read before sending: `max_request_bytes`,
`max_statements`, `max_result_bytes`, `max_result_rows` and the rest.

##### Errors

Returns `unsupported` when the host session predates
`GuestDatabase.limits`.

#### DatabaseBinding::chunk

```rust
pub async fn chunk(
    &self,
    statements: Vec<PreparedStatement>,
    caps: Option<ChunkCaps>,
) -> Result<Vec<Vec<PreparedStatement>>>
```

Splits `statements` into batches that each fit one request.

`for batch in db.chunk(stmts, None).await? { db.batch(batch, None).await?; }`.
Each batch is atomic; the loop is not. The caps come from the
session's limits (`max_statements`, and the tighter of the session's
and the binding's `max_request_bytes`) unless `caps` is given; zero
caps given explicitly mean one batch. A single statement larger than
the request cap is yielded alone, so [`Self::batch`](#databasebindingbatch) reports the
typed error. Batches are sized with the UUID-shaped retry id the
binding mints; a caller-chosen retry id that is longer leaves that
much less room.

##### Errors

Returns `unsupported` when neither the session's limits nor `caps`
are available, and `PluginError::invalid_params` when a statement
is missing terminal intent.

#### DatabaseBinding::prepare

```rust
pub fn prepare(&self, sql: impl Into<String>) -> PreparedStatement
```

Prepare one canonical-SQL statement (`?` placeholders).

#### DatabaseBinding::batch

```rust
pub async fn batch(
    &self,
    statements: Vec<PreparedStatement>,
    retry: Option<RetryToken>,
) -> Result<Vec<D1Result>>
```

Run prepared statements as one typed atomic batch.

##### Errors

Returns when a statement is missing terminal intent, the encoded
request exceeds `maxRequestBytes`, or `execute` fails.

#### DatabaseBinding::exec

```rust
pub async fn exec(
    &self,
    query: impl Into<String>,
    retry: Option<RetryToken>,
) -> Result<D1ExecResult>
```

Execute raw SQL without bind parameters (Cloudflare `D1Database::exec`).

Bookclerk accepts one or more newline-separated statements like D1.
Each non-empty line is executed as one prepared statement in a single
typed atomic batch.

##### Errors

Returns when the query is empty or `batch` fails.

#### DatabaseBinding::execute

```rust
pub async fn execute(
    &self,
    batch: Vec<TypedDbStatement>,
    retry: Option<RetryToken>,
) -> Result<ExecuteReply>
```

Internal typed-batch transport. Prefer [`Self::prepare`](#databasebindingprepare).

##### Errors

Returns when the batch is empty, the encoded request exceeds the
negotiated cap, or `execute` fails.

Implements: `Clone`

### DatabaseBindingOptions

```rust
pub struct DatabaseBindingOptions {
    pub max_request_bytes: u32,
    pub max_result_rows: u32,
    pub retry: Option<RetryToken>,
    pub deadline_unix_ms: u64,
}
```

Options for [`DatabaseBinding`](#databasebinding).

**Fields**

- `max_request_bytes`: Negotiated `maxRequestBytes` (`0` = unlimited).
- `max_result_rows`: Default `maxRows` for [`PreparedStatement::all`](#preparedstatementall) (`0` = host adapter cap).
- `retry`: Default retry token.
- `deadline_unix_ms`: Guest-visible deadline (unix ms).

Implements: `Clone`, `Debug`, `Default`

### PreparedStatement

```rust
pub struct PreparedStatement {
    /* private fields */
}
```

Cloudflare-style prepared statement.

#### PreparedStatement::bind

```rust
pub fn bind(self, values: Vec<DbValue>) -> Self
```

Replace bound parameters.

#### PreparedStatement::as_run

```rust
pub fn as_run(self) -> Self
```

Mark DML intent for [`DatabaseBinding::batch`](#databasebindingbatch).

#### PreparedStatement::as_first

```rust
pub fn as_first(self) -> Self
```

Mark `maxRows = 1` row intent for [`DatabaseBinding::batch`](#databasebindingbatch).

Preserves the caller SQL; the host enforces the one-row bound at execution.

#### PreparedStatement::as_all

```rust
pub fn as_all(self) -> Self
```

Mark row-returning intent for [`DatabaseBinding::batch`](#databasebindingbatch).

#### PreparedStatement::run

```rust
pub async fn run(self, retry: Option<RetryToken>) -> Result<D1Result>
```

Execute as Cloudflare `run()` (functionally equivalent to [`Self::all`](#preparedstatementall)).

##### Errors

Returns when `execute` fails.

#### PreparedStatement::first

```rust
pub async fn first(
    self,
    retry: Option<RetryToken>,
) -> Result<Option<Vec<(String, DbValue)>>>
```

First row as a name→value map, or `None` (Cloudflare `first()` without `colName`).

##### Errors

Returns when `execute` fails.

#### PreparedStatement::first_column

```rust
pub async fn first_column(
    self,
    col_name: &str,
    retry: Option<RetryToken>,
) -> Result<Option<DbValue>>
```

One column from the first row, or `None` when no row exists.

##### Errors

Returns when `execute` fails or the named column is absent from the row
(Cloudflare `first(colName)` throws `D1_ERROR` in that case).

#### PreparedStatement::raw

```rust
pub async fn raw(self, retry: Option<RetryToken>) -> Result<Vec<Vec<DbValue>>>
```

Positional cell values per row (Cloudflare `raw()`).

##### Errors

Returns when `execute` fails.

#### PreparedStatement::all

```rust
pub async fn all(self, retry: Option<RetryToken>) -> Result<D1Result>
```

Execute as a row-returning query. Returns a Cloudflare-shaped [`D1Result`](#d1result).

##### Errors

Returns when `execute` fails.

#### PreparedStatement::iterate

```rust
pub fn iterate(self, retry: Option<RetryToken>) -> RowStream
```

Rows one at a time from the same single-snapshot streamed read as
[`Self::all`](#preparedstatementall), without holding the whole result.

```rust
let mut rows = db.prepare("SELECT id, path FROM edition_files").iterate(None);
while let Some(row) = rows.next().await {
    let row = row?;
    // ...
}
```

Nothing is sent until the first [`RowStream::next`](#rowstreamnext). On a session
that streams, rows are decoded as their frames arrive and each is
handed out once, so memory stays near one frame however large
the result is. The read still fails closed (`limit_exceeded`) past
the session's streamed budget, after the rows that fit were handed
out. On a session that does not stream, or answers `unsupported`
before any row arrived, the rows come from one inline read like
[`Self::all`](#preparedstatementall). Dropping the stream stops reading it.

Implements: `Clone`

### RetryToken

```rust
pub struct RetryToken {
    pub operation_id: String,
    pub request_hash: String,
}
```

Explicit retry identity: reuse both `operationId` and `requestHash`.

**Fields**

- `operation_id`: Caller-chosen idempotency key.
- `request_hash`: Canonical Cap'n request hash stamped by the host.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### RowStream

```rust
pub struct RowStream {
    /* private fields */
}
```

Rows of one read, handed out one at a time
([`PreparedStatement::iterate`](#preparedstatementiterate)).

Each row is a column name → value map, the shape [`D1Result::results`](#d1result)
holds. The stream ends (`None`) after the last row, or after the first
error it yields.

#### RowStream::next

```rust
pub async fn next(&mut self) -> Option<Result<HashMap<String, DbValue>>>
```

The next row, `None` once the read is over.

##### Errors

Yields (once, then `None`) the error the read fails with: a typed
`limit_exceeded` past the session's budget, the adapter's error, an
`unavailable` when the stream breaks off, or `invalid_params` when
the reply does not answer the request or carries more rows than the
statement's `max_rows`.

Implements: `Debug`

### FetchWorkDir

```rust
pub struct FetchWorkDir {
    /* private fields */
}
```

An open fetch work directory received from the host for one `fetchTitle` call.

Holds the descriptor open for its lifetime so `/proc/self/fd/N` (or
`/dev/fd/N`) stays valid for the duration of the call. When the side channel
is not armed, falls back to the `cache_dir` field on
[`FetchTitleParams`](../../../internals/crates/bookclerk_plugin_abi/generated.md#fetchtitleparams) from the RPC params (absolute path string the host
already prepared inside the guest jail).

#### FetchWorkDir::open

```rust
pub fn open(params: &FetchTitleParams) -> Result<Self>
```

Resolves the directory a `fetchTitle` call should write downloaded audio into.

Uses the `cache_dir` field on `params` (hosts pass a jail-granted
directory under the guest `TMPDIR`). When [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env) is
set, receives one SCM_RIGHTS directory FD instead — hosts do not set
this env; do not arm it without a matching host send.

##### Arguments

* `params` - Host `fetchTitle` params; `cache_dir` is the work directory
  unless the unused FD side channel is armed.

##### Returns

A [`FetchWorkDir`](#fetchworkdir) whose [`Self::path`](#fetchworkdirpath) is valid for the rest of the RPC.

##### Errors

Returns [`SdkError`](#sdkerror) when the side channel env is set but
[`recv_passed_fd`](#recv_passed_fd) fails, or when FD → path mapping fails on this OS.

#### FetchWorkDir::path

```rust
pub fn path(&self) -> &Path
```

Absolute filesystem path of the fetch work directory.

On Unix with a passed FD this is typically `/proc/self/fd/N` (Linux) or
`/dev/fd/N` (macOS/BSD). Keep `self` alive while writing under this path.

Implements: `Deref`

### UploadFile

```rust
pub struct UploadFile {
    /* private fields */
}
```

An open upload file received from the host for one `putFile` call.

Same FD-lifetime rules as [`FetchWorkDir`]: keep the value alive while
reading the path so the `/proc/self/fd/N` symlink remains valid.

#### UploadFile::open

```rust
pub fn open(local_path: Option<&str>) -> Result<Self>
```

Resolves the local file path for a `putFile` upload.

Uses `local_path` from the RPC params. When [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env) is
set, receives one SCM_RIGHTS file FD instead — hosts do not set this
env (destinations stream bytes). Do not arm it without a matching send.

##### Arguments

* `local_path` - Optional absolute path from host params; ignored when
  the FD channel is active. Must be `Some` and non-empty when the
  channel is inactive.

##### Returns

An [`UploadFile`](#uploadfile) whose [`Self::path`](#uploadfilepath) points at the bytes to upload.

##### Errors

Returns [`SdkError`](#sdkerror) when neither an FD nor a usable `local_path` is
available, or when receiving/mapping the FD fails.

#### UploadFile::path

```rust
pub fn path(&self) -> &Path
```

Absolute filesystem path of the file the host wants uploaded.

Keep `self` alive for the duration of the read so a passed FD path stays
valid.

Implements: `Deref`

## Enums

### SdkError

```rust
pub enum SdkError {
    Message(String),
    Io(Error),
    Json(Error),
}
```

Errors from the guest stdio loop and SDK helpers (not plugin business logic).

Prefer mapping operator-visible failures into
[`bookclerk_plugin_abi::PluginError`](../../../internals/crates/bookclerk_plugin_abi/error.md#pluginerror) inside [`crate::PluginWorker`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginworker)
methods. Use [`SdkError`](#sdkerror) for framing, JSON, and filesystem problems in the
SDK itself.

**Variants**

- `Message`: Operator-facing or diagnostic text with no structured ABI error code.
  
  Built via [`SdkError::message`](#sdkerrormessage). Typical sources: oversize RPC frames,
  missing side-channel descriptors, or authoring-tool validation failures.
- `Io`: Underlying OS I/O failure (stdio, files, sockets) while running SDK helpers.
- `Json`: JSON serialize/deserialize failure for Workers RPC request or response lines.

#### SdkError::message

```rust
pub fn message(msg: impl Into<String>) -> Self
```

Builds a [`SdkError::Message`](#sdkerror) from any displayable string.

##### Arguments

* `msg` - Human-readable explanation (no secret values).

##### Returns

An [`SdkError`](#sdkerror) suitable for helper APIs returning [`Result`](#result).

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<SdkError>`

## Functions

### db_value_from_sea

```rust
pub fn db_value_from_sea(v: &Value) -> Result<DbValue, String>
```

Fallible SeaORM → [`bookclerk_plugin_abi::DbValue`](../../../internals/crates/bookclerk_plugin_abi/db_value.md#dbvalue) bridge.

#### Errors

Returns a static reason when the SeaORM value is outside the universal
domain, or TEXT contains U+0000.

### proxy_rows_from_typed

```rust
pub fn proxy_rows_from_typed(stmt: &StatementResult) -> Result<Vec<ProxyRow>, String>
```

Builds SeaORM proxy rows from a typed statement result.

#### Errors

Returns when the result is not positional.

### chunk_indexes

```rust
pub fn chunk_indexes(
    typed: &[TypedDbStatement],
    max_statements: u32,
    max_request_bytes: u32,
) -> Result<Vec<Vec<usize>>>
```

Greedy split of `typed` into index runs that each fit one request.

Sizes are estimated per statement against one probe statement and each
run is verified by one exact encode; a run the estimate let through is
halved until it fits, and a lone statement over the cap is yielded on
its own. `0` lifts a cap.

#### Errors

Returns when a statement cannot be encoded (U+0000 in SQL or TEXT).

### execute_reply_to_d1_results

```rust
pub fn execute_reply_to_d1_results(reply: &ExecuteReply) -> Vec<D1Result>
```

Maps a typed execute reply to Cloudflare [`D1Result`](#d1result) per statement.

### statement_result_to_d1_result

```rust
pub fn statement_result_to_d1_result(
    stmt: &StatementResult,
    timing: &DbTiming,
) -> D1Result
```

Maps one typed statement result to Cloudflare [`D1Result`](#d1result).

### fetch_work_dir

```rust
pub fn fetch_work_dir(params: &FetchTitleParams) -> Result<FetchWorkDir>
```

Resolves the directory a `fetchTitle` call should write into.

Thin wrapper around [`FetchWorkDir::open`](#fetchworkdiropen).

#### Arguments

* `params` - Host `fetchTitle` params (see [`FetchWorkDir::open`](#fetchworkdiropen)).

#### Returns

Open work directory for this RPC.

#### Errors

Propagates [`FetchWorkDir::open`](#fetchworkdiropen) failures.

### upload_file_path

```rust
pub fn upload_file_path(local_path: Option<&str>) -> Result<UploadFile>
```

Resolves the local file path for a `putFile` call.

Thin wrapper around [`UploadFile::open`](#uploadfileopen).

#### Arguments

* `local_path` - Optional path from host params (see [`UploadFile::open`](#uploadfileopen)).

#### Returns

Open upload file handle for this RPC.

#### Errors

Propagates [`UploadFile::open`](#uploadfileopen) failures.

### decode_json

```rust
pub fn decode_json<T: DeserializeOwned>(json: &str) -> Result<T, PluginError>
```

Deserialize a role-method JSON argument.

Empty input is treated as `{}`.

#### Errors

Returns `PluginError::invalid_params` when the payload is not valid JSON for `T`.

### encode_json

```rust
pub fn encode_json<T: Serialize>(value: T) -> Result<String, PluginError>
```

Serialize a role-method JSON result.

#### Errors

Returns `PluginError::invalid_params` when serialization fails, or
`PluginError::payload_too_large` when the encoded size exceeds
[`MAX_SCALAR_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_scalar_bytes).

### encode_atomic_result

```rust
pub fn encode_atomic_result<T: Serialize>(value: T) -> Result<String, PluginError>
```

Encodes an atomic result; an oversized scalar after commit is `unavailable`.

#### Errors

Returns `PluginError::payload_too_large` mapped to
`PluginError::unavailable` so the host retries the same `operationId`.

### page_rows

```rust
pub fn page_rows<T: Serialize>(
    rows: &[T],
    cursor: &str,
    limit: u32,
) -> Result<QueryPage, PluginError>
```

Page a JSON array of rows. `limit == 0` means [`MAX_LIST_PAGE`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_list_page).

#### Errors

Returns `PluginError::invalid_cursor` for a non-numeric cursor, or
`PluginError::payload_too_large` when the page exceeds [`MAX_SCALAR_BYTES`](../../../internals/crates/bookclerk_plugin_abi/README.md#max_scalar_bytes).

### manifest_capabilities

```rust
pub fn manifest_capabilities(
    manifest_toml: &str,
) -> Result<PluginCapabilities, PluginError>
```

Parses `plugin.toml` source and returns the [`PluginCapabilities`](../../../internals/crates/bookclerk_plugin_abi/generated.md#plugincapabilities) a guest
must advertise from `describe()`.

Keeping the manifest as the single source of truth means the host's
widening check (manifest vs describe vs operator grant) cannot drift: the
guest embeds the same file the host installed.

#### Arguments

* `manifest_toml` - Contents of the plugin's `plugin.toml`, typically
  `include_str!("../plugin.toml")`.

#### Returns

The typed capability declaration in manifest order.

#### Errors

Returns `PluginErrorCode::Internal` when the manifest does not parse or
fails validation — a build defect, not a runtime condition.

#### Examples

```rust
use bookclerk_plugin_sdk::manifest_capabilities;

let caps = manifest_capabilities(r#"
api_version = 3
id = "echo"
runtime = "native"
command = "./echo"
entrypoints = ["cli"]

[capabilities.network]
mode = "deny"
"#).unwrap();
assert_eq!(caps.entrypoints.len(), 1);
```

### fd_proc_path

```rust
pub fn fd_proc_path(fd: i32) -> PathBuf
```

Builds the OS path that refers to an open descriptor by number.

Linux uses `/proc/self/fd/{fd}`; macOS and other Unix use `/dev/fd/{fd}`.
The path is only valid while the descriptor remains open in this process.

#### Arguments

* `fd` - Open raw file descriptor previously received from the host.

#### Returns

PathBuf that can be passed to std / tokio filesystem APIs for as long as
`fd` stays open.

### recv_passed_fd

```rust
pub fn recv_passed_fd() -> Result<i32>
```

Receives one file descriptor from the host side channel (SCM_RIGHTS).

Blocks until the host sends a control message on [`PLUGIN_FD_CHANNEL`](#plugin_fd_channel).
Hosts do not send; prefer path strings unless you are implementing a
native shortcut that actually `sendmsg`s.

#### Returns

Raw OS file descriptor owned by the caller. Prefer wrapping it immediately
(see [`crate::FetchWorkDir`](#fetchworkdir) / [`crate::UploadFile`](#uploadfile)) so Drop closes it.

#### Errors

Returns [`SdkError`](#sdkerror) when [`PLUGIN_FD_CHANNEL_ENV`](#plugin_fd_channel_env) is unset or not equal to
[`PLUGIN_FD_CHANNEL`](#plugin_fd_channel), when `recvmsg` fails, or when the platform does not
support the side channel (non-Unix).

### serve

```rust
pub async fn serve(plugin: impl PluginWorker) -> Result<()>
```

Serves a [`PluginWorker`](../../../internals/crates/bookclerk_plugin_abi/README.md#pluginworker) on stdin/stdout (Cap'n Proto RPC).

Must run on a current-thread tokio runtime inside a `LocalSet` (this helper
creates the `LocalSet`). Abort is capability drop / stream cancel.

#### Errors

Returns [`SdkError`](#sdkerror) when the vat fails.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, SdkError>;
```

Result alias for guest helpers that fail with [`SdkError`](#sdkerror).

## Constants

### PLUGIN_FD_CHANNEL

```rust
pub const PLUGIN_FD_CHANNEL: i32 = 3;
```

Fixed descriptor number the host leaves open for the plugin side channel.

Always `3` (after stdin/stdout/stderr). Guests must not close or reuse this
FD for other purposes while the channel is active.

### PLUGIN_FD_CHANNEL_ENV

```rust
pub const PLUGIN_FD_CHANNEL_ENV: &str = "BOOKCLERK_PLUGIN_FD_CHANNEL";
```

Environment variable the host *would* set to the decimal form of [`PLUGIN_FD_CHANNEL`](#plugin_fd_channel).

Hosts do **not** set this. Presence still arms [`crate::FetchWorkDir::open`](#fetchworkdiropen)
/ [`crate::UploadFile::open`](#uploadfileopen) to call [`recv_passed_fd`](#recv_passed_fd) instead of path
strings — only set it when the host is actually sending an FD.
