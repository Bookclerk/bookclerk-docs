# bookclerk_plugin_abi::plugin_capnp::metadata_files

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`ParseParams`](#parseparams) | type alias |  |
| [`ParseResults`](#parseresults) | type alias |  |
| [`RenderParams`](#renderparams) | type alias |  |
| [`RenderResults`](#renderresults) | type alias |  |

## Modules

- [`health_params`](health_params.md)
- [`health_results`](health_results.md)
- [`parse_params`](parse_params.md)
- [`parse_results`](parse_results.md)
- [`render_params`](render_params.md)
- [`render_results`](render_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn parse(
        self: Rc<Self>,
        _: ParseParams,
        _: ParseResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn render(
        self: Rc<Self>,
        _: RenderParams,
        _: RenderResults,
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

#### Server::parse

```rust
fn parse(
    self: Rc<Self>,
    _: ParseParams,
    _: ParseResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::render

```rust
fn render(
    self: Rc<Self>,
    _: RenderParams,
    _: RenderResults,
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

#### Client::parse_request

```rust
pub fn parse_request(&self) -> Request<Owned, Owned>
```

#### Client::render_request

```rust
pub fn render_request(&self) -> Request<Owned, Owned>
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

### ParseParams

```rust
pub type ParseParams = Params<Owned>;
```

### ParseResults

```rust
pub type ParseResults = Results<Owned>;
```

### RenderParams

```rust
pub type RenderParams = Params<Owned>;
```

### RenderResults

```rust
pub type RenderResults = Results<Owned>;
```
