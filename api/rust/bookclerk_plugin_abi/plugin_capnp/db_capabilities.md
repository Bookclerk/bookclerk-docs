**bookclerk_plugin_abi > plugin_capnp > db_capabilities**

# Module: plugin_capnp::db_capabilities

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::db_capabilities::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_sql_contract_version(self: Self) -> u32`
- `fn set_sql_contract_version(self: & mut Self, value: u32)`
- `fn get_atomic_batch(self: Self) -> bool`
- `fn set_atomic_batch(self: & mut Self, value: bool)`
- `fn get_returning(self: Self) -> bool`
- `fn set_returning(self: & mut Self, value: bool)`
- `fn get_affected_rows(self: Self) -> bool`
- `fn set_affected_rows(self: & mut Self, value: bool)`
- `fn get_schema_migrations(self: Self) -> bool`
- `fn set_schema_migrations(self: & mut Self, value: bool)`
- `fn get_cancellation(self: Self) -> bool`
- `fn set_cancellation(self: & mut Self, value: bool)`
- `fn get_timing(self: Self) -> bool`
- `fn set_timing(self: & mut Self, value: bool)`
- `fn get_max_binds(self: Self) -> u32`
- `fn set_max_binds(self: & mut Self, value: u32)`
- `fn get_max_statements(self: Self) -> u32`
- `fn set_max_statements(self: & mut Self, value: u32)`
- `fn get_max_result_rows(self: Self) -> u32`
- `fn set_max_result_rows(self: & mut Self, value: u32)`
- `fn get_max_payload_bytes(self: Self) -> u32`
- `fn set_max_payload_bytes(self: & mut Self, value: u32)`
- `fn get_max_result_bytes(self: Self) -> u32`
- `fn set_max_result_bytes(self: & mut Self, value: u32)`
- `fn get_max_cell_bytes(self: Self) -> u32`
- `fn set_max_cell_bytes(self: & mut Self, value: u32)`
- `fn get_max_request_bytes(self: Self) -> u32`
- `fn set_max_request_bytes(self: & mut Self, value: u32)`
- `fn get_max_atomic_result_bytes(self: Self) -> u32`
- `fn set_max_atomic_result_bytes(self: & mut Self, value: u32)`
- `fn get_plugin_databases(self: Self) -> bool`
- `fn set_plugin_databases(self: & mut Self, value: bool)`
- `fn get_max_function_args(self: Self) -> u32`
- `fn set_max_function_args(self: & mut Self, value: u32)`
- `fn get_max_schema_columns(self: Self) -> u32`
- `fn set_max_schema_columns(self: & mut Self, value: u32)`
- `fn get_max_pattern_bytes(self: Self) -> u32`
- `fn set_max_pattern_bytes(self: & mut Self, value: u32)`
- `fn get_max_lowered_statement_bytes(self: Self) -> u32`
- `fn set_max_lowered_statement_bytes(self: & mut Self, value: u32)`
- `fn get_consistent_backup_read(self: Self) -> bool`
- `fn set_consistent_backup_read(self: & mut Self, value: bool)`
- `fn get_atomic_unit_restore(self: Self) -> bool`
- `fn set_atomic_unit_restore(self: & mut Self, value: bool)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::db_capabilities::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::db_capabilities::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::db_capabilities::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_sql_contract_version(self: Self) -> u32`
- `fn get_atomic_batch(self: Self) -> bool`
- `fn get_returning(self: Self) -> bool`
- `fn get_affected_rows(self: Self) -> bool`
- `fn get_schema_migrations(self: Self) -> bool`
- `fn get_cancellation(self: Self) -> bool`
- `fn get_timing(self: Self) -> bool`
- `fn get_max_binds(self: Self) -> u32`
- `fn get_max_statements(self: Self) -> u32`
- `fn get_max_result_rows(self: Self) -> u32`
- `fn get_max_payload_bytes(self: Self) -> u32`
- `fn get_max_result_bytes(self: Self) -> u32`
- `fn get_max_cell_bytes(self: Self) -> u32`
- `fn get_max_request_bytes(self: Self) -> u32`
- `fn get_max_atomic_result_bytes(self: Self) -> u32`
- `fn get_plugin_databases(self: Self) -> bool`
- `fn get_max_function_args(self: Self) -> u32`
- `fn get_max_schema_columns(self: Self) -> u32`
- `fn get_max_pattern_bytes(self: Self) -> u32`
- `fn get_max_lowered_statement_bytes(self: Self) -> u32`
- `fn get_consistent_backup_read(self: Self) -> bool`
- `fn get_atomic_unit_restore(self: Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`



