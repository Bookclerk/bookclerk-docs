# bookclerk_plugin_abi::plugin_capnp::event_publisher

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`PublishParams`](#publishparams) | type alias |  |
| [`PublishResults`](#publishresults) | type alias |  |

## Modules

- [`publish_params`](event_publisher/publish_params.md)
- [`publish_results`](event_publisher/publish_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn publish(
        self: Rc<Self>,
        _: PublishParams,
        _: PublishResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::publish

```rust
fn publish(
    self: Rc<Self>,
    _: PublishParams,
    _: PublishResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::publish_request

```rust
pub fn publish_request(&self) -> Request<Owned, Owned>
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

### PublishParams

```rust
pub type PublishParams = Params<Owned>;
```

### PublishResults

```rust
pub type PublishResults = Results<Owned>;
```
