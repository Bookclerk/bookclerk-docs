**bookclerk_plugin_abi > plugin_capnp > get_ok**

# Module: plugin_capnp::get_ok

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::get_ok::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_meta(self: Self) -> ::capnp::Result<crate::plugin_capnp::object_metadata::Builder<'a>>`
- `fn set_meta(self: & mut Self, value: crate::plugin_capnp::object_metadata::Reader) -> ::capnp::Result<()>`
- `fn init_meta(self: Self) -> crate::plugin_capnp::object_metadata::Builder<'a>`
- `fn has_meta(self: &Self) -> bool`
- `fn get_body(self: Self) -> ::capnp::Result<crate::plugin_capnp::byte_source::Client>`
- `fn set_body(self: & mut Self, value: crate::plugin_capnp::byte_source::Client)`
- `fn has_body(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::get_ok::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::get_ok::Pipeline

*Struct*

**Methods:**

- `fn get_meta(self: &Self) -> crate::plugin_capnp::object_metadata::Pipeline`
- `fn get_body(self: &Self) -> crate::plugin_capnp::byte_source::Client`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::get_ok::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_meta(self: Self) -> ::capnp::Result<crate::plugin_capnp::object_metadata::Reader<'a>>`
- `fn has_meta(self: &Self) -> bool`
- `fn get_body(self: Self) -> ::capnp::Result<crate::plugin_capnp::byte_source::Client>`
- `fn has_body(self: &Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`
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



