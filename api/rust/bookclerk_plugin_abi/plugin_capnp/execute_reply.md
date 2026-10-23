**bookclerk_plugin_abi > plugin_capnp > execute_reply**

# Module: plugin_capnp::execute_reply

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::execute_reply::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_operation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_operation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_operation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_operation_id(self: &Self) -> bool`
- `fn get_statements(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::statement_result::Owned>>`
- `fn set_statements(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::statement_result::Owned>) -> ::capnp::Result<()>`
- `fn init_statements(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::statement_result::Owned>`
- `fn has_statements(self: &Self) -> bool`
- `fn get_timing(self: Self) -> ::capnp::Result<crate::plugin_capnp::db_timing::Builder<'a>>`
- `fn set_timing(self: & mut Self, value: crate::plugin_capnp::db_timing::Reader) -> ::capnp::Result<()>`
- `fn init_timing(self: Self) -> crate::plugin_capnp::db_timing::Builder<'a>`
- `fn has_timing(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::execute_reply::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::execute_reply::Pipeline

*Struct*

**Methods:**

- `fn get_timing(self: &Self) -> crate::plugin_capnp::db_timing::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::execute_reply::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_operation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_operation_id(self: &Self) -> bool`
- `fn get_statements(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::statement_result::Owned>>`
- `fn has_statements(self: &Self) -> bool`
- `fn get_timing(self: Self) -> ::capnp::Result<crate::plugin_capnp::db_timing::Reader<'a>>`
- `fn has_timing(self: &Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`
- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`



