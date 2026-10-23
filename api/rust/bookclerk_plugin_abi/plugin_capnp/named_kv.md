**bookclerk_plugin_abi > plugin_capnp > named_kv**

# Module: plugin_capnp::named_kv

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::named_kv::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_name(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_name<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_name(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_name(self: &Self) -> bool`
- `fn get_namespace(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_namespace<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_namespace(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_namespace(self: &Self) -> bool`
- `fn get_kv(self: Self) -> ::capnp::Result<crate::plugin_capnp::kv_namespace::Client>`
- `fn set_kv(self: & mut Self, value: crate::plugin_capnp::kv_namespace::Client)`
- `fn has_kv(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::named_kv::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::named_kv::Pipeline

*Struct*

**Methods:**

- `fn get_kv(self: &Self) -> crate::plugin_capnp::kv_namespace::Client`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::named_kv::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_name(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_name(self: &Self) -> bool`
- `fn get_namespace(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_namespace(self: &Self) -> bool`
- `fn get_kv(self: Self) -> ::capnp::Result<crate::plugin_capnp::kv_namespace::Client>`
- `fn has_kv(self: &Self) -> bool`

**Traits:** HasTypeId, Copy

**Trait Implementations:**

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
- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



