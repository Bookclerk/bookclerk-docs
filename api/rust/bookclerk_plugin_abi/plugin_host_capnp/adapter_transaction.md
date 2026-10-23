**bookclerk_plugin_abi > plugin_host_capnp > adapter_transaction**

# Module: plugin_host_capnp::adapter_transaction

## Contents

**Modules**

- [`assert_restore_constraints_params`](#assert_restore_constraints_params)
- [`assert_restore_constraints_results`](#assert_restore_constraints_results)
- [`commit_params`](#commit_params)
- [`commit_results`](#commit_results)
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
- [`rollback_params`](#rollback_params)
- [`rollback_results`](#rollback_results)

**Structs**

- [`Client`](#client)
- [`Owned`](#owned)
- [`ServerDispatch`](#serverdispatch)

**Traits**

- [`Server`](#server)

**Type Aliases**

- [`AssertRestoreConstraintsParams`](#assertrestoreconstraintsparams)
- [`AssertRestoreConstraintsResults`](#assertrestoreconstraintsresults)
- [`CommitParams`](#commitparams)
- [`CommitResults`](#commitresults)
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
- [`RollbackParams`](#rollbackparams)
- [`RollbackResults`](#rollbackresults)

---

## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::AssertRestoreConstraintsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::assert_restore_constraints_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::AssertRestoreConstraintsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::assert_restore_constraints_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::Client

*Struct*

**Fields:**
- `client: ::capnp::capability::Client`

**Methods:**

- `fn execute_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::execute_params::Owned, crate::plugin_host_capnp::adapter_transaction::execute_results::Owned>`
- `fn commit_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::commit_params::Owned, crate::plugin_host_capnp::adapter_transaction::commit_results::Owned>`
- `fn rollback_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::rollback_params::Owned, crate::plugin_host_capnp::adapter_transaction::rollback_results::Owned>`
- `fn export_identity_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::export_identity_params::Owned, crate::plugin_host_capnp::adapter_transaction::export_identity_results::Owned>`
- `fn import_identity_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::import_identity_params::Owned, crate::plugin_host_capnp::adapter_transaction::import_identity_results::Owned>`
- `fn list_user_relations_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::list_user_relations_params::Owned, crate::plugin_host_capnp::adapter_transaction::list_user_relations_results::Owned>`
- `fn prepare_unit_restore_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::prepare_unit_restore_params::Owned, crate::plugin_host_capnp::adapter_transaction::prepare_unit_restore_results::Owned>`
- `fn drop_user_relations_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::drop_user_relations_params::Owned, crate::plugin_host_capnp::adapter_transaction::drop_user_relations_results::Owned>`
- `fn assert_restore_constraints_request(self: &Self) -> ::capnp::capability::Request<crate::plugin_host_capnp::adapter_transaction::assert_restore_constraints_params::Owned, crate::plugin_host_capnp::adapter_transaction::assert_restore_constraints_results::Owned>`

**Traits:** HasTypeId

**Trait Implementations:**

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
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, _default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::CommitParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::commit_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::CommitResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::commit_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::DropUserRelationsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::drop_user_relations_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::DropUserRelationsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::drop_user_relations_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ExecuteParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::execute_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ExecuteResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::execute_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ExportIdentityParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::export_identity_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ExportIdentityResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::export_identity_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ImportIdentityParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::import_identity_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ImportIdentityResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::import_identity_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ListUserRelationsParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::list_user_relations_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ListUserRelationsResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::list_user_relations_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::PrepareUnitRestoreParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::prepare_unit_restore_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::PrepareUnitRestoreResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::prepare_unit_restore_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::RollbackParams

*Type Alias*: `::capnp::capability::Params<crate::plugin_host_capnp::adapter_transaction::rollback_params::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::RollbackResults

*Type Alias*: `::capnp::capability::Results<crate::plugin_host_capnp::adapter_transaction::rollback_results::Owned>`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::Server

*Trait*

**Methods:**

- `execute`
- `commit`
- `rollback`
- `export_identity`
- `import_identity`
- `list_user_relations`
- `prepare_unit_restore`
- `drop_user_relations`
- `assert_restore_constraints`



## bookclerk_plugin_abi::plugin_host_capnp::adapter_transaction::ServerDispatch

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



## Module: commit_params



## Module: commit_results



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



## Module: rollback_params



## Module: rollback_results



