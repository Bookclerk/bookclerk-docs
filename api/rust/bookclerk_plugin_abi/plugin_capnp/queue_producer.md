**bookclerk_plugin_abi > plugin_capnp > queue_producer**

# Module: plugin_capnp::queue_producer

## Contents

**Modules**

- [`send_batch_params`](#send_batch_params)
- [`send_batch_results`](#send_batch_results)
- [`send_params`](#send_params)
- [`send_results`](#send_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`SendBatchParams`](#sendbatchparams)
- [`SendBatchResults`](#sendbatchresults)
- [`SendParams`](#sendparams)
- [`SendResults`](#sendresults)

---

## bookclerk_plugin_abi::plugin_capnp::queue_producer::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn send_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::queue_producer::send_params::Owned, crate::plugin_capnp::queue_producer::send_results::Owned>`
- `fn send_batch_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::queue_producer::send_batch_params::Owned, crate::plugin_capnp::queue_producer::send_batch_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::queue_producer::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::queue_producer::SendBatchParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::queue_producer::send_batch_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::queue_producer::SendBatchResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::queue_producer::send_batch_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::queue_producer::SendParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::queue_producer::send_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::queue_producer::SendResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::queue_producer::send_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::queue_producer::Server

*Trait*

**Methods:**

- `send`
- `send_batch`



## bookclerk_plugin_abi::plugin_capnp::queue_producer::ServerDispatch

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



## Module: send_batch_params



## Module: send_batch_results



## Module: send_params



## Module: send_results



