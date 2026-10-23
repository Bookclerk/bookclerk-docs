**bookclerk_plugin_abi > plugin_capnp > event_consumer**

# Module: plugin_capnp::event_consumer

## Contents

**Modules**

- [`event_params`](#event_params)
- [`event_results`](#event_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`EventParams`](#eventparams)
- [`EventResults`](#eventresults)

---

## bookclerk_plugin_abi::plugin_capnp::event_consumer::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn event_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::event_consumer::event_params::Owned, crate::plugin_capnp::event_consumer::event_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::event_consumer::EventParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::event_consumer::event_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::event_consumer::EventResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::event_consumer::event_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::event_consumer::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::event_consumer::Server

*Trait*

**Methods:**

- `event`



## bookclerk_plugin_abi::plugin_capnp::event_consumer::ServerDispatch

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



## Module: event_params



## Module: event_results



