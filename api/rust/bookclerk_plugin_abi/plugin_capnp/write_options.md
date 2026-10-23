**bookclerk_plugin_abi > plugin_capnp > write_options**

# Module: plugin_capnp::write_options

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::write_options::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_content_type(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_content_type<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_content_type(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_content_type(self: &Self) -> bool`
- `fn get_content_length(self: Self) -> u64`
- `fn set_content_length(self: & mut Self, value: u64)`
- `fn get_sha256(self: Self) -> ::capnp::Result<::capnp::data::Builder<'a>>`
- `fn set_sha256(self: & mut Self, value: ::capnp::data::Reader)`
- `fn init_sha256(self: Self, size: u32) -> ::capnp::data::Builder<'a>`
- `fn has_sha256(self: &Self) -> bool`
- `fn get_commit_token(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_commit_token<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_commit_token(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_commit_token(self: &Self) -> bool`
- `fn get_stage_only(self: Self) -> bool`
- `fn set_stage_only(self: & mut Self, value: bool)`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::write_options::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::write_options::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::write_options::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_content_type(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_content_type(self: &Self) -> bool`
- `fn get_content_length(self: Self) -> u64`
- `fn get_sha256(self: Self) -> ::capnp::Result<::capnp::data::Reader<'a>>`
- `fn has_sha256(self: &Self) -> bool`
- `fn get_commit_token(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_commit_token(self: &Self) -> bool`
- `fn get_stage_only(self: Self) -> bool`

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



