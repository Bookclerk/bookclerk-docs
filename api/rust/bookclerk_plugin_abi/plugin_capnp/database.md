# bookclerk_plugin_abi::plugin_capnp::database

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`OpenSessionParams`](#opensessionparams) | type alias |  |
| [`OpenSessionResults`](#opensessionresults) | type alias |  |
| [`DropUnitParams`](#dropunitparams) | type alias |  |
| [`DropUnitResults`](#dropunitresults) | type alias |  |

## Modules

- [`open_session_params`](database/open_session_params.md)
- [`open_session_results`](database/open_session_results.md)
- [`drop_unit_params`](database/drop_unit_params.md)
- [`drop_unit_results`](database/drop_unit_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn open_session(
        self: Rc<Self>,
        _: OpenSessionParams,
        _: OpenSessionResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn drop_unit(
        self: Rc<Self>,
        _: DropUnitParams,
        _: DropUnitResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::open_session

```rust
fn open_session(
    self: Rc<Self>,
    _: OpenSessionParams,
    _: OpenSessionResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::drop_unit

```rust
fn drop_unit(
    self: Rc<Self>,
    _: DropUnitParams,
    _: DropUnitResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::open_session_request

```rust
pub fn open_session_request(&self) -> Request<Owned, Owned>
```

#### Client::drop_unit_request

```rust
pub fn drop_unit_request(&self) -> Request<Owned, Owned>
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

### OpenSessionParams

```rust
pub type OpenSessionParams = Params<Owned>;
```

### OpenSessionResults

```rust
pub type OpenSessionResults = Results<Owned>;
```

### DropUnitParams

```rust
pub type DropUnitParams = Params<Owned>;
```

### DropUnitResults

```rust
pub type DropUnitResults = Results<Owned>;
```
