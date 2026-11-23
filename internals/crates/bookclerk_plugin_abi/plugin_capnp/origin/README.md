# bookclerk_plugin_abi::plugin_capnp::origin

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`FetchParams`](#fetchparams) | type alias |  |
| [`FetchResults`](#fetchresults) | type alias |  |
| [`ConnectParams`](#connectparams) | type alias |  |
| [`ConnectResults`](#connectresults) | type alias |  |

## Modules

- [`fetch_params`](fetch_params.md)
- [`fetch_results`](fetch_results.md)
- [`connect_params`](connect_params.md)
- [`connect_results`](connect_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn fetch(
        self: Rc<Self>,
        _: FetchParams,
        _: FetchResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn connect(
        self: Rc<Self>,
        _: ConnectParams,
        _: ConnectResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::fetch

```rust
fn fetch(
    self: Rc<Self>,
    _: FetchParams,
    _: FetchResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::connect

```rust
fn connect(
    self: Rc<Self>,
    _: ConnectParams,
    _: ConnectResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::fetch_request

```rust
pub fn fetch_request(&self) -> Request<Owned, Owned>
```

#### Client::connect_request

```rust
pub fn connect_request(&self) -> Request<Owned, Owned>
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

### FetchParams

```rust
pub type FetchParams = Params<Owned>;
```

### FetchResults

```rust
pub type FetchResults = Results<Owned>;
```

### ConnectParams

```rust
pub type ConnectParams = Params<Owned>;
```

### ConnectResults

```rust
pub type ConnectResults = Results<Owned>;
```
