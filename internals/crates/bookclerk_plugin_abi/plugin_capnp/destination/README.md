# bookclerk_plugin_abi::plugin_capnp::destination

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`HeadParams`](#headparams) | type alias |  |
| [`HeadResults`](#headresults) | type alias |  |
| [`ListParams`](#listparams) | type alias |  |
| [`ListResults`](#listresults) | type alias |  |
| [`GetParams`](#getparams) | type alias |  |
| [`GetResults`](#getresults) | type alias |  |
| [`PutParams`](#putparams) | type alias |  |
| [`PutResults`](#putresults) | type alias |  |
| [`CopyParams`](#copyparams) | type alias |  |
| [`CopyResults`](#copyresults) | type alias |  |
| [`DeleteParams`](#deleteparams) | type alias |  |
| [`DeleteResults`](#deleteresults) | type alias |  |
| [`CommitParams`](#commitparams) | type alias |  |
| [`CommitResults`](#commitresults) | type alias |  |
| [`AbortStageParams`](#abortstageparams) | type alias |  |
| [`AbortStageResults`](#abortstageresults) | type alias |  |
| [`MoveParams`](#moveparams) | type alias |  |
| [`MoveResults`](#moveresults) | type alias |  |
| [`MoveManyParams`](#movemanyparams) | type alias |  |
| [`MoveManyResults`](#movemanyresults) | type alias |  |
| [`DeleteManyParams`](#deletemanyparams) | type alias |  |
| [`DeleteManyResults`](#deletemanyresults) | type alias |  |
| [`HeadManyParams`](#headmanyparams) | type alias |  |
| [`HeadManyResults`](#headmanyresults) | type alias |  |
| [`ChangesParams`](#changesparams) | type alias |  |
| [`ChangesResults`](#changesresults) | type alias |  |

## Modules

- [`head_params`](head_params.md)
- [`head_results`](head_results.md)
- [`list_params`](list_params.md)
- [`list_results`](list_results.md)
- [`get_params`](get_params.md)
- [`get_results`](get_results.md)
- [`put_params`](put_params.md)
- [`put_results`](put_results.md)
- [`copy_params`](copy_params.md)
- [`copy_results`](copy_results.md)
- [`delete_params`](delete_params.md)
- [`delete_results`](delete_results.md)
- [`commit_params`](commit_params.md)
- [`commit_results`](commit_results.md)
- [`abort_stage_params`](abort_stage_params.md)
- [`abort_stage_results`](abort_stage_results.md)
- [`move_params`](move_params.md)
- [`move_results`](move_results.md)
- [`move_many_params`](move_many_params.md)
- [`move_many_results`](move_many_results.md)
- [`delete_many_params`](delete_many_params.md)
- [`delete_many_results`](delete_many_results.md)
- [`head_many_params`](head_many_params.md)
- [`head_many_results`](head_many_results.md)
- [`changes_params`](changes_params.md)
- [`changes_results`](changes_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn head(
        self: Rc<Self>,
        _: HeadParams,
        _: HeadResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn list(
        self: Rc<Self>,
        _: ListParams,
        _: ListResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn get(
        self: Rc<Self>,
        _: GetParams,
        _: GetResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn put(
        self: Rc<Self>,
        _: PutParams,
        _: PutResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn copy(
        self: Rc<Self>,
        _: CopyParams,
        _: CopyResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn delete(
        self: Rc<Self>,
        _: DeleteParams,
        _: DeleteResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn commit(
        self: Rc<Self>,
        _: CommitParams,
        _: CommitResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn abort_stage(
        self: Rc<Self>,
        _: AbortStageParams,
        _: AbortStageResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn move_(
        self: Rc<Self>,
        _: MoveParams,
        _: MoveResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn move_many(
        self: Rc<Self>,
        _: MoveManyParams,
        _: MoveManyResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn delete_many(
        self: Rc<Self>,
        _: DeleteManyParams,
        _: DeleteManyResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn head_many(
        self: Rc<Self>,
        _: HeadManyParams,
        _: HeadManyResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn changes(
        self: Rc<Self>,
        _: ChangesParams,
        _: ChangesResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::head

```rust
fn head(
    self: Rc<Self>,
    _: HeadParams,
    _: HeadResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::list

```rust
fn list(
    self: Rc<Self>,
    _: ListParams,
    _: ListResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::get

```rust
fn get(
    self: Rc<Self>,
    _: GetParams,
    _: GetResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::put

```rust
fn put(
    self: Rc<Self>,
    _: PutParams,
    _: PutResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::copy

```rust
fn copy(
    self: Rc<Self>,
    _: CopyParams,
    _: CopyResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::delete

```rust
fn delete(
    self: Rc<Self>,
    _: DeleteParams,
    _: DeleteResults,
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

#### Server::abort_stage

```rust
fn abort_stage(
    self: Rc<Self>,
    _: AbortStageParams,
    _: AbortStageResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::move_

```rust
fn move_(
    self: Rc<Self>,
    _: MoveParams,
    _: MoveResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::move_many

```rust
fn move_many(
    self: Rc<Self>,
    _: MoveManyParams,
    _: MoveManyResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::delete_many

```rust
fn delete_many(
    self: Rc<Self>,
    _: DeleteManyParams,
    _: DeleteManyResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::head_many

```rust
fn head_many(
    self: Rc<Self>,
    _: HeadManyParams,
    _: HeadManyResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::changes

```rust
fn changes(
    self: Rc<Self>,
    _: ChangesParams,
    _: ChangesResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

Implemented by: `DestinationServer`

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::head_request

```rust
pub fn head_request(&self) -> Request<Owned, Owned>
```

#### Client::list_request

```rust
pub fn list_request(&self) -> Request<Owned, Owned>
```

#### Client::get_request

```rust
pub fn get_request(&self) -> Request<Owned, Owned>
```

#### Client::put_request

```rust
pub fn put_request(&self) -> Request<Owned, Owned>
```

#### Client::copy_request

```rust
pub fn copy_request(&self) -> Request<Owned, Owned>
```

#### Client::delete_request

```rust
pub fn delete_request(&self) -> Request<Owned, Owned>
```

#### Client::commit_request

```rust
pub fn commit_request(&self) -> Request<Owned, Owned>
```

#### Client::abort_stage_request

```rust
pub fn abort_stage_request(&self) -> Request<Owned, Owned>
```

#### Client::move_request

```rust
pub fn move_request(&self) -> Request<Owned, Owned>
```

#### Client::move_many_request

```rust
pub fn move_many_request(&self) -> Request<Owned, Owned>
```

#### Client::delete_many_request

```rust
pub fn delete_many_request(&self) -> Request<Owned, Owned>
```

#### Client::head_many_request

```rust
pub fn head_many_request(&self) -> Request<Owned, Owned>
```

#### Client::changes_request

```rust
pub fn changes_request(&self) -> Request<Owned, Owned>
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

### HeadParams

```rust
pub type HeadParams = Params<Owned>;
```

### HeadResults

```rust
pub type HeadResults = Results<Owned>;
```

### ListParams

```rust
pub type ListParams = Params<Owned>;
```

### ListResults

```rust
pub type ListResults = Results<Owned>;
```

### GetParams

```rust
pub type GetParams = Params<Owned>;
```

### GetResults

```rust
pub type GetResults = Results<Owned>;
```

### PutParams

```rust
pub type PutParams = Params<Owned>;
```

### PutResults

```rust
pub type PutResults = Results<Owned>;
```

### CopyParams

```rust
pub type CopyParams = Params<Owned>;
```

### CopyResults

```rust
pub type CopyResults = Results<Owned>;
```

### DeleteParams

```rust
pub type DeleteParams = Params<Owned>;
```

### DeleteResults

```rust
pub type DeleteResults = Results<Owned>;
```

### CommitParams

```rust
pub type CommitParams = Params<Owned>;
```

### CommitResults

```rust
pub type CommitResults = Results<Owned>;
```

### AbortStageParams

```rust
pub type AbortStageParams = Params<Owned>;
```

### AbortStageResults

```rust
pub type AbortStageResults = Results<Owned>;
```

### MoveParams

```rust
pub type MoveParams = Params<Owned>;
```

### MoveResults

```rust
pub type MoveResults = Results<Owned>;
```

### MoveManyParams

```rust
pub type MoveManyParams = Params<Owned>;
```

### MoveManyResults

```rust
pub type MoveManyResults = Results<Owned>;
```

### DeleteManyParams

```rust
pub type DeleteManyParams = Params<Owned>;
```

### DeleteManyResults

```rust
pub type DeleteManyResults = Results<Owned>;
```

### HeadManyParams

```rust
pub type HeadManyParams = Params<Owned>;
```

### HeadManyResults

```rust
pub type HeadManyResults = Results<Owned>;
```

### ChangesParams

```rust
pub type ChangesParams = Params<Owned>;
```

### ChangesResults

```rust
pub type ChangesResults = Results<Owned>;
```
