**bookclerk_plugin_abi > plugin_capnp > adapter_execute_request**

# Module: plugin_capnp::adapter_execute_request

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::adapter_execute_request::Builder

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
- `fn get_statements(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::adapter_statement::Owned>>`
- `fn set_statements(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::adapter_statement::Owned>) -> ::capnp::Result<()>`
- `fn init_statements(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::adapter_statement::Owned>`
- `fn has_statements(self: &Self) -> bool`
- `fn get_deadline_unix_ms(self: Self) -> u64`
- `fn set_deadline_unix_ms(self: & mut Self, value: u64)`
- `fn get_isolation(self: Self) -> ::core::result::Result<crate::plugin_capnp::IsolationReq, ::capnp::NotInSchema>`
- `fn set_isolation(self: & mut Self, value: crate::plugin_capnp::IsolationReq)`
- `fn get_receipt(self: Self) -> ::capnp::Result<crate::plugin_capnp::adapter_receipt::Builder<'a>>`
- `fn set_receipt(self: & mut Self, value: crate::plugin_capnp::adapter_receipt::Reader) -> ::capnp::Result<()>`
- `fn init_receipt(self: Self) -> crate::plugin_capnp::adapter_receipt::Builder<'a>`
- `fn has_receipt(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



## bookclerk_plugin_abi::plugin_capnp::adapter_execute_request::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::adapter_execute_request::Pipeline

*Struct*

**Methods:**

- `fn get_receipt(self: &Self) -> crate::plugin_capnp::adapter_receipt::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::adapter_execute_request::Reader

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
- `fn get_statements(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::adapter_statement::Owned>>`
- `fn has_statements(self: &Self) -> bool`
- `fn get_deadline_unix_ms(self: Self) -> u64`
- `fn get_isolation(self: Self) -> ::core::result::Result<crate::plugin_capnp::IsolationReq, ::capnp::NotInSchema>`
- `fn get_receipt(self: Self) -> ::capnp::Result<crate::plugin_capnp::adapter_receipt::Reader<'a>>`
- `fn has_receipt(self: &Self) -> bool`

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



