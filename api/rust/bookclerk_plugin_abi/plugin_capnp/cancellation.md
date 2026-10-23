# bookclerk_plugin_abi::plugin_capnp::cancellation

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`PollParams`](#pollparams) | type alias |  |
| [`PollResults`](#pollresults) | type alias |  |

## Modules

- [`poll_params`](cancellation/poll_params.md)
- [`poll_results`](cancellation/poll_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn poll(
        self: Rc<Self>,
        _: PollParams,
        _: PollResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::poll

```rust
fn poll(
    self: Rc<Self>,
    _: PollParams,
    _: PollResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::poll_request

```rust
pub fn poll_request(&self) -> Request<Owned, Owned>
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

### PollParams

```rust
pub type PollParams = Params<Owned>;
```

### PollResults

```rust
pub type PollResults = Results<Owned>;
```
