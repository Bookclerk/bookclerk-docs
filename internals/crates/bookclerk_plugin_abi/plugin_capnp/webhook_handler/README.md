# bookclerk_plugin_abi::plugin_capnp::webhook_handler

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`DeliveredParams`](#deliveredparams) | type alias |  |
| [`DeliveredResults`](#deliveredresults) | type alias |  |

## Modules

- [`delivered_params`](delivered_params.md)
- [`delivered_results`](delivered_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn delivered(
        self: Rc<Self>,
        _: DeliveredParams,
        _: DeliveredResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::delivered

```rust
fn delivered(
    self: Rc<Self>,
    _: DeliveredParams,
    _: DeliveredResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::delivered_request

```rust
pub fn delivered_request(&self) -> Request<Owned, Owned>
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

### DeliveredParams

```rust
pub type DeliveredParams = Params<Owned>;
```

### DeliveredResults

```rust
pub type DeliveredResults = Results<Owned>;
```
