# bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`BeginParams`](#beginparams) | type alias |  |
| [`BeginResults`](#beginresults) | type alias |  |
| [`ExecuteParams`](#executeparams) | type alias |  |
| [`ExecuteResults`](#executeresults) | type alias |  |

## Modules

- [`begin_params`](host_adapter_database_session/begin_params.md)
- [`begin_results`](host_adapter_database_session/begin_results.md)
- [`execute_params`](host_adapter_database_session/execute_params.md)
- [`execute_results`](host_adapter_database_session/execute_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn begin(
        self: Rc<Self>,
        _: BeginParams,
        _: BeginResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn execute(
        self: Rc<Self>,
        _: ExecuteParams,
        _: ExecuteResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::begin

```rust
fn begin(
    self: Rc<Self>,
    _: BeginParams,
    _: BeginResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::execute

```rust
fn execute(
    self: Rc<Self>,
    _: ExecuteParams,
    _: ExecuteResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::begin_request

```rust
pub fn begin_request(&self) -> Request<Owned, Owned>
```

#### Client::execute_request

```rust
pub fn execute_request(&self) -> Request<Owned, Owned>
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

### BeginParams

```rust
pub type BeginParams = Params<Owned>;
```

### BeginResults

```rust
pub type BeginResults = Results<Owned>;
```

### ExecuteParams

```rust
pub type ExecuteParams = Params<Owned>;
```

### ExecuteResults

```rust
pub type ExecuteResults = Results<Owned>;
```
