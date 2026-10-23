**bookclerk_plugin_abi > plugin_capnp > db_statement**

# Module: plugin_capnp::db_statement

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::db_statement::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_sql(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_sql<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_sql(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_sql(self: &Self) -> bool`
- `fn get_parameters(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::db_value::Owned>>`
- `fn set_parameters(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::db_value::Owned>) -> ::capnp::Result<()>`
- `fn init_parameters(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::db_value::Owned>`
- `fn has_parameters(self: &Self) -> bool`
- `fn get_kind(self: Self) -> ::core::result::Result<crate::plugin_capnp::DbStatementKind, ::capnp::NotInSchema>`
- `fn set_kind(self: & mut Self, value: crate::plugin_capnp::DbStatementKind)`
- `fn get_max_rows(self: Self) -> u32`
- `fn set_max_rows(self: & mut Self, value: u32)`
- `fn get_result_selection(self: Self) -> ::core::result::Result<crate::plugin_capnp::DbResultSelection, ::capnp::NotInSchema>`
- `fn set_result_selection(self: & mut Self, value: crate::plugin_capnp::DbResultSelection)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::db_statement::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::db_statement::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::db_statement::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_sql(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_sql(self: &Self) -> bool`
- `fn get_parameters(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::db_value::Owned>>`
- `fn has_parameters(self: &Self) -> bool`
- `fn get_kind(self: Self) -> ::core::result::Result<crate::plugin_capnp::DbStatementKind, ::capnp::NotInSchema>`
- `fn get_max_rows(self: Self) -> u32`
- `fn get_result_selection(self: Self) -> ::core::result::Result<crate::plugin_capnp::DbResultSelection, ::capnp::NotInSchema>`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`



