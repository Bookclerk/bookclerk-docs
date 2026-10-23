# bookclerk_plugin_abi::plugin_capnp::remote_library

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`StartParams`](#startparams) | type alias |  |
| [`StartResults`](#startresults) | type alias |  |
| [`StopParams`](#stopparams) | type alias |  |
| [`StopResults`](#stopresults) | type alias |  |
| [`DiagnoseParams`](#diagnoseparams) | type alias |  |
| [`DiagnoseResults`](#diagnoseresults) | type alias |  |
| [`ScanLibraryParams`](#scanlibraryparams) | type alias |  |
| [`ScanLibraryResults`](#scanlibraryresults) | type alias |  |
| [`SyncListeningParams`](#synclisteningparams) | type alias |  |
| [`SyncListeningResults`](#synclisteningresults) | type alias |  |
| [`PollEventsParams`](#polleventsparams) | type alias |  |
| [`PollEventsResults`](#polleventsresults) | type alias |  |

## Modules

- [`health_params`](remote_library/health_params.md)
- [`health_results`](remote_library/health_results.md)
- [`start_params`](remote_library/start_params.md)
- [`start_results`](remote_library/start_results.md)
- [`stop_params`](remote_library/stop_params.md)
- [`stop_results`](remote_library/stop_results.md)
- [`diagnose_params`](remote_library/diagnose_params.md)
- [`diagnose_results`](remote_library/diagnose_results.md)
- [`scan_library_params`](remote_library/scan_library_params.md)
- [`scan_library_results`](remote_library/scan_library_results.md)
- [`sync_listening_params`](remote_library/sync_listening_params.md)
- [`sync_listening_results`](remote_library/sync_listening_results.md)
- [`poll_events_params`](remote_library/poll_events_params.md)
- [`poll_events_results`](remote_library/poll_events_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn start(
        self: Rc<Self>,
        _: StartParams,
        _: StartResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn stop(
        self: Rc<Self>,
        _: StopParams,
        _: StopResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn diagnose(
        self: Rc<Self>,
        _: DiagnoseParams,
        _: DiagnoseResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn scan_library(
        self: Rc<Self>,
        _: ScanLibraryParams,
        _: ScanLibraryResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn sync_listening(
        self: Rc<Self>,
        _: SyncListeningParams,
        _: SyncListeningResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn poll_events(
        self: Rc<Self>,
        _: PollEventsParams,
        _: PollEventsResults,
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

#### Server::start

```rust
fn start(
    self: Rc<Self>,
    _: StartParams,
    _: StartResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::stop

```rust
fn stop(
    self: Rc<Self>,
    _: StopParams,
    _: StopResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::diagnose

```rust
fn diagnose(
    self: Rc<Self>,
    _: DiagnoseParams,
    _: DiagnoseResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::scan_library

```rust
fn scan_library(
    self: Rc<Self>,
    _: ScanLibraryParams,
    _: ScanLibraryResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::sync_listening

```rust
fn sync_listening(
    self: Rc<Self>,
    _: SyncListeningParams,
    _: SyncListeningResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::poll_events

```rust
fn poll_events(
    self: Rc<Self>,
    _: PollEventsParams,
    _: PollEventsResults,
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

#### Client::start_request

```rust
pub fn start_request(&self) -> Request<Owned, Owned>
```

#### Client::stop_request

```rust
pub fn stop_request(&self) -> Request<Owned, Owned>
```

#### Client::diagnose_request

```rust
pub fn diagnose_request(&self) -> Request<Owned, Owned>
```

#### Client::scan_library_request

```rust
pub fn scan_library_request(&self) -> Request<Owned, Owned>
```

#### Client::sync_listening_request

```rust
pub fn sync_listening_request(&self) -> Request<Owned, Owned>
```

#### Client::poll_events_request

```rust
pub fn poll_events_request(&self) -> Request<Owned, Owned>
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

### StartParams

```rust
pub type StartParams = Params<Owned>;
```

### StartResults

```rust
pub type StartResults = Results<Owned>;
```

### StopParams

```rust
pub type StopParams = Params<Owned>;
```

### StopResults

```rust
pub type StopResults = Results<Owned>;
```

### DiagnoseParams

```rust
pub type DiagnoseParams = Params<Owned>;
```

### DiagnoseResults

```rust
pub type DiagnoseResults = Results<Owned>;
```

### ScanLibraryParams

```rust
pub type ScanLibraryParams = Params<Owned>;
```

### ScanLibraryResults

```rust
pub type ScanLibraryResults = Results<Owned>;
```

### SyncListeningParams

```rust
pub type SyncListeningParams = Params<Owned>;
```

### SyncListeningResults

```rust
pub type SyncListeningResults = Results<Owned>;
```

### PollEventsParams

```rust
pub type PollEventsParams = Params<Owned>;
```

### PollEventsResults

```rust
pub type PollEventsResults = Results<Owned>;
```
