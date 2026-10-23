**bookclerk_plugin_abi > plugin_capnp > plugin_worker**

# Module: plugin_capnp::plugin_worker

## Contents

**Modules**

- [`database_migrations_params`](#database_migrations_params)
- [`database_migrations_results`](#database_migrations_results)
- [`describe_params`](#describe_params)
- [`describe_results`](#describe_results)
- [`open_params`](#open_params)
- [`open_results`](#open_results)
- [`shutdown_params`](#shutdown_params)
- [`shutdown_results`](#shutdown_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`DatabaseMigrationsParams`](#databasemigrationsparams)
- [`DatabaseMigrationsResults`](#databasemigrationsresults)
- [`DescribeParams`](#describeparams)
- [`DescribeResults`](#describeresults)
- [`OpenParams`](#openparams)
- [`OpenResults`](#openresults)
- [`ShutdownParams`](#shutdownparams)
- [`ShutdownResults`](#shutdownresults)

---

## bookclerk_plugin_abi::plugin_capnp::plugin_worker::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn describe_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::plugin_worker::describe_params::Owned, crate::plugin_capnp::plugin_worker::describe_results::Owned>`
- `fn open_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::plugin_worker::open_params::Owned, crate::plugin_capnp::plugin_worker::open_results::Owned>`
- `fn shutdown_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::plugin_worker::shutdown_params::Owned, crate::plugin_capnp::plugin_worker::shutdown_results::Owned>`
- `fn database_migrations_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::plugin_worker::database_migrations_params::Owned, crate::plugin_capnp::plugin_worker::database_migrations_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::DatabaseMigrationsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::plugin_worker::database_migrations_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::DatabaseMigrationsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::plugin_worker::database_migrations_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::DescribeParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::plugin_worker::describe_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::DescribeResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::plugin_worker::describe_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::OpenParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::plugin_worker::open_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::OpenResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::plugin_worker::open_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::Server

*Trait*

**Methods:**

- `describe`
- `open`
- `shutdown`
- `database_migrations`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::ServerDispatch

*Struct*

**Generic Parameters:**
- _T

**Fields:**
- `server: ::capnp::capability::Rc<_T>`

**Methods:**

- `fn dispatch_call_internal(this: ::capnp::capability::Rc<_T>, method_id: u16, params: ::capnp::capability::Params<::capnp::any_pointer::Owned>, results: ::capnp::capability::Results<::capnp::any_pointer::Owned>) -> ::capnp::capability::DispatchCallResult`

**Trait Implementations:**

- **Server**
  - `fn dispatch_call(self: Self, interface_id: u64, method_id: u16, params: ::capnp::capability::Params<::capnp::any_pointer::Owned>, results: ::capnp::capability::Results<::capnp::any_pointer::Owned>) -> ::capnp::capability::DispatchCallResult`
  - `fn as_ptr(self: &Self) -> usize`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **Deref**
  - `fn deref(self: &Self) -> &_T`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::ShutdownParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::plugin_worker::shutdown_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::plugin_worker::ShutdownResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::plugin_worker::shutdown_results::Owned>`



## Module: database_migrations_params



## Module: database_migrations_results



## Module: describe_params



## Module: describe_results



## Module: open_params



## Module: open_results



## Module: shutdown_params



## Module: shutdown_results



