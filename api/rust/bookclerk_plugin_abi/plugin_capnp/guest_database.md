**bookclerk_plugin_abi > plugin_capnp > guest_database**

# Module: plugin_capnp::guest_database

## Contents

**Modules**

- [`close_params`](#close_params)
- [`close_results`](#close_results)
- [`execute_params`](#execute_params)
- [`execute_results`](#execute_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`CloseParams`](#closeparams)
- [`CloseResults`](#closeresults)
- [`ExecuteParams`](#executeparams)
- [`ExecuteResults`](#executeresults)

---

## bookclerk_plugin_abi::plugin_capnp::guest_database::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn execute_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::guest_database::execute_params::Owned, crate::plugin_capnp::guest_database::execute_results::Owned>`
- `fn close_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::guest_database::close_params::Owned, crate::plugin_capnp::guest_database::close_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::guest_database::CloseParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::guest_database::close_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::guest_database::CloseResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::guest_database::close_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::guest_database::ExecuteParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::guest_database::execute_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::guest_database::ExecuteResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::guest_database::execute_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::guest_database::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::guest_database::Server

*Trait*

**Methods:**

- `execute`
- `close`



## bookclerk_plugin_abi::plugin_capnp::guest_database::ServerDispatch

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



## Module: close_params



## Module: close_results



## Module: execute_params



## Module: execute_results



