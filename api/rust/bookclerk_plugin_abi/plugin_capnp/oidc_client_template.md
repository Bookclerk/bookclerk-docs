**bookclerk_plugin_abi > plugin_capnp > oidc_client_template**

# Module: plugin_capnp::oidc_client_template

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::oidc_client_template::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_client_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_client_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_client_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_client_id(self: &Self) -> bool`
- `fn get_display_name(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_display_name<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_display_name(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_display_name(self: &Self) -> bool`
- `fn get_callback_path(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_callback_path<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_callback_path(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_callback_path(self: &Self) -> bool`
- `fn get_public_client(self: Self) -> bool`
- `fn set_public_client(self: & mut Self, value: bool)`
- `fn get_default_scopes(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_default_scopes<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_default_scopes(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_default_scopes(self: &Self) -> bool`
- `fn get_issue_refresh_token(self: Self) -> bool`
- `fn set_issue_refresh_token(self: & mut Self, value: bool)`
- `fn get_origin_config_key(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_origin_config_key<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_origin_config_key(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_origin_config_key(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::oidc_client_template::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::oidc_client_template::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::oidc_client_template::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_client_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_client_id(self: &Self) -> bool`
- `fn get_display_name(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_display_name(self: &Self) -> bool`
- `fn get_callback_path(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_callback_path(self: &Self) -> bool`
- `fn get_public_client(self: Self) -> bool`
- `fn get_default_scopes(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_default_scopes(self: &Self) -> bool`
- `fn get_issue_refresh_token(self: Self) -> bool`
- `fn get_origin_config_key(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_origin_config_key(self: &Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`
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



