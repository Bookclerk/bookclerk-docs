**bookclerk_plugin_abi > plugin_capnp > database**

# Module: plugin_capnp::database

## Contents

**Modules**

- [`drop_unit_params`](#drop_unit_params)
- [`drop_unit_results`](#drop_unit_results)
- [`open_session_params`](#open_session_params)
- [`open_session_results`](#open_session_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`DropUnitParams`](#dropunitparams)
- [`DropUnitResults`](#dropunitresults)
- [`OpenSessionParams`](#opensessionparams)
- [`OpenSessionResults`](#opensessionresults)

---

## bookclerk_plugin_abi::plugin_capnp::database::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn open_session_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::database::open_session_params::Owned, crate::plugin_capnp::database::open_session_results::Owned>`
- `fn drop_unit_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::database::drop_unit_params::Owned, crate::plugin_capnp::database::drop_unit_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::database::DropUnitParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::database::drop_unit_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::database::DropUnitResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::database::drop_unit_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::database::OpenSessionParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::database::open_session_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::database::OpenSessionResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::database::open_session_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::database::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::database::Server

*Trait*

**Methods:**

- `open_session`
- `drop_unit`



## bookclerk_plugin_abi::plugin_capnp::database::ServerDispatch

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



## Module: drop_unit_params



## Module: drop_unit_results



## Module: open_session_params



## Module: open_session_results



