# bookclerk_plugin_abi::plugin_capnp::queue_producer

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`SendParams`](#sendparams) | type alias |  |
| [`SendResults`](#sendresults) | type alias |  |
| [`SendBatchParams`](#sendbatchparams) | type alias |  |
| [`SendBatchResults`](#sendbatchresults) | type alias |  |

## Modules

- [`send_params`](queue_producer/send_params.md)
- [`send_results`](queue_producer/send_results.md)
- [`send_batch_params`](queue_producer/send_batch_params.md)
- [`send_batch_results`](queue_producer/send_batch_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn send(
        self: Rc<Self>,
        _: SendParams,
        _: SendResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn send_batch(
        self: Rc<Self>,
        _: SendBatchParams,
        _: SendBatchResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::send

```rust
fn send(
    self: Rc<Self>,
    _: SendParams,
    _: SendResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::send_batch

```rust
fn send_batch(
    self: Rc<Self>,
    _: SendBatchParams,
    _: SendBatchResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::send_request

```rust
pub fn send_request(&self) -> Request<Owned, Owned>
```

#### Client::send_batch_request

```rust
pub fn send_batch_request(&self) -> Request<Owned, Owned>
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

### SendParams

```rust
pub type SendParams = Params<Owned>;
```

### SendResults

```rust
pub type SendResults = Results<Owned>;
```

### SendBatchParams

```rust
pub type SendBatchParams = Params<Owned>;
```

### SendBatchResults

```rust
pub type SendBatchResults = Results<Owned>;
```
