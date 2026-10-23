# bookclerk_plugin_abi::plugin_capnp::source

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`OpenParams`](#openparams) | type alias |  |
| [`OpenResults`](#openresults) | type alias |  |

## Modules

- [`open_params`](source/open_params.md)
- [`open_results`](source/open_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn open(
        self: Rc<Self>,
        _: OpenParams,
        _: OpenResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::open

```rust
fn open(
    self: Rc<Self>,
    _: OpenParams,
    _: OpenResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

Implemented by: `SourceServer`

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::open_request

```rust
pub fn open_request(&self) -> Request<Owned, Owned>
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

### OpenParams

```rust
pub type OpenParams = Params<Owned>;
```

### OpenResults

```rust
pub type OpenResults = Results<Owned>;
```
