**bookclerk_plugin_abi > plugin_capnp > plugin_cli**

# Module: plugin_capnp::plugin_cli

## Contents

**Modules**

- [`describe_params`](#describe_params)
- [`describe_results`](#describe_results)
- [`invoke_params`](#invoke_params)
- [`invoke_results`](#invoke_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`DescribeParams`](#describeparams)
- [`DescribeResults`](#describeresults)
- [`InvokeParams`](#invokeparams)
- [`InvokeResults`](#invokeresults)

---

## bookclerk_plugin_abi::plugin_capnp::plugin_cli::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn describe_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::plugin_cli::describe_params::Owned, crate::plugin_capnp::plugin_cli::describe_results::Owned>`
- `fn invoke_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::plugin_cli::invoke_params::Owned, crate::plugin_capnp::plugin_cli::invoke_results::Owned>`

**Traits:** HasTypeId

**Trait Implementations:**

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
- **Clone**
  - `fn clone(self: &Self) -> Self`



## bookclerk_plugin_abi::plugin_capnp::plugin_cli::DescribeParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::plugin_cli::describe_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_cli::DescribeResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::plugin_cli::describe_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_cli::InvokeParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::plugin_cli::invoke_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_cli::InvokeResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::plugin_cli::invoke_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_cli::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, Pipelined, Owned

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::plugin_cli::Server

*Trait*

**Methods:**

- `describe`
- `invoke`



## bookclerk_plugin_abi::plugin_capnp::plugin_cli::ServerDispatch

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



## Module: describe_params



## Module: describe_results



## Module: invoke_params



## Module: invoke_results



