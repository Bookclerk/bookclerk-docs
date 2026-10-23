# bookclerk_plugin_abi::plugin_capnp::event_consumer

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`EventParams`](#eventparams) | type alias |  |
| [`EventResults`](#eventresults) | type alias |  |

## Modules

- [`event_params`](event_consumer/event_params.md)
- [`event_results`](event_consumer/event_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn event(
        self: Rc<Self>,
        _: EventParams,
        _: EventResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::event

```rust
fn event(
    self: Rc<Self>,
    _: EventParams,
    _: EventResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::event_request

```rust
pub fn event_request(&self) -> Request<Owned, Owned>
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

### EventParams

```rust
pub type EventParams = Params<Owned>;
```

### EventResults

```rust
pub type EventResults = Results<Owned>;
```
