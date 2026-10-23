**bookclerk_plugin_abi > plugin_capnp > content_source**

# Module: plugin_capnp::content_source

## Contents

**Modules**

- [`catalog_detail_params`](#catalog_detail_params)
- [`catalog_detail_results`](#catalog_detail_results)
- [`diagnose_params`](#diagnose_params)
- [`diagnose_results`](#diagnose_results)
- [`expand_candidates_params`](#expand_candidates_params)
- [`expand_candidates_results`](#expand_candidates_results)
- [`fetch_title_params`](#fetch_title_params)
- [`fetch_title_results`](#fetch_title_results)
- [`health_params`](#health_params)
- [`health_results`](#health_results)
- [`list_accounts_params`](#list_accounts_params)
- [`list_accounts_results`](#list_accounts_results)
- [`list_deals_params`](#list_deals_params)
- [`list_deals_results`](#list_deals_results)
- [`login_complete_params`](#login_complete_params)
- [`login_complete_results`](#login_complete_results)
- [`login_params`](#login_params)
- [`login_results`](#login_results)
- [`login_start_params`](#login_start_params)
- [`login_start_results`](#login_start_results)
- [`purchase_hint_params`](#purchase_hint_params)
- [`purchase_hint_results`](#purchase_hint_results)
- [`scan_params`](#scan_params)
- [`scan_results`](#scan_results)
- [`search_catalog_params`](#search_catalog_params)
- [`search_catalog_results`](#search_catalog_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`CatalogDetailParams`](#catalogdetailparams)
- [`CatalogDetailResults`](#catalogdetailresults)
- [`DiagnoseParams`](#diagnoseparams)
- [`DiagnoseResults`](#diagnoseresults)
- [`ExpandCandidatesParams`](#expandcandidatesparams)
- [`ExpandCandidatesResults`](#expandcandidatesresults)
- [`FetchTitleParams`](#fetchtitleparams)
- [`FetchTitleResults`](#fetchtitleresults)
- [`HealthParams`](#healthparams)
- [`HealthResults`](#healthresults)
- [`ListAccountsParams`](#listaccountsparams)
- [`ListAccountsResults`](#listaccountsresults)
- [`ListDealsParams`](#listdealsparams)
- [`ListDealsResults`](#listdealsresults)
- [`LoginCompleteParams`](#logincompleteparams)
- [`LoginCompleteResults`](#logincompleteresults)
- [`LoginParams`](#loginparams)
- [`LoginResults`](#loginresults)
- [`LoginStartParams`](#loginstartparams)
- [`LoginStartResults`](#loginstartresults)
- [`PurchaseHintParams`](#purchasehintparams)
- [`PurchaseHintResults`](#purchasehintresults)
- [`ScanParams`](#scanparams)
- [`ScanResults`](#scanresults)
- [`SearchCatalogParams`](#searchcatalogparams)
- [`SearchCatalogResults`](#searchcatalogresults)

---

## bookclerk_plugin_abi::plugin_capnp::content_source::CatalogDetailParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::catalog_detail_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::CatalogDetailResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::catalog_detail_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn login_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::login_params::Owned, crate::plugin_capnp::content_source::login_results::Owned>`
- `fn scan_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::scan_params::Owned, crate::plugin_capnp::content_source::scan_results::Owned>`
- `fn fetch_title_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::fetch_title_params::Owned, crate::plugin_capnp::content_source::fetch_title_results::Owned>`
- `fn list_accounts_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::list_accounts_params::Owned, crate::plugin_capnp::content_source::list_accounts_results::Owned>`
- `fn login_start_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::login_start_params::Owned, crate::plugin_capnp::content_source::login_start_results::Owned>`
- `fn login_complete_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::login_complete_params::Owned, crate::plugin_capnp::content_source::login_complete_results::Owned>`
- `fn search_catalog_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::search_catalog_params::Owned, crate::plugin_capnp::content_source::search_catalog_results::Owned>`
- `fn expand_candidates_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::expand_candidates_params::Owned, crate::plugin_capnp::content_source::expand_candidates_results::Owned>`
- `fn purchase_hint_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::purchase_hint_params::Owned, crate::plugin_capnp::content_source::purchase_hint_results::Owned>`
- `fn list_deals_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::list_deals_params::Owned, crate::plugin_capnp::content_source::list_deals_results::Owned>`
- `fn health_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::health_params::Owned, crate::plugin_capnp::content_source::health_results::Owned>`
- `fn diagnose_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::diagnose_params::Owned, crate::plugin_capnp::content_source::diagnose_results::Owned>`
- `fn catalog_detail_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::content_source::catalog_detail_params::Owned, crate::plugin_capnp::content_source::catalog_detail_results::Owned>`

**Traits:** HasTypeId

**Trait Implementations:**

- **FromServer**
  - `fn from_server(s: ::capnp::capability::Rc<_S>) -> ServerDispatch<_S>`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, _default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, from: Self, _canonicalize: bool) -> ::capnp::Result<()>`
- **FromClientHook**
  - `fn new(hook: Box<::capnp::capability::DynClientHook>) -> Self`
  - `fn into_client_hook(self: Self) -> Box<::capnp::capability::DynClientHook>`
  - `fn as_client_hook(self: &Self) -> &::capnp::capability::DynClientHook`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **FromPointerBuilder**
  - `fn init_pointer(_builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



## bookclerk_plugin_abi::plugin_capnp::content_source::DiagnoseParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::diagnose_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::DiagnoseResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::diagnose_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ExpandCandidatesParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::expand_candidates_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ExpandCandidatesResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::expand_candidates_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::FetchTitleParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::fetch_title_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::FetchTitleResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::fetch_title_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::HealthParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::health_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::HealthResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::health_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ListAccountsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::list_accounts_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ListAccountsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::list_accounts_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ListDealsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::list_deals_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ListDealsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::list_deals_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::LoginCompleteParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::login_complete_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::LoginCompleteResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::login_complete_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::LoginParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::login_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::LoginResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::login_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::LoginStartParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::login_start_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::LoginStartResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::login_start_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::content_source::PurchaseHintParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::purchase_hint_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::PurchaseHintResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::purchase_hint_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ScanParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::scan_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::ScanResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::scan_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::SearchCatalogParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::content_source::search_catalog_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::SearchCatalogResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::content_source::search_catalog_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::content_source::Server

*Trait*

**Methods:**

- `login`
- `scan`
- `fetch_title`
- `list_accounts`
- `login_start`
- `login_complete`
- `search_catalog`
- `expand_candidates`
- `purchase_hint`
- `list_deals`
- `health`
- `diagnose`
- `catalog_detail`



## bookclerk_plugin_abi::plugin_capnp::content_source::ServerDispatch

*Struct*

**Generic Parameters:**
- _T

**Fields:**
- `server: ::capnp::capability::Rc<_T>`

**Methods:**

- `fn dispatch_call_internal(this: ::capnp::capability::Rc<_T>, method_id: u16, params: ::capnp::capability::Params<::capnp::any_pointer::Owned>, results: ::capnp::capability::Results<::capnp::any_pointer::Owned>) -> ::capnp::capability::DispatchCallResult`

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Self`
- **Deref**
  - `fn deref(self: &Self) -> &_T`
- **Server**
  - `fn dispatch_call(self: Self, interface_id: u64, method_id: u16, params: ::capnp::capability::Params<::capnp::any_pointer::Owned>, results: ::capnp::capability::Results<::capnp::any_pointer::Owned>) -> ::capnp::capability::DispatchCallResult`
  - `fn as_ptr(self: &Self) -> usize`



## Module: catalog_detail_params



## Module: catalog_detail_results



## Module: diagnose_params



## Module: diagnose_results



## Module: expand_candidates_params



## Module: expand_candidates_results



## Module: fetch_title_params



## Module: fetch_title_results



## Module: health_params



## Module: health_results



## Module: list_accounts_params



## Module: list_accounts_results



## Module: list_deals_params



## Module: list_deals_results



## Module: login_complete_params



## Module: login_complete_results



## Module: login_params



## Module: login_results



## Module: login_start_params



## Module: login_start_results



## Module: purchase_hint_params



## Module: purchase_hint_results



## Module: scan_params



## Module: scan_results



## Module: search_catalog_params



## Module: search_catalog_results



