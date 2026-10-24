# bookclerk_plugin_abi::plugin_capnp::content_source

| Item | Kind | Summary |
| --- | --- | --- |
| [`Server`](#server) | trait |  |
| [`Client`](#client) | struct |  |
| [`Owned`](#owned) | struct |  |
| [`ServerDispatch`](#serverdispatch) | struct |  |
| [`LoginParams`](#loginparams) | type alias |  |
| [`LoginResults`](#loginresults) | type alias |  |
| [`ScanParams`](#scanparams) | type alias |  |
| [`ScanResults`](#scanresults) | type alias |  |
| [`FetchTitleParams`](#fetchtitleparams) | type alias |  |
| [`FetchTitleResults`](#fetchtitleresults) | type alias |  |
| [`ListAccountsParams`](#listaccountsparams) | type alias |  |
| [`ListAccountsResults`](#listaccountsresults) | type alias |  |
| [`LoginStartParams`](#loginstartparams) | type alias |  |
| [`LoginStartResults`](#loginstartresults) | type alias |  |
| [`LoginCompleteParams`](#logincompleteparams) | type alias |  |
| [`LoginCompleteResults`](#logincompleteresults) | type alias |  |
| [`SearchCatalogParams`](#searchcatalogparams) | type alias |  |
| [`SearchCatalogResults`](#searchcatalogresults) | type alias |  |
| [`ExpandCandidatesParams`](#expandcandidatesparams) | type alias |  |
| [`ExpandCandidatesResults`](#expandcandidatesresults) | type alias |  |
| [`PurchaseHintParams`](#purchasehintparams) | type alias |  |
| [`PurchaseHintResults`](#purchasehintresults) | type alias |  |
| [`ListDealsParams`](#listdealsparams) | type alias |  |
| [`ListDealsResults`](#listdealsresults) | type alias |  |
| [`HealthParams`](#healthparams) | type alias |  |
| [`HealthResults`](#healthresults) | type alias |  |
| [`DiagnoseParams`](#diagnoseparams) | type alias |  |
| [`DiagnoseResults`](#diagnoseresults) | type alias |  |
| [`CatalogDetailParams`](#catalogdetailparams) | type alias |  |
| [`CatalogDetailResults`](#catalogdetailresults) | type alias |  |
| [`RevokeAccountParams`](#revokeaccountparams) | type alias |  |
| [`RevokeAccountResults`](#revokeaccountresults) | type alias |  |

## Modules

- [`login_params`](content_source/login_params.md)
- [`login_results`](content_source/login_results.md)
- [`scan_params`](content_source/scan_params.md)
- [`scan_results`](content_source/scan_results.md)
- [`fetch_title_params`](content_source/fetch_title_params.md)
- [`fetch_title_results`](content_source/fetch_title_results.md)
- [`list_accounts_params`](content_source/list_accounts_params.md)
- [`list_accounts_results`](content_source/list_accounts_results.md)
- [`login_start_params`](content_source/login_start_params.md)
- [`login_start_results`](content_source/login_start_results.md)
- [`login_complete_params`](content_source/login_complete_params.md)
- [`login_complete_results`](content_source/login_complete_results.md)
- [`search_catalog_params`](content_source/search_catalog_params.md)
- [`search_catalog_results`](content_source/search_catalog_results.md)
- [`expand_candidates_params`](content_source/expand_candidates_params.md)
- [`expand_candidates_results`](content_source/expand_candidates_results.md)
- [`purchase_hint_params`](content_source/purchase_hint_params.md)
- [`purchase_hint_results`](content_source/purchase_hint_results.md)
- [`list_deals_params`](content_source/list_deals_params.md)
- [`list_deals_results`](content_source/list_deals_results.md)
- [`health_params`](content_source/health_params.md)
- [`health_results`](content_source/health_results.md)
- [`diagnose_params`](content_source/diagnose_params.md)
- [`diagnose_results`](content_source/diagnose_results.md)
- [`catalog_detail_params`](content_source/catalog_detail_params.md)
- [`catalog_detail_results`](content_source/catalog_detail_results.md)
- [`revoke_account_params`](content_source/revoke_account_params.md)
- [`revoke_account_results`](content_source/revoke_account_results.md)

## Traits

### Server

```rust
pub trait Server: 'static {
    fn login(
        self: Rc<Self>,
        _: LoginParams,
        _: LoginResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn scan(
        self: Rc<Self>,
        _: ScanParams,
        _: ScanResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn fetch_title(
        self: Rc<Self>,
        _: FetchTitleParams,
        _: FetchTitleResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn list_accounts(
        self: Rc<Self>,
        _: ListAccountsParams,
        _: ListAccountsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn login_start(
        self: Rc<Self>,
        _: LoginStartParams,
        _: LoginStartResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn login_complete(
        self: Rc<Self>,
        _: LoginCompleteParams,
        _: LoginCompleteResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn search_catalog(
        self: Rc<Self>,
        _: SearchCatalogParams,
        _: SearchCatalogResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn expand_candidates(
        self: Rc<Self>,
        _: ExpandCandidatesParams,
        _: ExpandCandidatesResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn purchase_hint(
        self: Rc<Self>,
        _: PurchaseHintParams,
        _: PurchaseHintResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn list_deals(
        self: Rc<Self>,
        _: ListDealsParams,
        _: ListDealsResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn health(
        self: Rc<Self>,
        _: HealthParams,
        _: HealthResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn diagnose(
        self: Rc<Self>,
        _: DiagnoseParams,
        _: DiagnoseResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn catalog_detail(
        self: Rc<Self>,
        _: CatalogDetailParams,
        _: CatalogDetailResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
    fn revoke_account(
        self: Rc<Self>,
        _: RevokeAccountParams,
        _: RevokeAccountResults,
    ) -> impl Future<Output = Result<(), Error>> + 'static;
}
```

#### Server::login

```rust
fn login(
    self: Rc<Self>,
    _: LoginParams,
    _: LoginResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::scan

```rust
fn scan(
    self: Rc<Self>,
    _: ScanParams,
    _: ScanResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::fetch_title

```rust
fn fetch_title(
    self: Rc<Self>,
    _: FetchTitleParams,
    _: FetchTitleResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::list_accounts

```rust
fn list_accounts(
    self: Rc<Self>,
    _: ListAccountsParams,
    _: ListAccountsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::login_start

```rust
fn login_start(
    self: Rc<Self>,
    _: LoginStartParams,
    _: LoginStartResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::login_complete

```rust
fn login_complete(
    self: Rc<Self>,
    _: LoginCompleteParams,
    _: LoginCompleteResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::search_catalog

```rust
fn search_catalog(
    self: Rc<Self>,
    _: SearchCatalogParams,
    _: SearchCatalogResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::expand_candidates

```rust
fn expand_candidates(
    self: Rc<Self>,
    _: ExpandCandidatesParams,
    _: ExpandCandidatesResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::purchase_hint

```rust
fn purchase_hint(
    self: Rc<Self>,
    _: PurchaseHintParams,
    _: PurchaseHintResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::list_deals

```rust
fn list_deals(
    self: Rc<Self>,
    _: ListDealsParams,
    _: ListDealsResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::health

```rust
fn health(
    self: Rc<Self>,
    _: HealthParams,
    _: HealthResults,
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

#### Server::catalog_detail

```rust
fn catalog_detail(
    self: Rc<Self>,
    _: CatalogDetailParams,
    _: CatalogDetailResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

#### Server::revoke_account

```rust
fn revoke_account(
    self: Rc<Self>,
    _: RevokeAccountParams,
    _: RevokeAccountResults,
) -> impl Future<Output = Result<(), Error>> + 'static
```

## Structs

### Client

```rust
pub struct Client {
    pub client: Client,
}
```

#### Client::login_request

```rust
pub fn login_request(&self) -> Request<Owned, Owned>
```

#### Client::scan_request

```rust
pub fn scan_request(&self) -> Request<Owned, Owned>
```

#### Client::fetch_title_request

```rust
pub fn fetch_title_request(&self) -> Request<Owned, Owned>
```

#### Client::list_accounts_request

```rust
pub fn list_accounts_request(&self) -> Request<Owned, Owned>
```

#### Client::login_start_request

```rust
pub fn login_start_request(&self) -> Request<Owned, Owned>
```

#### Client::login_complete_request

```rust
pub fn login_complete_request(&self) -> Request<Owned, Owned>
```

#### Client::search_catalog_request

```rust
pub fn search_catalog_request(&self) -> Request<Owned, Owned>
```

#### Client::expand_candidates_request

```rust
pub fn expand_candidates_request(&self) -> Request<Owned, Owned>
```

#### Client::purchase_hint_request

```rust
pub fn purchase_hint_request(&self) -> Request<Owned, Owned>
```

#### Client::list_deals_request

```rust
pub fn list_deals_request(&self) -> Request<Owned, Owned>
```

#### Client::health_request

```rust
pub fn health_request(&self) -> Request<Owned, Owned>
```

#### Client::diagnose_request

```rust
pub fn diagnose_request(&self) -> Request<Owned, Owned>
```

#### Client::catalog_detail_request

```rust
pub fn catalog_detail_request(&self) -> Request<Owned, Owned>
```

#### Client::revoke_account_request

```rust
pub fn revoke_account_request(&self) -> Request<Owned, Owned>
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

### LoginParams

```rust
pub type LoginParams = Params<Owned>;
```

### LoginResults

```rust
pub type LoginResults = Results<Owned>;
```

### ScanParams

```rust
pub type ScanParams = Params<Owned>;
```

### ScanResults

```rust
pub type ScanResults = Results<Owned>;
```

### FetchTitleParams

```rust
pub type FetchTitleParams = Params<Owned>;
```

### FetchTitleResults

```rust
pub type FetchTitleResults = Results<Owned>;
```

### ListAccountsParams

```rust
pub type ListAccountsParams = Params<Owned>;
```

### ListAccountsResults

```rust
pub type ListAccountsResults = Results<Owned>;
```

### LoginStartParams

```rust
pub type LoginStartParams = Params<Owned>;
```

### LoginStartResults

```rust
pub type LoginStartResults = Results<Owned>;
```

### LoginCompleteParams

```rust
pub type LoginCompleteParams = Params<Owned>;
```

### LoginCompleteResults

```rust
pub type LoginCompleteResults = Results<Owned>;
```

### SearchCatalogParams

```rust
pub type SearchCatalogParams = Params<Owned>;
```

### SearchCatalogResults

```rust
pub type SearchCatalogResults = Results<Owned>;
```

### ExpandCandidatesParams

```rust
pub type ExpandCandidatesParams = Params<Owned>;
```

### ExpandCandidatesResults

```rust
pub type ExpandCandidatesResults = Results<Owned>;
```

### PurchaseHintParams

```rust
pub type PurchaseHintParams = Params<Owned>;
```

### PurchaseHintResults

```rust
pub type PurchaseHintResults = Results<Owned>;
```

### ListDealsParams

```rust
pub type ListDealsParams = Params<Owned>;
```

### ListDealsResults

```rust
pub type ListDealsResults = Results<Owned>;
```

### HealthParams

```rust
pub type HealthParams = Params<Owned>;
```

### HealthResults

```rust
pub type HealthResults = Results<Owned>;
```

### DiagnoseParams

```rust
pub type DiagnoseParams = Params<Owned>;
```

### DiagnoseResults

```rust
pub type DiagnoseResults = Results<Owned>;
```

### CatalogDetailParams

```rust
pub type CatalogDetailParams = Params<Owned>;
```

### CatalogDetailResults

```rust
pub type CatalogDetailResults = Results<Owned>;
```

### RevokeAccountParams

```rust
pub type RevokeAccountParams = Params<Owned>;
```

### RevokeAccountResults

```rust
pub type RevokeAccountResults = Results<Owned>;
```
