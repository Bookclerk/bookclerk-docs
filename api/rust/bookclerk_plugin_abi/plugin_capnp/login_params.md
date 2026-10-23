**bookclerk_plugin_abi > plugin_capnp > login_params**

# Module: plugin_capnp::login_params

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::login_params::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_plugin_data_dir(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_plugin_data_dir<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_plugin_data_dir(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_plugin_data_dir(self: &Self) -> bool`
- `fn get_marketplace(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_marketplace<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_marketplace(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_marketplace(self: &Self) -> bool`
- `fn get_label(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_label<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_label(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_label(self: &Self) -> bool`
- `fn get_email(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_email<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_email(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_email(self: &Self) -> bool`
- `fn get_password(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_password<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_password(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_password(self: &Self) -> bool`
- `fn get_force(self: Self) -> bool`
- `fn set_force(self: & mut Self, value: bool)`
- `fn get_callback_bind(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_callback_bind<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_callback_bind(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_callback_bind(self: &Self) -> bool`
- `fn get_callback_ipc(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_callback_ipc<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_callback_ipc(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_callback_ipc(self: &Self) -> bool`
- `fn get_callback_public_base(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_callback_public_base<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_callback_public_base(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_callback_public_base(self: &Self) -> bool`
- `fn get_external(self: Self) -> bool`
- `fn set_external(self: & mut Self, value: bool)`
- `fn get_response_url(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_response_url<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_response_url(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_response_url(self: &Self) -> bool`
- `fn get_show_qr(self: Self) -> bool`
- `fn set_show_qr(self: & mut Self, value: bool)`
- `fn get_timeout_secs(self: Self) -> u64`
- `fn set_timeout_secs(self: & mut Self, value: u64)`
- `fn get_extra(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Builder<'a>>`
- `fn set_extra(self: & mut Self, value: crate::plugin_capnp::extensible_config::Reader) -> ::capnp::Result<()>`
- `fn init_extra(self: Self) -> crate::plugin_capnp::extensible_config::Builder<'a>`
- `fn has_extra(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::login_params::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::login_params::Pipeline

*Struct*

**Methods:**

- `fn get_extra(self: &Self) -> crate::plugin_capnp::extensible_config::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::login_params::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_plugin_data_dir(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_plugin_data_dir(self: &Self) -> bool`
- `fn get_marketplace(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_marketplace(self: &Self) -> bool`
- `fn get_label(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_label(self: &Self) -> bool`
- `fn get_email(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_email(self: &Self) -> bool`
- `fn get_password(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_password(self: &Self) -> bool`
- `fn get_force(self: Self) -> bool`
- `fn get_callback_bind(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_callback_bind(self: &Self) -> bool`
- `fn get_callback_ipc(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_callback_ipc(self: &Self) -> bool`
- `fn get_callback_public_base(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_callback_public_base(self: &Self) -> bool`
- `fn get_external(self: Self) -> bool`
- `fn get_response_url(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_response_url(self: &Self) -> bool`
- `fn get_show_qr(self: Self) -> bool`
- `fn get_timeout_secs(self: Self) -> u64`
- `fn get_extra(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Reader<'a>>`
- `fn has_extra(self: &Self) -> bool`

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



