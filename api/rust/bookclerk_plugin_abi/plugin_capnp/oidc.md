# bookclerk_plugin_abi::plugin_capnp::oidc

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`ClientsParams`](#clientsparams) | type alias |  |
| [`ClientsResults`](#clientsresults) | type alias |  |
| [`AuthenticateUserParams`](#authenticateuserparams) | type alias |  |
| [`AuthenticateUserResults`](#authenticateuserresults) | type alias |  |

## Modules

- [`clients_params`](oidc/clients_params.md)
- [`clients_results`](oidc/clients_results.md)
- [`authenticate_user_params`](oidc/authenticate_user_params.md)
- [`authenticate_user_results`](oidc/authenticate_user_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn clients(
        self: Rc<Self>,
        _: ClientsParams,
        _: ClientsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn authenticate_user(
        self: Rc<Self>,
        _: AuthenticateUserParams,
        _: AuthenticateUserResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::clients

```rust
fn clients(
    self: Rc<Self>,
    _: ClientsParams,
    _: ClientsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::authenticate_user

```rust
fn authenticate_user(
    self: Rc<Self>,
    _: AuthenticateUserParams,
    _: AuthenticateUserResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::clients_request

```rust
pub fn clients_request(&self) -> Request<Owned, Owned>
```

#### Client::authenticate_user_request

```rust
pub fn authenticate_user_request(&self) -> Request<Owned, Owned>
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

### ClientsParams

```rust
pub type ClientsParams = Params<Owned>;
```

### ClientsResults

```rust
pub type ClientsResults = Results<Owned>;
```

### AuthenticateUserParams

```rust
pub type AuthenticateUserParams = Params<Owned>;
```

### AuthenticateUserResults

```rust
pub type AuthenticateUserResults = Results<Owned>;
```
