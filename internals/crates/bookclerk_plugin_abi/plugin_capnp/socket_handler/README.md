# bookclerk_plugin_abi::plugin_capnp::socket_handler

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`MessageParams`](#messageparams) | type alias |  |
| [`MessageResults`](#messageresults) | type alias |  |
| [`ClosedParams`](#closedparams) | type alias |  |
| [`ClosedResults`](#closedresults) | type alias |  |

## Modules

- [`message_params`](message_params.md)
- [`message_results`](message_results.md)
- [`closed_params`](closed_params.md)
- [`closed_results`](closed_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn message(
        self: Rc<Self>,
        _: MessageParams,
        _: MessageResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn closed(
        self: Rc<Self>,
        _: ClosedParams,
        _: ClosedResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::message

```rust
fn message(
    self: Rc<Self>,
    _: MessageParams,
    _: MessageResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::closed

```rust
fn closed(
    self: Rc<Self>,
    _: ClosedParams,
    _: ClosedResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::message_request

```rust
pub fn message_request(&self) -> Request<Owned, Owned>
```

#### Client::closed_request

```rust
pub fn closed_request(&self) -> Request<Owned, Owned>
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

### MessageParams

```rust
pub type MessageParams = Params<Owned>;
```

### MessageResults

```rust
pub type MessageResults = Results<Owned>;
```

### ClosedParams

```rust
pub type ClosedParams = Params<Owned>;
```

### ClosedResults

```rust
pub type ClosedResults = Results<Owned>;
```
