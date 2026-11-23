# bookclerk_plugin_abi::plugin_capnp::provisioning

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`ProvisionParams`](#provisionparams) | type alias |  |
| [`ProvisionResults`](#provisionresults) | type alias |  |
| [`DeprovisionParams`](#deprovisionparams) | type alias |  |
| [`DeprovisionResults`](#deprovisionresults) | type alias |  |

## Modules

- [`provision_params`](provision_params.md)
- [`provision_results`](provision_results.md)
- [`deprovision_params`](deprovision_params.md)
- [`deprovision_results`](deprovision_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn provision(
        self: Rc<Self>,
        _: ProvisionParams,
        _: ProvisionResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn deprovision(
        self: Rc<Self>,
        _: DeprovisionParams,
        _: DeprovisionResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::provision

```rust
fn provision(
    self: Rc<Self>,
    _: ProvisionParams,
    _: ProvisionResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::deprovision

```rust
fn deprovision(
    self: Rc<Self>,
    _: DeprovisionParams,
    _: DeprovisionResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::provision_request

```rust
pub fn provision_request(&self) -> Request<Owned, Owned>
```

#### Client::deprovision_request

```rust
pub fn deprovision_request(&self) -> Request<Owned, Owned>
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

### ProvisionParams

```rust
pub type ProvisionParams = Params<Owned>;
```

### ProvisionResults

```rust
pub type ProvisionResults = Results<Owned>;
```

### DeprovisionParams

```rust
pub type DeprovisionParams = Params<Owned>;
```

### DeprovisionResults

```rust
pub type DeprovisionResults = Results<Owned>;
```
