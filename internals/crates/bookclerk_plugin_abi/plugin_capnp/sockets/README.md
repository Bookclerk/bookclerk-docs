# bookclerk_plugin_abi::plugin_capnp::sockets

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`ConnectParams`](#connectparams) | type alias |  |
| [`ConnectResults`](#connectresults) | type alias |  |
| [`SendParams`](#sendparams) | type alias |  |
| [`SendResults`](#sendresults) | type alias |  |
| [`CloseParams`](#closeparams) | type alias |  |
| [`CloseResults`](#closeresults) | type alias |  |
| [`SetAttachmentParams`](#setattachmentparams) | type alias |  |
| [`SetAttachmentResults`](#setattachmentresults) | type alias |  |
| [`ListParams`](#listparams) | type alias |  |
| [`ListResults`](#listresults) | type alias |  |

## Modules

- [`connect_params`](connect_params.md)
- [`connect_results`](connect_results.md)
- [`send_params`](send_params.md)
- [`send_results`](send_results.md)
- [`close_params`](close_params.md)
- [`close_results`](close_results.md)
- [`set_attachment_params`](set_attachment_params.md)
- [`set_attachment_results`](set_attachment_results.md)
- [`list_params`](list_params.md)
- [`list_results`](list_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn connect(
        self: Rc<Self>,
        _: ConnectParams,
        _: ConnectResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn send(
        self: Rc<Self>,
        _: SendParams,
        _: SendResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn close(
        self: Rc<Self>,
        _: CloseParams,
        _: CloseResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn set_attachment(
        self: Rc<Self>,
        _: SetAttachmentParams,
        _: SetAttachmentResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn list(
        self: Rc<Self>,
        _: ListParams,
        _: ListResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::connect

```rust
fn connect(
    self: Rc<Self>,
    _: ConnectParams,
    _: ConnectResults,
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

#### Server::close

```rust
fn close(
    self: Rc<Self>,
    _: CloseParams,
    _: CloseResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::set_attachment

```rust
fn set_attachment(
    self: Rc<Self>,
    _: SetAttachmentParams,
    _: SetAttachmentResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::list

```rust
fn list(
    self: Rc<Self>,
    _: ListParams,
    _: ListResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::connect_request

```rust
pub fn connect_request(&self) -> Request<Owned, Owned>
```

#### Client::send_request

```rust
pub fn send_request(&self) -> Request<Owned, Owned>
```

#### Client::close_request

```rust
pub fn close_request(&self) -> Request<Owned, Owned>
```

#### Client::set_attachment_request

```rust
pub fn set_attachment_request(&self) -> Request<Owned, Owned>
```

#### Client::list_request

```rust
pub fn list_request(&self) -> Request<Owned, Owned>
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

### ConnectParams

```rust
pub type ConnectParams = Params<Owned>;
```

### ConnectResults

```rust
pub type ConnectResults = Results<Owned>;
```

### SendParams

```rust
pub type SendParams = Params<Owned>;
```

### SendResults

```rust
pub type SendResults = Results<Owned>;
```

### CloseParams

```rust
pub type CloseParams = Params<Owned>;
```

### CloseResults

```rust
pub type CloseResults = Results<Owned>;
```

### SetAttachmentParams

```rust
pub type SetAttachmentParams = Params<Owned>;
```

### SetAttachmentResults

```rust
pub type SetAttachmentResults = Results<Owned>;
```

### ListParams

```rust
pub type ListParams = Params<Owned>;
```

### ListResults

```rust
pub type ListResults = Results<Owned>;
```
