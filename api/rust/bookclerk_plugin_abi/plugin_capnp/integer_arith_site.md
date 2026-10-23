**bookclerk_plugin_abi > plugin_capnp > integer_arith_site**

# Module: plugin_capnp::integer_arith_site

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::integer_arith_site::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_full(self: Self) -> ::capnp::Result<crate::plugin_capnp::sql_span::Builder<'a>>`
- `fn set_full(self: & mut Self, value: crate::plugin_capnp::sql_span::Reader) -> ::capnp::Result<()>`
- `fn init_full(self: Self) -> crate::plugin_capnp::sql_span::Builder<'a>`
- `fn has_full(self: &Self) -> bool`
- `fn get_lhs(self: Self) -> ::capnp::Result<crate::plugin_capnp::sql_span::Builder<'a>>`
- `fn set_lhs(self: & mut Self, value: crate::plugin_capnp::sql_span::Reader) -> ::capnp::Result<()>`
- `fn init_lhs(self: Self) -> crate::plugin_capnp::sql_span::Builder<'a>`
- `fn has_lhs(self: &Self) -> bool`
- `fn get_rhs(self: Self) -> ::capnp::Result<crate::plugin_capnp::sql_span::Builder<'a>>`
- `fn set_rhs(self: & mut Self, value: crate::plugin_capnp::sql_span::Reader) -> ::capnp::Result<()>`
- `fn init_rhs(self: Self) -> crate::plugin_capnp::sql_span::Builder<'a>`
- `fn has_rhs(self: &Self) -> bool`
- `fn get_kind(self: Self) -> ::core::result::Result<crate::plugin_capnp::IntegerArithKind, ::capnp::NotInSchema>`
- `fn set_kind(self: & mut Self, value: crate::plugin_capnp::IntegerArithKind)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::integer_arith_site::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::integer_arith_site::Pipeline

*Struct*

**Methods:**

- `fn get_full(self: &Self) -> crate::plugin_capnp::sql_span::Pipeline`
- `fn get_lhs(self: &Self) -> crate::plugin_capnp::sql_span::Pipeline`
- `fn get_rhs(self: &Self) -> crate::plugin_capnp::sql_span::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::integer_arith_site::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_full(self: Self) -> ::capnp::Result<crate::plugin_capnp::sql_span::Reader<'a>>`
- `fn has_full(self: &Self) -> bool`
- `fn get_lhs(self: Self) -> ::capnp::Result<crate::plugin_capnp::sql_span::Reader<'a>>`
- `fn has_lhs(self: &Self) -> bool`
- `fn get_rhs(self: Self) -> ::capnp::Result<crate::plugin_capnp::sql_span::Reader<'a>>`
- `fn has_rhs(self: &Self) -> bool`
- `fn get_kind(self: Self) -> ::core::result::Result<crate::plugin_capnp::IntegerArithKind, ::capnp::NotInSchema>`

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



