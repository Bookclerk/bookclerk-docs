# bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`ExecuteParams`](#executeparams) | type alias |  |
| [`ExecuteResults`](#executeresults) | type alias |  |
| [`CommitParams`](#commitparams) | type alias |  |
| [`CommitResults`](#commitresults) | type alias |  |
| [`RollbackParams`](#rollbackparams) | type alias |  |
| [`RollbackResults`](#rollbackresults) | type alias |  |
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

- [`execute_params`](adapter_transaction/execute_params.md)
- [`execute_results`](adapter_transaction/execute_results.md)
- [`commit_params`](adapter_transaction/commit_params.md)
- [`commit_results`](adapter_transaction/commit_results.md)
- [`rollback_params`](adapter_transaction/rollback_params.md)
- [`rollback_results`](adapter_transaction/rollback_results.md)
- [`export_identity_params`](adapter_transaction/export_identity_params.md)
- [`export_identity_results`](adapter_transaction/export_identity_results.md)
- [`import_identity_params`](adapter_transaction/import_identity_params.md)
- [`import_identity_results`](adapter_transaction/import_identity_results.md)
- [`list_user_relations_params`](adapter_transaction/list_user_relations_params.md)
- [`list_user_relations_results`](adapter_transaction/list_user_relations_results.md)
- [`prepare_unit_restore_params`](adapter_transaction/prepare_unit_restore_params.md)
- [`prepare_unit_restore_results`](adapter_transaction/prepare_unit_restore_results.md)
- [`drop_user_relations_params`](adapter_transaction/drop_user_relations_params.md)
- [`drop_user_relations_results`](adapter_transaction/drop_user_relations_results.md)
- [`assert_restore_constraints_params`](adapter_transaction/assert_restore_constraints_params.md)
- [`assert_restore_constraints_results`](adapter_transaction/assert_restore_constraints_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn execute(
        self: Rc<Self>,
        _: ExecuteParams,
        _: ExecuteResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn commit(
        self: Rc<Self>,
        _: CommitParams,
        _: CommitResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn rollback(
        self: Rc<Self>,
        _: RollbackParams,
        _: RollbackResults,
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

#### Server::execute

```rust
fn execute(
    self: Rc<Self>,
    _: ExecuteParams,
    _: ExecuteResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::commit

```rust
fn commit(
    self: Rc<Self>,
    _: CommitParams,
    _: CommitResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::rollback

```rust
fn rollback(
    self: Rc<Self>,
    _: RollbackParams,
    _: RollbackResults,
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

#### Client::execute_request

```rust
pub fn execute_request(&self) -> Request<Owned, Owned>
```

#### Client::commit_request

```rust
pub fn commit_request(&self) -> Request<Owned, Owned>
```

#### Client::rollback_request

```rust
pub fn rollback_request(&self) -> Request<Owned, Owned>
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

### ExecuteParams

```rust
pub type ExecuteParams = Params<Owned>;
```

### ExecuteResults

```rust
pub type ExecuteResults = Results<Owned>;
```

### CommitParams

```rust
pub type CommitParams = Params<Owned>;
```

### CommitResults

```rust
pub type CommitResults = Results<Owned>;
```

### RollbackParams

```rust
pub type RollbackParams = Params<Owned>;
```

### RollbackResults

```rust
pub type RollbackResults = Results<Owned>;
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
