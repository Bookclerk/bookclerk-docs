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

## Modules

- [`head_params`](destination/head_params.md)
- [`head_results`](destination/head_results.md)
- [`list_params`](destination/list_params.md)
- [`list_results`](destination/list_results.md)
- [`get_params`](destination/get_params.md)
- [`get_results`](destination/get_results.md)
- [`put_params`](destination/put_params.md)
- [`put_results`](destination/put_results.md)
- [`copy_params`](destination/copy_params.md)
- [`copy_results`](destination/copy_results.md)
- [`delete_params`](destination/delete_params.md)
- [`delete_results`](destination/delete_results.md)
- [`commit_params`](destination/commit_params.md)
- [`commit_results`](destination/commit_results.md)
- [`abort_stage_params`](destination/abort_stage_params.md)
- [`abort_stage_results`](destination/abort_stage_results.md)

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
