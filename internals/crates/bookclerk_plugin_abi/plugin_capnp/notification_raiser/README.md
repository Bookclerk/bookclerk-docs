# bookclerk_plugin_abi::plugin_capnp::notification_raiser

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`RaiseParams`](#raiseparams) | type alias |  |
| [`RaiseResults`](#raiseresults) | type alias |  |

## Modules

- [`raise_params`](raise_params.md)
- [`raise_results`](raise_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn raise(
        self: Rc<Self>,
        _: RaiseParams,
        _: RaiseResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::raise

```rust
fn raise(
    self: Rc<Self>,
    _: RaiseParams,
    _: RaiseResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::raise_request

```rust
pub fn raise_request(&self) -> Request<Owned, Owned>
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

### RaiseParams

```rust
pub type RaiseParams = Params<Owned>;
```

### RaiseResults

```rust
pub type RaiseResults = Results<Owned>;
```
