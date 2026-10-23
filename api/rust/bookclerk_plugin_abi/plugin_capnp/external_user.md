**bookclerk_plugin_abi > plugin_capnp > external_user**

# Module: plugin_capnp::external_user

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::external_user::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_provider(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_provider<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_provider(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_provider(self: &Self) -> bool`
- `fn get_external_user_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_external_user_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_external_user_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_external_user_id(self: &Self) -> bool`
- `fn get_display_name(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_display_name<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_display_name(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_display_name(self: &Self) -> bool`
- `fn get_access_token(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_access_token<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_access_token(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_access_token(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::external_user::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** OwnedStruct, Owned, Pipelined, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::external_user::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::external_user::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_provider(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_provider(self: &Self) -> bool`
- `fn get_external_user_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_external_user_id(self: &Self) -> bool`
- `fn get_display_name(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_display_name(self: &Self) -> bool`
- `fn get_access_token(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_access_token(self: &Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`
- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`



