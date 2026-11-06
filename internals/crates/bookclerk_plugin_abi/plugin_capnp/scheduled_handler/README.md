# bookclerk_plugin_abi::plugin_capnp::scheduled_handler

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`ScheduledParams`](#scheduledparams) | type alias |  |
| [`ScheduledResults`](#scheduledresults) | type alias |  |

## Modules

- [`scheduled_params`](scheduled_params.md)
- [`scheduled_results`](scheduled_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn scheduled(
        self: Rc<Self>,
        _: ScheduledParams,
        _: ScheduledResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::scheduled

```rust
fn scheduled(
    self: Rc<Self>,
    _: ScheduledParams,
    _: ScheduledResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::scheduled_request

```rust
pub fn scheduled_request(&self) -> Request<Owned, Owned>
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

### ScheduledParams

```rust
pub type ScheduledParams = Params<Owned>;
```

### ScheduledResults

```rust
pub type ScheduledResults = Results<Owned>;
```
