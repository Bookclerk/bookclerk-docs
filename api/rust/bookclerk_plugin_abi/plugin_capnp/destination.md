**bookclerk_plugin_abi > plugin_capnp > destination**

# Module: plugin_capnp::destination

## Contents

**Modules**

- [`abort_stage_params`](#abort_stage_params)
- [`abort_stage_results`](#abort_stage_results)
- [`commit_params`](#commit_params)
- [`commit_results`](#commit_results)
- [`copy_params`](#copy_params)
- [`copy_results`](#copy_results)
- [`delete_params`](#delete_params)
- [`delete_results`](#delete_results)
- [`get_params`](#get_params)
- [`get_results`](#get_results)
- [`head_params`](#head_params)
- [`head_results`](#head_results)
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

- [`AbortStageParams`](#abortstageparams)
- [`AbortStageResults`](#abortstageresults)
- [`CommitParams`](#commitparams)
- [`CommitResults`](#commitresults)
- [`CopyParams`](#copyparams)
- [`CopyResults`](#copyresults)
- [`DeleteParams`](#deleteparams)
- [`DeleteResults`](#deleteresults)
- [`GetParams`](#getparams)
- [`GetResults`](#getresults)
- [`HeadParams`](#headparams)
- [`HeadResults`](#headresults)
- [`ListParams`](#listparams)
- [`ListResults`](#listresults)
- [`PutParams`](#putparams)
- [`PutResults`](#putresults)

---

## bookclerk_plugin_abi::plugin_capnp::destination::AbortStageParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::abort_stage_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::AbortStageResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::abort_stage_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn head_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::head_params::Owned, crate::plugin_capnp::destination::head_results::Owned>`
- `fn list_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::list_params::Owned, crate::plugin_capnp::destination::list_results::Owned>`
- `fn get_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::get_params::Owned, crate::plugin_capnp::destination::get_results::Owned>`
- `fn put_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::put_params::Owned, crate::plugin_capnp::destination::put_results::Owned>`
- `fn copy_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::copy_params::Owned, crate::plugin_capnp::destination::copy_results::Owned>`
- `fn delete_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::delete_params::Owned, crate::plugin_capnp::destination::delete_results::Owned>`
- `fn commit_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::commit_params::Owned, crate::plugin_capnp::destination::commit_results::Owned>`
- `fn abort_stage_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::destination::abort_stage_params::Owned, crate::plugin_capnp::destination::abort_stage_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::destination::CommitParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::commit_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::CommitResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::commit_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::CopyParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::copy_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::CopyResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::copy_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::DeleteParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::delete_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::DeleteResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::delete_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::GetParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::get_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::GetResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::get_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::HeadParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::head_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::HeadResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::head_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::ListParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::list_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::ListResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::list_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Pipelined, Owned, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::destination::PutParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::destination::put_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::PutResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::destination::put_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::destination::Server

*Trait*

**Methods:**

- `head`
- `list`
- `get`
- `put`
- `copy`
- `delete`
- `commit`
- `abort_stage`



## bookclerk_plugin_abi::plugin_capnp::destination::ServerDispatch

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



## Module: abort_stage_params



## Module: abort_stage_results



## Module: commit_params



## Module: commit_results



## Module: copy_params



## Module: copy_results



## Module: delete_params



## Module: delete_results



## Module: get_params



## Module: get_results



## Module: head_params



## Module: head_results



## Module: list_params



## Module: list_results



## Module: put_params



## Module: put_results



