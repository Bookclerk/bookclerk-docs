# bookclerk_plugin_abi::plugin_capnp::queue_consumer

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`QueueParams`](#queueparams) | type alias |  |
| [`QueueResults`](#queueresults) | type alias |  |

## Modules

- [`queue_params`](queue_consumer/queue_params.md)
- [`queue_results`](queue_consumer/queue_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn queue(
        self: Rc<Self>,
        _: QueueParams,
        _: QueueResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::queue

```rust
fn queue(
    self: Rc<Self>,
    _: QueueParams,
    _: QueueResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::queue_request

```rust
pub fn queue_request(&self) -> Request<Owned, Owned>
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

### QueueParams

```rust
pub type QueueParams = Params<Owned>;
```

### QueueResults

```rust
pub type QueueResults = Results<Owned>;
```
