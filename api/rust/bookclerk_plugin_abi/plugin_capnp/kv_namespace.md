**bookclerk_plugin_abi > plugin_capnp > kv_namespace**

# Module: plugin_capnp::kv_namespace

## Contents

**Modules**

- [`delete_params`](#delete_params)
- [`delete_results`](#delete_results)
- [`get_params`](#get_params)
- [`get_results`](#get_results)
- [`list_params`](#list_params)
- [`list_results`](#list_results)
- [`put_params`](#put_params)
- [`put_results`](#put_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`DeleteParams`](#deleteparams)
- [`DeleteResults`](#deleteresults)
- [`GetParams`](#getparams)
- [`GetResults`](#getresults)
- [`ListParams`](#listparams)
- [`ListResults`](#listresults)
- [`PutParams`](#putparams)
- [`PutResults`](#putresults)

---

## bookclerk_plugin_abi::plugin_capnp::kv_namespace::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn get_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::kv_namespace::get_params::Owned, crate::plugin_capnp::kv_namespace::get_results::Owned>`
- `fn put_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::kv_namespace::put_params::Owned, crate::plugin_capnp::kv_namespace::put_results::Owned>`
- `fn delete_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::kv_namespace::delete_params::Owned, crate::plugin_capnp::kv_namespace::delete_results::Owned>`
- `fn list_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::kv_namespace::list_params::Owned, crate::plugin_capnp::kv_namespace::list_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::DeleteParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::kv_namespace::delete_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::DeleteResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::kv_namespace::delete_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::GetParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::kv_namespace::get_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::GetResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::kv_namespace::get_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::ListParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::kv_namespace::list_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::ListResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::kv_namespace::list_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::PutParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::kv_namespace::put_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::PutResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::kv_namespace::put_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::Server

*Trait*

**Methods:**

- `get`
- `put`
- `delete`
- `list`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::ServerDispatch

*Struct*

**Generic Parameters:**
- _T

**Fields:**
- `server: ::capnp::capability::Rc<_T>`

**Methods:**

- `fn dispatch_call_internal(this: ::capnp::capability::Rc<_T>, method_id: u16, params: ::capnp::capability::Params<::capnp::any_pointer::Owned>, results: ::capnp::capability::Results<::capnp::any_pointer::Owned>) -> ::capnp::capability::DispatchCallResult`

**Trait Implementations:**

- **Deref**
  - `fn deref(self: &Self) -> &_T`
- **Server**
  - `fn dispatch_call(self: Self, interface_id: u64, method_id: u16, params: ::capnp::capability::Params<::capnp::any_pointer::Owned>, results: ::capnp::capability::Results<::capnp::any_pointer::Owned>) -> ::capnp::capability::DispatchCallResult`
  - `fn as_ptr(self: &Self) -> usize`
- **Clone**
  - `fn clone(self: &Self) -> Self`



## Module: delete_params



## Module: delete_results



## Module: get_params



## Module: get_results



## Module: list_params



## Module: list_results



## Module: put_params



## Module: put_results



