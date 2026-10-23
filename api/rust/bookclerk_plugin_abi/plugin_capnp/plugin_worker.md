# bookclerk_plugin_abi::plugin_capnp::plugin_worker

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`DescribeParams`](#describeparams) | type alias |  |
| [`DescribeResults`](#describeresults) | type alias |  |
| [`OpenParams`](#openparams) | type alias |  |
| [`OpenResults`](#openresults) | type alias |  |
| [`ShutdownParams`](#shutdownparams) | type alias |  |
| [`ShutdownResults`](#shutdownresults) | type alias |  |
| [`DatabaseMigrationsParams`](#databasemigrationsparams) | type alias |  |
| [`DatabaseMigrationsResults`](#databasemigrationsresults) | type alias |  |

## Modules

- [`describe_params`](plugin_worker/describe_params.md)
- [`describe_results`](plugin_worker/describe_results.md)
- [`open_params`](plugin_worker/open_params.md)
- [`open_results`](plugin_worker/open_results.md)
- [`shutdown_params`](plugin_worker/shutdown_params.md)
- [`shutdown_results`](plugin_worker/shutdown_results.md)
- [`database_migrations_params`](plugin_worker/database_migrations_params.md)
- [`database_migrations_results`](plugin_worker/database_migrations_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn describe(
        self: Rc<Self>,
        _: DescribeParams,
        _: DescribeResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn open(
        self: Rc<Self>,
        _: OpenParams,
        _: OpenResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn shutdown(
        self: Rc<Self>,
        _: ShutdownParams,
        _: ShutdownResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn database_migrations(
        self: Rc<Self>,
        _: DatabaseMigrationsParams,
        _: DatabaseMigrationsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::describe

```rust
fn describe(
    self: Rc<Self>,
    _: DescribeParams,
    _: DescribeResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::open

```rust
fn open(
    self: Rc<Self>,
    _: OpenParams,
    _: OpenResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::shutdown

```rust
fn shutdown(
    self: Rc<Self>,
    _: ShutdownParams,
    _: ShutdownResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::database_migrations

```rust
fn database_migrations(
    self: Rc<Self>,
    _: DatabaseMigrationsParams,
    _: DatabaseMigrationsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

Implemented by: `PluginServer`

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::describe_request

```rust
pub fn describe_request(&self) -> Request<Owned, Owned>
```

#### Client::open_request

```rust
pub fn open_request(&self) -> Request<Owned, Owned>
```

#### Client::shutdown_request

```rust
pub fn shutdown_request(&self) -> Request<Owned, Owned>
```

#### Client::database_migrations_request

```rust
pub fn database_migrations_request(&self) -> Request<Owned, Owned>
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

### DescribeParams

```rust
pub type DescribeParams = Params<Owned>;
```

### DescribeResults

```rust
pub type DescribeResults = Results<Owned>;
```

### OpenParams

```rust
pub type OpenParams = Params<Owned>;
```

### OpenResults

```rust
pub type OpenResults = Results<Owned>;
```

### ShutdownParams

```rust
pub type ShutdownParams = Params<Owned>;
```

### ShutdownResults

```rust
pub type ShutdownResults = Results<Owned>;
```

### DatabaseMigrationsParams

```rust
pub type DatabaseMigrationsParams = Params<Owned>;
```

### DatabaseMigrationsResults

```rust
pub type DatabaseMigrationsResults = Results<Owned>;
```
