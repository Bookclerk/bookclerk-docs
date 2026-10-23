**bookclerk_plugin_abi > plugin_capnp > oidc**

# Module: plugin_capnp::oidc

## Contents

**Modules**

- [`authenticate_user_params`](#authenticate_user_params)
- [`authenticate_user_results`](#authenticate_user_results)
- [`clients_params`](#clients_params)
- [`clients_results`](#clients_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`AuthenticateUserParams`](#authenticateuserparams)
- [`AuthenticateUserResults`](#authenticateuserresults)
- [`ClientsParams`](#clientsparams)
- [`ClientsResults`](#clientsresults)

---

## bookclerk_plugin_abi::plugin_capnp::oidc::AuthenticateUserParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::oidc::authenticate_user_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::oidc::AuthenticateUserResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::oidc::authenticate_user_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::oidc::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn clients_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::oidc::clients_params::Owned, crate::plugin_capnp::oidc::clients_results::Owned>`
- `fn authenticate_user_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::oidc::authenticate_user_params::Owned, crate::plugin_capnp::oidc::authenticate_user_results::Owned>`

**Traits:** HasTypeId

**Trait Implementations:**

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
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, _default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, from: Self, _canonicalize: bool) -> ::capnp::Result<()>`



## bookclerk_plugin_abi::plugin_capnp::oidc::ClientsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::oidc::clients_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::oidc::ClientsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::oidc::clients_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::oidc::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::oidc::Server

*Trait*

**Methods:**

- `clients`
- `authenticate_user`



## bookclerk_plugin_abi::plugin_capnp::oidc::ServerDispatch

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



## Module: authenticate_user_params



## Module: authenticate_user_results



## Module: clients_params



## Module: clients_results



