**bookclerk_plugin_abi > plugin_capnp > adapter_database_session**

# Module: plugin_capnp::adapter_database_session

## Contents

**Modules**

- [`assert_restore_constraints_params`](#assert_restore_constraints_params)
- [`assert_restore_constraints_results`](#assert_restore_constraints_results)
- [`bootstrap_params`](#bootstrap_params)
- [`bootstrap_results`](#bootstrap_results)
- [`capabilities_params`](#capabilities_params)
- [`capabilities_results`](#capabilities_results)
- [`close_params`](#close_params)
- [`close_results`](#close_results)
- [`drop_user_relations_params`](#drop_user_relations_params)
- [`drop_user_relations_results`](#drop_user_relations_results)
- [`execute_params`](#execute_params)
- [`execute_results`](#execute_results)
- [`export_identity_params`](#export_identity_params)
- [`export_identity_results`](#export_identity_results)
- [`import_identity_params`](#import_identity_params)
- [`import_identity_results`](#import_identity_results)
- [`list_user_relations_params`](#list_user_relations_params)
- [`list_user_relations_results`](#list_user_relations_results)
- [`prepare_unit_restore_params`](#prepare_unit_restore_params)
- [`prepare_unit_restore_results`](#prepare_unit_restore_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`AssertRestoreConstraintsParams`](#assertrestoreconstraintsparams)
- [`AssertRestoreConstraintsResults`](#assertrestoreconstraintsresults)
- [`BootstrapParams`](#bootstrapparams)
- [`BootstrapResults`](#bootstrapresults)
- [`CapabilitiesParams`](#capabilitiesparams)
- [`CapabilitiesResults`](#capabilitiesresults)
- [`CloseParams`](#closeparams)
- [`CloseResults`](#closeresults)
- [`DropUserRelationsParams`](#dropuserrelationsparams)
- [`DropUserRelationsResults`](#dropuserrelationsresults)
- [`ExecuteParams`](#executeparams)
- [`ExecuteResults`](#executeresults)
- [`ExportIdentityParams`](#exportidentityparams)
- [`ExportIdentityResults`](#exportidentityresults)
- [`ImportIdentityParams`](#importidentityparams)
- [`ImportIdentityResults`](#importidentityresults)
- [`ListUserRelationsParams`](#listuserrelationsparams)
- [`ListUserRelationsResults`](#listuserrelationsresults)
- [`PrepareUnitRestoreParams`](#prepareunitrestoreparams)
- [`PrepareUnitRestoreResults`](#prepareunitrestoreresults)

---

## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::AssertRestoreConstraintsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::assert_restore_constraints_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::AssertRestoreConstraintsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::assert_restore_constraints_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::BootstrapParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::bootstrap_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::BootstrapResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::bootstrap_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::CapabilitiesParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::capabilities_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::CapabilitiesResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::capabilities_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn capabilities_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::capabilities_params::Owned, crate::plugin_capnp::adapter_database_session::capabilities_results::Owned>`
- `fn execute_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::execute_params::Owned, crate::plugin_capnp::adapter_database_session::execute_results::Owned>`
- `fn close_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::close_params::Owned, crate::plugin_capnp::adapter_database_session::close_results::Owned>`
- `fn bootstrap_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::bootstrap_params::Owned, crate::plugin_capnp::adapter_database_session::bootstrap_results::Owned>`
- `fn export_identity_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::export_identity_params::Owned, crate::plugin_capnp::adapter_database_session::export_identity_results::Owned>`
- `fn import_identity_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::import_identity_params::Owned, crate::plugin_capnp::adapter_database_session::import_identity_results::Owned>`
- `fn list_user_relations_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::list_user_relations_params::Owned, crate::plugin_capnp::adapter_database_session::list_user_relations_results::Owned>`
- `fn prepare_unit_restore_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::prepare_unit_restore_params::Owned, crate::plugin_capnp::adapter_database_session::prepare_unit_restore_results::Owned>`
- `fn drop_user_relations_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::drop_user_relations_params::Owned, crate::plugin_capnp::adapter_database_session::drop_user_relations_results::Owned>`
- `fn assert_restore_constraints_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_capnp::adapter_database_session::assert_restore_constraints_params::Owned, crate::plugin_capnp::adapter_database_session::assert_restore_constraints_results::Owned>`

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



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::CloseParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::close_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::CloseResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::close_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::DropUserRelationsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::drop_user_relations_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::DropUserRelationsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::drop_user_relations_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ExecuteParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::execute_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ExecuteResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::execute_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ExportIdentityParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::export_identity_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ExportIdentityResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::export_identity_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ImportIdentityParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::import_identity_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ImportIdentityResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::import_identity_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ListUserRelationsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::list_user_relations_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ListUserRelationsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::list_user_relations_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::PrepareUnitRestoreParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_capnp::adapter_database_session::prepare_unit_restore_params::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::PrepareUnitRestoreResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_capnp::adapter_database_session::prepare_unit_restore_results::Owned>`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::Server

*Trait*

**Methods:**

- `capabilities`
- `execute`
- `close`
- `bootstrap`
- `export_identity`
- `import_identity`
- `list_user_relations`
- `prepare_unit_restore`
- `drop_user_relations`
- `assert_restore_constraints`



## bookclerk_plugin_abi::plugin_capnp::adapter_database_session::ServerDispatch

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



## Module: assert_restore_constraints_params



## Module: assert_restore_constraints_results



## Module: bootstrap_params



## Module: bootstrap_results



## Module: capabilities_params



## Module: capabilities_results



## Module: close_params



## Module: close_results



## Module: drop_user_relations_params



## Module: drop_user_relations_results



## Module: execute_params



## Module: execute_results



## Module: export_identity_params



## Module: export_identity_results



## Module: import_identity_params



## Module: import_identity_results



## Module: list_user_relations_params



## Module: list_user_relations_results



## Module: prepare_unit_restore_params



## Module: prepare_unit_restore_results



