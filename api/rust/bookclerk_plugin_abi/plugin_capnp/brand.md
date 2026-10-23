**bookclerk_plugin_abi > plugin_capnp > brand**

# Module: plugin_capnp::brand

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::brand::Builder

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
- `fn get_name(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_name<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_name(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_name(self: &Self) -> bool`
- `fn get_bg(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_bg<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_bg(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_bg(self: &Self) -> bool`
- `fn get_fg(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_fg<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_fg(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_fg(self: &Self) -> bool`
- `fn get_accent(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_accent<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_accent(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_accent(self: &Self) -> bool`
- `fn get_icon_url(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_icon_url<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_icon_url(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_icon_url(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::brand::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** OwnedStruct, Owned, Copy, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::brand::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::brand::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_id(self: &Self) -> bool`
- `fn get_name(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_name(self: &Self) -> bool`
- `fn get_bg(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_bg(self: &Self) -> bool`
- `fn get_fg(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_fg(self: &Self) -> bool`
- `fn get_accent(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_accent(self: &Self) -> bool`
- `fn get_icon_url(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_icon_url(self: &Self) -> bool`

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



