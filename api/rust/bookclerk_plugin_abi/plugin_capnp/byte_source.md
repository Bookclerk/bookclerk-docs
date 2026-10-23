# bookclerk_plugin_abi::plugin_capnp::byte_source

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`PullParams`](#pullparams) | type alias |  |
| [`PullResults`](#pullresults) | type alias |  |

## Modules

- [`pull_params`](byte_source/pull_params.md)
- [`pull_results`](byte_source/pull_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn pull(
        self: Rc<Self>,
        _: PullParams,
        _: PullResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::pull

```rust
fn pull(
    self: Rc<Self>,
    _: PullParams,
    _: PullResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::pull_request

```rust
pub fn pull_request(&self) -> Request<Owned, Owned>
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

### PullParams

```rust
pub type PullParams = Params<Owned>;
```

### PullResults

```rust
pub type PullResults = Results<Owned>;
```
