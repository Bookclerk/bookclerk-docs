# bookclerk_plugin_abi::plugin_capnp::adapter_database_session

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`CapabilitiesParams`](#capabilitiesparams) | type alias |  |
| [`CapabilitiesResults`](#capabilitiesresults) | type alias |  |
| [`ExecuteParams`](#executeparams) | type alias |  |
| [`ExecuteResults`](#executeresults) | type alias |  |
| [`CloseParams`](#closeparams) | type alias |  |
| [`CloseResults`](#closeresults) | type alias |  |
| [`BootstrapParams`](#bootstrapparams) | type alias |  |
| [`BootstrapResults`](#bootstrapresults) | type alias |  |
| [`ExportIdentityParams`](#exportidentityparams) | type alias |  |
| [`ExportIdentityResults`](#exportidentityresults) | type alias |  |
| [`ImportIdentityParams`](#importidentityparams) | type alias |  |
| [`ImportIdentityResults`](#importidentityresults) | type alias |  |
| [`ListUserRelationsParams`](#listuserrelationsparams) | type alias |  |
| [`ListUserRelationsResults`](#listuserrelationsresults) | type alias |  |
| [`PrepareUnitRestoreParams`](#prepareunitrestoreparams) | type alias |  |
| [`PrepareUnitRestoreResults`](#prepareunitrestoreresults) | type alias |  |
| [`DropUserRelationsParams`](#dropuserrelationsparams) | type alias |  |
| [`DropUserRelationsResults`](#dropuserrelationsresults) | type alias |  |
| [`AssertRestoreConstraintsParams`](#assertrestoreconstraintsparams) | type alias |  |
| [`AssertRestoreConstraintsResults`](#assertrestoreconstraintsresults) | type alias |  |

## Modules

- [`capabilities_params`](adapter_database_session/capabilities_params.md)
- [`capabilities_results`](adapter_database_session/capabilities_results.md)
- [`execute_params`](adapter_database_session/execute_params.md)
- [`execute_results`](adapter_database_session/execute_results.md)
- [`close_params`](adapter_database_session/close_params.md)
- [`close_results`](adapter_database_session/close_results.md)
- [`bootstrap_params`](adapter_database_session/bootstrap_params.md)
- [`bootstrap_results`](adapter_database_session/bootstrap_results.md)
- [`export_identity_params`](adapter_database_session/export_identity_params.md)
- [`export_identity_results`](adapter_database_session/export_identity_results.md)
- [`import_identity_params`](adapter_database_session/import_identity_params.md)
- [`import_identity_results`](adapter_database_session/import_identity_results.md)
- [`list_user_relations_params`](adapter_database_session/list_user_relations_params.md)
- [`list_user_relations_results`](adapter_database_session/list_user_relations_results.md)
- [`prepare_unit_restore_params`](adapter_database_session/prepare_unit_restore_params.md)
- [`prepare_unit_restore_results`](adapter_database_session/prepare_unit_restore_results.md)
- [`drop_user_relations_params`](adapter_database_session/drop_user_relations_params.md)
- [`drop_user_relations_results`](adapter_database_session/drop_user_relations_results.md)
- [`assert_restore_constraints_params`](adapter_database_session/assert_restore_constraints_params.md)
- [`assert_restore_constraints_results`](adapter_database_session/assert_restore_constraints_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn capabilities(
        self: Rc<Self>,
        _: CapabilitiesParams,
        _: CapabilitiesResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn execute(
        self: Rc<Self>,
        _: ExecuteParams,
        _: ExecuteResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn close(
        self: Rc<Self>,
        _: CloseParams,
        _: CloseResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn bootstrap(
        self: Rc<Self>,
        _: BootstrapParams,
        _: BootstrapResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn export_identity(
        self: Rc<Self>,
        _: ExportIdentityParams,
        _: ExportIdentityResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn import_identity(
        self: Rc<Self>,
        _: ImportIdentityParams,
        _: ImportIdentityResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn list_user_relations(
        self: Rc<Self>,
        _: ListUserRelationsParams,
        _: ListUserRelationsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn prepare_unit_restore(
        self: Rc<Self>,
        _: PrepareUnitRestoreParams,
        _: PrepareUnitRestoreResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn drop_user_relations(
        self: Rc<Self>,
        _: DropUserRelationsParams,
        _: DropUserRelationsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn assert_restore_constraints(
        self: Rc<Self>,
        _: AssertRestoreConstraintsParams,
        _: AssertRestoreConstraintsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::capabilities

```rust
fn capabilities(
    self: Rc<Self>,
    _: CapabilitiesParams,
    _: CapabilitiesResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::execute

```rust
fn execute(
    self: Rc<Self>,
    _: ExecuteParams,
    _: ExecuteResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::close

```rust
fn close(
    self: Rc<Self>,
    _: CloseParams,
    _: CloseResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::bootstrap

```rust
fn bootstrap(
    self: Rc<Self>,
    _: BootstrapParams,
    _: BootstrapResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::export_identity

```rust
fn export_identity(
    self: Rc<Self>,
    _: ExportIdentityParams,
    _: ExportIdentityResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::import_identity

```rust
fn import_identity(
    self: Rc<Self>,
    _: ImportIdentityParams,
    _: ImportIdentityResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::list_user_relations

```rust
fn list_user_relations(
    self: Rc<Self>,
    _: ListUserRelationsParams,
    _: ListUserRelationsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::prepare_unit_restore

```rust
fn prepare_unit_restore(
    self: Rc<Self>,
    _: PrepareUnitRestoreParams,
    _: PrepareUnitRestoreResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::drop_user_relations

```rust
fn drop_user_relations(
    self: Rc<Self>,
    _: DropUserRelationsParams,
    _: DropUserRelationsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::assert_restore_constraints

```rust
fn assert_restore_constraints(
    self: Rc<Self>,
    _: AssertRestoreConstraintsParams,
    _: AssertRestoreConstraintsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::capabilities_request

```rust
pub fn capabilities_request(&self) -> Request<Owned, Owned>
```

#### Client::execute_request

```rust
pub fn execute_request(&self) -> Request<Owned, Owned>
```

#### Client::close_request

```rust
pub fn close_request(&self) -> Request<Owned, Owned>
```

#### Client::bootstrap_request

```rust
pub fn bootstrap_request(&self) -> Request<Owned, Owned>
```

#### Client::export_identity_request

```rust
pub fn export_identity_request(&self) -> Request<Owned, Owned>
```

#### Client::import_identity_request

```rust
pub fn import_identity_request(&self) -> Request<Owned, Owned>
```

#### Client::list_user_relations_request

```rust
pub fn list_user_relations_request(&self) -> Request<Owned, Owned>
```

#### Client::prepare_unit_restore_request

```rust
pub fn prepare_unit_restore_request(&self) -> Request<Owned, Owned>
```

#### Client::drop_user_relations_request

```rust
pub fn drop_user_relations_request(&self) -> Request<Owned, Owned>
```

#### Client::assert_restore_constraints_request

```rust
pub fn assert_restore_constraints_request(&self) -> Request<Owned, Owned>
```

Implements: `Clone`, `FromClientHook`, `FromPointerBuilder<'a>`, `FromPointerReader<'a>`, `FromServer<_S>`, `HasTypeId`, `SetterInput<Owned>`

### Owned

```rust
pub struct Owned(/* private */);
```

Implements: `Clone`, `Copy`, `Introspect`, `Owned`, `Pipelined`, `SetterInput<Owned>`

### ServerDispatch

```rust
pub struct ServerDispatch<_T> {
    pub server: Rc<_T>,
}
```

#### ServerDispatch::dispatch_call_internal

```rust
pub fn dispatch_call_internal(
    this: Rc<_T>,
    method_id: u16,
    params: Params<Owned>,
    results: Results<Owned>,
) -> DispatchCallResult
```

Implements: `Clone`, `Deref`, `Server`

## Type aliases

### CapabilitiesParams

```rust
pub type CapabilitiesParams = Params<Owned>;
```

### CapabilitiesResults

```rust
pub type CapabilitiesResults = Results<Owned>;
```

### ExecuteParams

```rust
pub type ExecuteParams = Params<Owned>;
```

### ExecuteResults

```rust
pub type ExecuteResults = Results<Owned>;
```

### CloseParams

```rust
pub type CloseParams = Params<Owned>;
```

### CloseResults

```rust
pub type CloseResults = Results<Owned>;
```

### BootstrapParams

```rust
pub type BootstrapParams = Params<Owned>;
```

### BootstrapResults

```rust
pub type BootstrapResults = Results<Owned>;
```

### ExportIdentityParams

```rust
pub type ExportIdentityParams = Params<Owned>;
```

### ExportIdentityResults

```rust
pub type ExportIdentityResults = Results<Owned>;
```

### ImportIdentityParams

```rust
pub type ImportIdentityParams = Params<Owned>;
```

### ImportIdentityResults

```rust
pub type ImportIdentityResults = Results<Owned>;
```

### ListUserRelationsParams

```rust
pub type ListUserRelationsParams = Params<Owned>;
```

### ListUserRelationsResults

```rust
pub type ListUserRelationsResults = Results<Owned>;
```

### PrepareUnitRestoreParams

```rust
pub type PrepareUnitRestoreParams = Params<Owned>;
```

### PrepareUnitRestoreResults

```rust
pub type PrepareUnitRestoreResults = Results<Owned>;
```

### DropUserRelationsParams

```rust
pub type DropUserRelationsParams = Params<Owned>;
```

### DropUserRelationsResults

```rust
pub type DropUserRelationsResults = Results<Owned>;
```

### AssertRestoreConstraintsParams

```rust
pub type AssertRestoreConstraintsParams = Params<Owned>;
```

### AssertRestoreConstraintsResults

```rust
pub type AssertRestoreConstraintsResults = Results<Owned>;
```
