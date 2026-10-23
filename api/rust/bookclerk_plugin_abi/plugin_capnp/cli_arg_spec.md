**bookclerk_plugin_abi > plugin_capnp > cli_arg_spec**

# Module: plugin_capnp::cli_arg_spec

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::cli_arg_spec::Builder

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
- `fn get_long(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_long<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_long(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_long(self: &Self) -> bool`
- `fn get_short(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_short<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_short(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_short(self: &Self) -> bool`
- `fn get_kind(self: Self) -> ::core::result::Result<crate::plugin_capnp::CliArgKind, ::capnp::NotInSchema>`
- `fn set_kind(self: & mut Self, value: crate::plugin_capnp::CliArgKind)`
- `fn get_required(self: Self) -> bool`
- `fn set_required(self: & mut Self, value: bool)`
- `fn get_default(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_default<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_default(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_default(self: &Self) -> bool`
- `fn get_about(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_about<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_about(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_about(self: &Self) -> bool`
- `fn get_positional(self: Self) -> bool`
- `fn set_positional(self: & mut Self, value: bool)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



## bookclerk_plugin_abi::plugin_capnp::cli_arg_spec::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::cli_arg_spec::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::cli_arg_spec::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_name(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_name(self: &Self) -> bool`
- `fn get_long(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_long(self: &Self) -> bool`
- `fn get_short(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_short(self: &Self) -> bool`
- `fn get_kind(self: Self) -> ::core::result::Result<crate::plugin_capnp::CliArgKind, ::capnp::NotInSchema>`
- `fn get_required(self: Self) -> bool`
- `fn get_default(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_default(self: &Self) -> bool`
- `fn get_about(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_about(self: &Self) -> bool`
- `fn get_positional(self: Self) -> bool`

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



