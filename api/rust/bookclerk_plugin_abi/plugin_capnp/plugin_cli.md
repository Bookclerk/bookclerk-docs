# bookclerk_plugin_abi::plugin_capnp::plugin_cli

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`DescribeParams`](#describeparams) | type alias |  |
| [`DescribeResults`](#describeresults) | type alias |  |
| [`InvokeParams`](#invokeparams) | type alias |  |
| [`InvokeResults`](#invokeresults) | type alias |  |

## Modules

- [`describe_params`](plugin_cli/describe_params.md)
- [`describe_results`](plugin_cli/describe_results.md)
- [`invoke_params`](plugin_cli/invoke_params.md)
- [`invoke_results`](plugin_cli/invoke_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn describe(
        self: Rc<Self>,
        _: DescribeParams,
        _: DescribeResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn invoke(
        self: Rc<Self>,
        _: InvokeParams,
        _: InvokeResults,
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

#### Server::invoke

```rust
fn invoke(
    self: Rc<Self>,
    _: InvokeParams,
    _: InvokeResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

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

#### Client::invoke_request

```rust
pub fn invoke_request(&self) -> Request<Owned, Owned>
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

### InvokeParams

```rust
pub type InvokeParams = Params<Owned>;
```

### InvokeResults

```rust
pub type InvokeResults = Results<Owned>;
```
