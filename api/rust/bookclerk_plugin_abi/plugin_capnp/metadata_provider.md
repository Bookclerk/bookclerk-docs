# bookclerk_plugin_abi::plugin_capnp::metadata_provider

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`LookupParams`](#lookupparams) | type alias |  |
| [`LookupResults`](#lookupresults) | type alias |  |
| [`SearchParams`](#searchparams) | type alias |  |
| [`SearchResults`](#searchresults) | type alias |  |
| [`ChaptersParams`](#chaptersparams) | type alias |  |
| [`ChaptersResults`](#chaptersresults) | type alias |  |

## Modules

- [`health_params`](metadata_provider/health_params.md)
- [`health_results`](metadata_provider/health_results.md)
- [`lookup_params`](metadata_provider/lookup_params.md)
- [`lookup_results`](metadata_provider/lookup_results.md)
- [`search_params`](metadata_provider/search_params.md)
- [`search_results`](metadata_provider/search_results.md)
- [`chapters_params`](metadata_provider/chapters_params.md)
- [`chapters_results`](metadata_provider/chapters_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn lookup(
        self: Rc<Self>,
        _: LookupParams,
        _: LookupResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn search(
        self: Rc<Self>,
        _: SearchParams,
        _: SearchResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn chapters(
        self: Rc<Self>,
        _: ChaptersParams,
        _: ChaptersResults,
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

#### Server::lookup

```rust
fn lookup(
    self: Rc<Self>,
    _: LookupParams,
    _: LookupResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::search

```rust
fn search(
    self: Rc<Self>,
    _: SearchParams,
    _: SearchResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::chapters

```rust
fn chapters(
    self: Rc<Self>,
    _: ChaptersParams,
    _: ChaptersResults,
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

#### Client::lookup_request

```rust
pub fn lookup_request(&self) -> Request<Owned, Owned>
```

#### Client::search_request

```rust
pub fn search_request(&self) -> Request<Owned, Owned>
```

#### Client::chapters_request

```rust
pub fn chapters_request(&self) -> Request<Owned, Owned>
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

### LookupParams

```rust
pub type LookupParams = Params<Owned>;
```

### LookupResults

```rust
pub type LookupResults = Results<Owned>;
```

### SearchParams

```rust
pub type SearchParams = Params<Owned>;
```

### SearchResults

```rust
pub type SearchResults = Results<Owned>;
```

### ChaptersParams

```rust
pub type ChaptersParams = Params<Owned>;
```

### ChaptersResults

```rust
pub type ChaptersResults = Results<Owned>;
```
