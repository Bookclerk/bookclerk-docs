**bookclerk_plugin_abi > plugin_capnp > execute_request**

# Module: plugin_capnp::execute_request

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::execute_request::Builder

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
- `fn get_request_hash(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_request_hash<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_request_hash(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_request_hash(self: &Self) -> bool`
- `fn get_statements(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::db_statement::Owned>>`
- `fn set_statements(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::db_statement::Owned>) -> ::capnp::Result<()>`
- `fn init_statements(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::db_statement::Owned>`
- `fn has_statements(self: &Self) -> bool`
- `fn get_deadline_unix_ms(self: Self) -> u64`
- `fn set_deadline_unix_ms(self: & mut Self, value: u64)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::execute_request::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::execute_request::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::execute_request::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_operation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_operation_id(self: &Self) -> bool`
- `fn get_request_hash(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_request_hash(self: &Self) -> bool`
- `fn get_statements(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::db_statement::Owned>>`
- `fn has_statements(self: &Self) -> bool`
- `fn get_deadline_unix_ms(self: Self) -> u64`

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
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`



