# bookclerk_plugin_abi::plugin_capnp::progress_sink

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`ReportParams`](#reportparams) | type alias |  |
| [`ReportResults`](#reportresults) | type alias |  |

## Modules

- [`report_params`](progress_sink/report_params.md)
- [`report_results`](progress_sink/report_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn report(
        self: Rc<Self>,
        _: ReportParams,
        _: ReportResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::report

```rust
fn report(
    self: Rc<Self>,
    _: ReportParams,
    _: ReportResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::report_request

```rust
pub fn report_request(&self) -> Request<Owned, Owned>
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

### ReportParams

```rust
pub type ReportParams = Params<Owned>;
```

### ReportResults

```rust
pub type ReportResults = Results<Owned>;
```
