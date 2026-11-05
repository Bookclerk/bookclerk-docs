# bookclerk_plugin_abi::plugin_capnp::cover_fetcher

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`FetchCoversParams`](#fetchcoversparams) | type alias |  |
| [`FetchCoversResults`](#fetchcoversresults) | type alias |  |

## Modules

- [`health_params`](cover_fetcher/health_params.md)
- [`health_results`](cover_fetcher/health_results.md)
- [`fetch_covers_params`](cover_fetcher/fetch_covers_params.md)
- [`fetch_covers_results`](cover_fetcher/fetch_covers_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn fetch_covers(
        self: Rc<Self>,
        _: FetchCoversParams,
        _: FetchCoversResults,
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

#### Server::fetch_covers

```rust
fn fetch_covers(
    self: Rc<Self>,
    _: FetchCoversParams,
    _: FetchCoversResults,
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

#### Client::fetch_covers_request

```rust
pub fn fetch_covers_request(&self) -> Request<Owned, Owned>
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

### FetchCoversParams

```rust
pub type FetchCoversParams = Params<Owned>;
```

### FetchCoversResults

```rust
pub type FetchCoversResults = Results<Owned>;
```
