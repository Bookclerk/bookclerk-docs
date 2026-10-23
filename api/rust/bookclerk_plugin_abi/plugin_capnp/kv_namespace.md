# bookclerk_plugin_abi::plugin_capnp::kv_namespace

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`GetParams`](#getparams) | type alias |  |
| [`GetResults`](#getresults) | type alias |  |
| [`PutParams`](#putparams) | type alias |  |
| [`PutResults`](#putresults) | type alias |  |
| [`DeleteParams`](#deleteparams) | type alias |  |
| [`DeleteResults`](#deleteresults) | type alias |  |
| [`ListParams`](#listparams) | type alias |  |
| [`ListResults`](#listresults) | type alias |  |

## Modules

- [`get_params`](kv_namespace/get_params.md)
- [`get_results`](kv_namespace/get_results.md)
- [`put_params`](kv_namespace/put_params.md)
- [`put_results`](kv_namespace/put_results.md)
- [`delete_params`](kv_namespace/delete_params.md)
- [`delete_results`](kv_namespace/delete_results.md)
- [`list_params`](kv_namespace/list_params.md)
- [`list_results`](kv_namespace/list_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
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
    fn delete(
        self: Rc<Self>,
        _: DeleteParams,
        _: DeleteResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn list(
        self: Rc<Self>,
        _: ListParams,
        _: ListResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
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

#### Server::delete

```rust
fn delete(
    self: Rc<Self>,
    _: DeleteParams,
    _: DeleteResults,
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

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::get_request

```rust
pub fn get_request(&self) -> Request<Owned, Owned>
```

#### Client::put_request

```rust
pub fn put_request(&self) -> Request<Owned, Owned>
```

#### Client::delete_request

```rust
pub fn delete_request(&self) -> Request<Owned, Owned>
```

#### Client::list_request

```rust
pub fn list_request(&self) -> Request<Owned, Owned>
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

### DeleteParams

```rust
pub type DeleteParams = Params<Owned>;
```

### DeleteResults

```rust
pub type DeleteResults = Results<Owned>;
```

### ListParams

```rust
pub type ListParams = Params<Owned>;
```

### ListResults

```rust
pub type ListResults = Results<Owned>;
```
