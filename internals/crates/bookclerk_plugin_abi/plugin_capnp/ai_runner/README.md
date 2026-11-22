# bookclerk_plugin_abi::plugin_capnp::ai_runner

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`RunParams`](#runparams) | type alias |  |
| [`RunResults`](#runresults) | type alias |  |

## Modules

- [`run_params`](run_params.md)
- [`run_results`](run_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn run(
        self: Rc<Self>,
        _: RunParams,
        _: RunResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::run

```rust
fn run(
    self: Rc<Self>,
    _: RunParams,
    _: RunResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::run_request

```rust
pub fn run_request(&self) -> Request<Owned, Owned>
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

### RunParams

```rust
pub type RunParams = Params<Owned>;
```

### RunResults

```rust
pub type RunResults = Results<Owned>;
```
