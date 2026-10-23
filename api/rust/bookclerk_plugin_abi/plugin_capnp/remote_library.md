**bookclerk_plugin_abi > plugin_capnp > remote_library**

# Module: plugin_capnp::remote_library

## Contents

**Modules**

- [`diagnose_params`](#diagnose_params)
- [`diagnose_results`](#diagnose_results)
- [`health_params`](#health_params)
- [`health_results`](#health_results)
- [`poll_events_params`](#poll_events_params)
- [`poll_events_results`](#poll_events_results)
- [`scan_library_params`](#scan_library_params)
- [`scan_library_results`](#scan_library_results)
- [`start_params`](#start_params)
- [`start_results`](#start_results)
- [`stop_params`](#stop_params)
- [`stop_results`](#stop_results)
- [`sync_listening_params`](#sync_listening_params)
- [`sync_listening_results`](#sync_listening_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`DiagnoseParams`](#diagnoseparams)
- [`DiagnoseResults`](#diagnoseresults)
- [`HealthParams`](#healthparams)
- [`HealthResults`](#healthresults)
- [`PollEventsParams`](#polleventsparams)
- [`PollEventsResults`](#polleventsresults)
- [`ScanLibraryParams`](#scanlibraryparams)
- [`ScanLibraryResults`](#scanlibraryresults)
- [`StartParams`](#startparams)
- [`StartResults`](#startresults)
- [`StopParams`](#stopparams)
- [`StopResults`](#stopresults)
- [`SyncListeningParams`](#synclisteningparams)
- [`SyncListeningResults`](#synclisteningresults)

---

## bookclerk_plugin_abi::plugin_capnp::remote_library::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn health_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::remote_library::health_params::Owned, crate::plugin_capnp::remote_library::health_results::Owned>`
- `fn start_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::remote_library::start_params::Owned, crate::plugin_capnp::remote_library::start_results::Owned>`
- `fn stop_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::remote_library::stop_params::Owned, crate::plugin_capnp::remote_library::stop_results::Owned>`
- `fn diagnose_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::remote_library::diagnose_params::Owned, crate::plugin_capnp::remote_library::diagnose_results::Owned>`
- `fn scan_library_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::remote_library::scan_library_params::Owned, crate::plugin_capnp::remote_library::scan_library_results::Owned>`
- `fn sync_listening_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::remote_library::sync_listening_params::Owned, crate::plugin_capnp::remote_library::sync_listening_results::Owned>`
- `fn poll_events_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::remote_library::poll_events_params::Owned, crate::plugin_capnp::remote_library::poll_events_results::Owned>`

**Traits:** HasTypeId

**Trait Implementations:**

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
- **FromServer**
  - `fn from_server(s: ::capnp::capability::Rc<_S>) -> ServerDispatch<_S>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::DiagnoseParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::remote_library::diagnose_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::DiagnoseResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::remote_library::diagnose_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::HealthParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::remote_library::health_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::HealthResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::remote_library::health_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::remote_library::PollEventsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::remote_library::poll_events_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::PollEventsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::remote_library::poll_events_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::ScanLibraryParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::remote_library::scan_library_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::ScanLibraryResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::remote_library::scan_library_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::Server

*Trait*

**Methods:**

- `health`
- `start`
- `stop`
- `diagnose`
- `scan_library`
- `sync_listening`
- `poll_events`



## bookclerk_plugin_abi::plugin_capnp::remote_library::ServerDispatch

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



## bookclerk_plugin_abi::plugin_capnp::remote_library::StartParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::remote_library::start_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::StartResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::remote_library::start_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::StopParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::remote_library::stop_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::StopResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::remote_library::stop_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::SyncListeningParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::remote_library::sync_listening_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::remote_library::SyncListeningResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::remote_library::sync_listening_results::Owned>`



## Module: diagnose_params



## Module: diagnose_results



## Module: health_params



## Module: health_results



## Module: poll_events_params



## Module: poll_events_results



## Module: scan_library_params



## Module: scan_library_results



## Module: start_params



## Module: start_results



## Module: stop_params



## Module: stop_results



## Module: sync_listening_params



## Module: sync_listening_results



