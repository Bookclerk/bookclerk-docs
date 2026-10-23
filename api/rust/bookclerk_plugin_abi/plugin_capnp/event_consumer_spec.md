**bookclerk_plugin_abi > plugin_capnp > event_consumer_spec**

# Module: plugin_capnp::event_consumer_spec

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::event_consumer_spec::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_type(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_event_type<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_event_type(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_event_type(self: &Self) -> bool`
- `fn get_schema_versions(self: Self) -> ::capnp::Result<::capnp::primitive_list::Builder<'a, u32>>`
- `fn set_schema_versions<impl ::capnp::traits::SetterInput<::capnp::primitive_list::Owned<u32>>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_schema_versions(self: Self, size: u32) -> ::capnp::primitive_list::Builder<'a, u32>`
- `fn has_schema_versions(self: &Self) -> bool`
- `fn get_supports_suspend(self: Self) -> bool`
- `fn set_supports_suspend(self: & mut Self, value: bool)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::event_consumer_spec::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** OwnedStruct, Owned, Pipelined, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::event_consumer_spec::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::event_consumer_spec::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_type(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_event_type(self: &Self) -> bool`
- `fn get_schema_versions(self: Self) -> ::capnp::Result<::capnp::primitive_list::Reader<'a, u32>>`
- `fn has_schema_versions(self: &Self) -> bool`
- `fn get_supports_suspend(self: Self) -> bool`

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



