# bookclerk_plugin_abi::plugin_capnp::job_runner

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`JobParams`](#jobparams) | type alias |  |
| [`JobResults`](#jobresults) | type alias |  |

## Modules

- [`job_params`](job_runner/job_params.md)
- [`job_results`](job_runner/job_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn job(
        self: Rc<Self>,
        _: JobParams,
        _: JobResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::job

```rust
fn job(
    self: Rc<Self>,
    _: JobParams,
    _: JobResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::job_request

```rust
pub fn job_request(&self) -> Request<Owned, Owned>
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

### JobParams

```rust
pub type JobParams = Params<Owned>;
```

### JobResults

```rust
pub type JobResults = Results<Owned>;
```
