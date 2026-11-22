# bookclerk_plugin_abi::plugin_capnp::ai_models

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`ModelsParams`](#modelsparams) | type alias |  |
| [`ModelsResults`](#modelsresults) | type alias |  |
| [`RunParams`](#runparams) | type alias |  |
| [`RunResults`](#runresults) | type alias |  |

## Modules

- [`health_params`](health_params.md)
- [`health_results`](health_results.md)
- [`models_params`](models_params.md)
- [`models_results`](models_results.md)
- [`run_params`](run_params.md)
- [`run_results`](run_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn models(
        self: Rc<Self>,
        _: ModelsParams,
        _: ModelsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn run(
        self: Rc<Self>,
        _: RunParams,
        _: RunResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::health

```rust
fn health(
    self: Rc<Self>,
    _: HealthParams,
    _: HealthResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::models

```rust
fn models(
    self: Rc<Self>,
    _: ModelsParams,
    _: ModelsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
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

#### Client::health_request

```rust
pub fn health_request(&self) -> Request<Owned, Owned>
```

#### Client::models_request

```rust
pub fn models_request(&self) -> Request<Owned, Owned>
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

### HealthParams

```rust
pub type HealthParams = Params<Owned>;
```

### HealthResults

```rust
pub type HealthResults = Results<Owned>;
```

### ModelsParams

```rust
pub type ModelsParams = Params<Owned>;
```

### ModelsResults

```rust
pub type ModelsResults = Results<Owned>;
```

### RunParams

```rust
pub type RunParams = Params<Owned>;
```

### RunResults

```rust
pub type RunResults = Results<Owned>;
```
