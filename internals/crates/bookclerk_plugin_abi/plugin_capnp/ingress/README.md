# bookclerk_plugin_abi::plugin_capnp::ingress

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`StartParams`](#startparams) | type alias |  |
| [`StartResults`](#startresults) | type alias |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`StopParams`](#stopparams) | type alias |  |
| [`StopResults`](#stopresults) | type alias |  |

## Modules

- [`start_params`](start_params.md)
- [`start_results`](start_results.md)
- [`health_params`](health_params.md)
- [`health_results`](health_results.md)
- [`stop_params`](stop_params.md)
- [`stop_results`](stop_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn start(
        self: Rc<Self>,
        _: StartParams,
        _: StartResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn stop(
        self: Rc<Self>,
        _: StopParams,
        _: StopResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::start

```rust
fn start(
    self: Rc<Self>,
    _: StartParams,
    _: StartResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::health

```rust
fn health(
    self: Rc<Self>,
    _: HealthParams,
    _: HealthResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::stop

```rust
fn stop(
    self: Rc<Self>,
    _: StopParams,
    _: StopResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::start_request

```rust
pub fn start_request(&self) -> Request<Owned, Owned>
```

#### Client::health_request

```rust
pub fn health_request(&self) -> Request<Owned, Owned>
```

#### Client::stop_request

```rust
pub fn stop_request(&self) -> Request<Owned, Owned>
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

### StartParams

```rust
pub type StartParams = Params<Owned>;
```

### StartResults

```rust
pub type StartResults = Results<Owned>;
```

### HealthParams

```rust
pub type HealthParams = Params<Owned>;
```

### HealthResults

```rust
pub type HealthResults = Results<Owned>;
```

### StopParams

```rust
pub type StopParams = Params<Owned>;
```

### StopResults

```rust
pub type StopResults = Results<Owned>;
```
