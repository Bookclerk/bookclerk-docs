# bookclerk_plugin_abi::plugin_capnp::guest_database

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`ExecuteParams`](#executeparams) | type alias |  |
| [`ExecuteResults`](#executeresults) | type alias |  |
| [`CloseParams`](#closeparams) | type alias |  |
| [`CloseResults`](#closeresults) | type alias |  |
| [`LimitsParams`](#limitsparams) | type alias |  |
| [`LimitsResults`](#limitsresults) | type alias |  |
| [`ExecuteStreamedParams`](#executestreamedparams) | type alias |  |
| [`ExecuteStreamedResults`](#executestreamedresults) | type alias |  |

## Modules

- [`execute_params`](execute_params.md)
- [`execute_results`](execute_results.md)
- [`close_params`](close_params.md)
- [`close_results`](close_results.md)
- [`limits_params`](limits_params.md)
- [`limits_results`](limits_results.md)
- [`execute_streamed_params`](execute_streamed_params.md)
- [`execute_streamed_results`](execute_streamed_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn execute(
        self: Rc<Self>,
        _: ExecuteParams,
        _: ExecuteResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn close(
        self: Rc<Self>,
        _: CloseParams,
        _: CloseResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn limits(
        self: Rc<Self>,
        _: LimitsParams,
        _: LimitsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn execute_streamed(
        self: Rc<Self>,
        _: ExecuteStreamedParams,
        _: ExecuteStreamedResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::execute

```rust
fn execute(
    self: Rc<Self>,
    _: ExecuteParams,
    _: ExecuteResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::close

```rust
fn close(
    self: Rc<Self>,
    _: CloseParams,
    _: CloseResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::limits

```rust
fn limits(
    self: Rc<Self>,
    _: LimitsParams,
    _: LimitsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::execute_streamed

```rust
fn execute_streamed(
    self: Rc<Self>,
    _: ExecuteStreamedParams,
    _: ExecuteStreamedResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::execute_request

```rust
pub fn execute_request(&self) -> Request<Owned, Owned>
```

#### Client::close_request

```rust
pub fn close_request(&self) -> Request<Owned, Owned>
```

#### Client::limits_request

```rust
pub fn limits_request(&self) -> Request<Owned, Owned>
```

#### Client::execute_streamed_request

```rust
pub fn execute_streamed_request(&self) -> Request<Owned, Owned>
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

### ExecuteParams

```rust
pub type ExecuteParams = Params<Owned>;
```

### ExecuteResults

```rust
pub type ExecuteResults = Results<Owned>;
```

### CloseParams

```rust
pub type CloseParams = Params<Owned>;
```

### CloseResults

```rust
pub type CloseResults = Results<Owned>;
```

### LimitsParams

```rust
pub type LimitsParams = Params<Owned>;
```

### LimitsResults

```rust
pub type LimitsResults = Results<Owned>;
```

### ExecuteStreamedParams

```rust
pub type ExecuteStreamedParams = Params<Owned>;
```

### ExecuteStreamedResults

```rust
pub type ExecuteStreamedResults = Results<Owned>;
```
