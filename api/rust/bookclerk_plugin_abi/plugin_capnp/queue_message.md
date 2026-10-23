**bookclerk_plugin_abi > plugin_capnp > queue_message**

# Module: plugin_capnp::queue_message

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::queue_message::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_id(self: &Self) -> bool`
- `fn get_timestamp_unix_ms(self: Self) -> u64`
- `fn set_timestamp_unix_ms(self: & mut Self, value: u64)`
- `fn get_body(self: Self) -> ::capnp::Result<::capnp::data::Builder<'a>>`
- `fn set_body(self: & mut Self, value: ::capnp::data::Reader)`
- `fn init_body(self: Self, size: u32) -> ::capnp::data::Builder<'a>`
- `fn has_body(self: &Self) -> bool`
- `fn get_content_type(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_content_type<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_content_type(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_content_type(self: &Self) -> bool`
- `fn get_attempts(self: Self) -> u32`
- `fn set_attempts(self: & mut Self, value: u32)`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_source<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_source(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_source(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



## bookclerk_plugin_abi::plugin_capnp::queue_message::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** OwnedStruct, Owned, Pipelined, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::queue_message::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::queue_message::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_id(self: &Self) -> bool`
- `fn get_timestamp_unix_ms(self: Self) -> u64`
- `fn get_body(self: Self) -> ::capnp::Result<::capnp::data::Reader<'a>>`
- `fn has_body(self: &Self) -> bool`
- `fn get_content_type(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_content_type(self: &Self) -> bool`
- `fn get_attempts(self: Self) -> u32`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_source(self: &Self) -> bool`

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



