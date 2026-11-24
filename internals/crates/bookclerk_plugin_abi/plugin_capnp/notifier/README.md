# bookclerk_plugin_abi::plugin_capnp::notifier

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`SendParams`](#sendparams) | type alias |  |
| [`SendResults`](#sendresults) | type alias |  |
| [`ValidateDestinationParams`](#validatedestinationparams) | type alias |  |
| [`ValidateDestinationResults`](#validatedestinationresults) | type alias |  |
| [`CheckDeliverabilityParams`](#checkdeliverabilityparams) | type alias |  |
| [`CheckDeliverabilityResults`](#checkdeliverabilityresults) | type alias |  |

## Modules

- [`health_params`](health_params.md)
- [`health_results`](health_results.md)
- [`send_params`](send_params.md)
- [`send_results`](send_results.md)
- [`validate_destination_params`](validate_destination_params.md)
- [`validate_destination_results`](validate_destination_results.md)
- [`check_deliverability_params`](check_deliverability_params.md)
- [`check_deliverability_results`](check_deliverability_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn send(
        self: Rc<Self>,
        _: SendParams,
        _: SendResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn validate_destination(
        self: Rc<Self>,
        _: ValidateDestinationParams,
        _: ValidateDestinationResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn check_deliverability(
        self: Rc<Self>,
        _: CheckDeliverabilityParams,
        _: CheckDeliverabilityResults,
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

#### Server::send

```rust
fn send(
    self: Rc<Self>,
    _: SendParams,
    _: SendResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::validate_destination

```rust
fn validate_destination(
    self: Rc<Self>,
    _: ValidateDestinationParams,
    _: ValidateDestinationResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::check_deliverability

```rust
fn check_deliverability(
    self: Rc<Self>,
    _: CheckDeliverabilityParams,
    _: CheckDeliverabilityResults,
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

#### Client::send_request

```rust
pub fn send_request(&self) -> Request<Owned, Owned>
```

#### Client::validate_destination_request

```rust
pub fn validate_destination_request(&self) -> Request<Owned, Owned>
```

#### Client::check_deliverability_request

```rust
pub fn check_deliverability_request(&self) -> Request<Owned, Owned>
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

### SendParams

```rust
pub type SendParams = Params<Owned>;
```

### SendResults

```rust
pub type SendResults = Results<Owned>;
```

### ValidateDestinationParams

```rust
pub type ValidateDestinationParams = Params<Owned>;
```

### ValidateDestinationResults

```rust
pub type ValidateDestinationResults = Results<Owned>;
```

### CheckDeliverabilityParams

```rust
pub type CheckDeliverabilityParams = Params<Owned>;
```

### CheckDeliverabilityResults

```rust
pub type CheckDeliverabilityResults = Results<Owned>;
```
