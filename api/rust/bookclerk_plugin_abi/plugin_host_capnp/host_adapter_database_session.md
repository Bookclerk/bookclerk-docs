**bookclerk_plugin_abi > plugin_host_capnp > host_adapter_database_session**

# Module: plugin_host_capnp::host_adapter_database_session

## Contents

**Modules**

- [`begin_params`](#begin_params)
- [`begin_results`](#begin_results)
- [`execute_params`](#execute_params)
- [`execute_results`](#execute_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`BeginParams`](#beginparams)
- [`BeginResults`](#beginresults)
- [`ExecuteParams`](#executeparams)
- [`ExecuteResults`](#executeresults)

---

## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::BeginParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::host_adapter_database_session::begin_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::BeginResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::host_adapter_database_session::begin_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn begin_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::host_adapter_database_session::begin_params::Owned, crate::plugin_host_capnp::host_adapter_database_session::begin_results::Owned>`
- `fn execute_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::host_adapter_database_session::execute_params::Owned, crate::plugin_host_capnp::host_adapter_database_session::execute_results::Owned>`

**Traits:** HasTypeId

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Self`
- **FromPointerBuilder**
  - `fn init_pointer(_builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
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



## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::ExecuteParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::host_adapter_database_session::execute_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::ExecuteResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::host_adapter_database_session::execute_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::Server

*Trait*

**Methods:**

- `begin`
- `execute`



## bookclerk_plugin_abi::plugin_host_capnp::host_adapter_database_session::ServerDispatch

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



## Module: begin_params



## Module: begin_results



## Module: execute_params



## Module: execute_results



