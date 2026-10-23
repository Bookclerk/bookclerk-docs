**bookclerk_plugin_abi > plugin_capnp > plugin_describe**

# Module: plugin_capnp::plugin_describe

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::plugin_describe::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_api_version(self: Self) -> u32`
- `fn set_api_version(self: & mut Self, value: u32)`
- `fn get_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_id(self: &Self) -> bool`
- `fn get_display_name(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_display_name<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_display_name(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_display_name(self: &Self) -> bool`
- `fn get_rpc_features(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_rpc_features<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_rpc_features(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_rpc_features(self: &Self) -> bool`
- `fn get_scalar_limits(self: Self) -> ::capnp::Result<crate::plugin_capnp::scalar_limits::Builder<'a>>`
- `fn set_scalar_limits(self: & mut Self, value: crate::plugin_capnp::scalar_limits::Reader) -> ::capnp::Result<()>`
- `fn init_scalar_limits(self: Self) -> crate::plugin_capnp::scalar_limits::Builder<'a>`
- `fn has_scalar_limits(self: &Self) -> bool`
- `fn get_capabilities(self: Self) -> ::capnp::Result<crate::plugin_capnp::plugin_capabilities::Builder<'a>>`
- `fn set_capabilities(self: & mut Self, value: crate::plugin_capnp::plugin_capabilities::Reader) -> ::capnp::Result<()>`
- `fn init_capabilities(self: Self) -> crate::plugin_capnp::plugin_capabilities::Builder<'a>`
- `fn has_capabilities(self: &Self) -> bool`
- `fn get_portal_auth_mode(self: Self) -> ::core::result::Result<crate::plugin_capnp::PortalAuthMode, ::capnp::NotInSchema>`
- `fn set_portal_auth_mode(self: & mut Self, value: crate::plugin_capnp::PortalAuthMode)`
- `fn get_password_env_var(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_password_env_var<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_password_env_var(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_password_env_var(self: &Self) -> bool`
- `fn get_aliases(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_aliases<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_aliases(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_aliases(self: &Self) -> bool`
- `fn get_sort_key(self: Self) -> u32`
- `fn set_sort_key(self: & mut Self, value: u32)`
- `fn get_brand(self: Self) -> ::capnp::Result<crate::plugin_capnp::brand::Builder<'a>>`
- `fn set_brand(self: & mut Self, value: crate::plugin_capnp::brand::Reader) -> ::capnp::Result<()>`
- `fn init_brand(self: Self) -> crate::plugin_capnp::brand::Builder<'a>`
- `fn has_brand(self: &Self) -> bool`
- `fn get_config_options(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::config_option::Owned>>`
- `fn set_config_options(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::config_option::Owned>) -> ::capnp::Result<()>`
- `fn init_config_options(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::config_option::Owned>`
- `fn has_config_options(self: &Self) -> bool`
- `fn get_cli(self: Self) -> ::capnp::Result<crate::plugin_capnp::cli_schema::Builder<'a>>`
- `fn set_cli(self: & mut Self, value: crate::plugin_capnp::cli_schema::Reader) -> ::capnp::Result<()>`
- `fn init_cli(self: Self) -> crate::plugin_capnp::cli_schema::Builder<'a>`
- `fn has_cli(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::plugin_describe::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::plugin_describe::Pipeline

*Struct*

**Methods:**

- `fn get_scalar_limits(self: &Self) -> crate::plugin_capnp::scalar_limits::Pipeline`
- `fn get_capabilities(self: &Self) -> crate::plugin_capnp::plugin_capabilities::Pipeline`
- `fn get_brand(self: &Self) -> crate::plugin_capnp::brand::Pipeline`
- `fn get_cli(self: &Self) -> crate::plugin_capnp::cli_schema::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::plugin_describe::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_api_version(self: Self) -> u32`
- `fn get_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_id(self: &Self) -> bool`
- `fn get_display_name(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_display_name(self: &Self) -> bool`
- `fn get_rpc_features(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_rpc_features(self: &Self) -> bool`
- `fn get_scalar_limits(self: Self) -> ::capnp::Result<crate::plugin_capnp::scalar_limits::Reader<'a>>`
- `fn has_scalar_limits(self: &Self) -> bool`
- `fn get_capabilities(self: Self) -> ::capnp::Result<crate::plugin_capnp::plugin_capabilities::Reader<'a>>`
- `fn has_capabilities(self: &Self) -> bool`
- `fn get_portal_auth_mode(self: Self) -> ::core::result::Result<crate::plugin_capnp::PortalAuthMode, ::capnp::NotInSchema>`
- `fn get_password_env_var(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_password_env_var(self: &Self) -> bool`
- `fn get_aliases(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_aliases(self: &Self) -> bool`
- `fn get_sort_key(self: Self) -> u32`
- `fn get_brand(self: Self) -> ::capnp::Result<crate::plugin_capnp::brand::Reader<'a>>`
- `fn has_brand(self: &Self) -> bool`
- `fn get_config_options(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::config_option::Owned>>`
- `fn has_config_options(self: &Self) -> bool`
- `fn get_cli(self: Self) -> ::capnp::Result<crate::plugin_capnp::cli_schema::Reader<'a>>`
- `fn has_cli(self: &Self) -> bool`

**Traits:** HasTypeId, Copy

**Trait Implementations:**

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
- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



