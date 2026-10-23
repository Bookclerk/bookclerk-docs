**bookclerk_plugin_abi > plugin_capnp > scan_params**

# Module: plugin_capnp::scan_params

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::scan_params::Builder

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
- `fn get_accounts(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_accounts<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_accounts(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_accounts(self: &Self) -> bool`
- `fn get_page_size(self: Self) -> u32`
- `fn set_page_size(self: & mut Self, value: u32)`
- `fn get_import_episodes(self: Self) -> bool`
- `fn set_import_episodes(self: & mut Self, value: bool)`
- `fn get_import_plus_titles(self: Self) -> bool`
- `fn set_import_plus_titles(self: & mut Self, value: bool)`
- `fn get_credentials(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::account_credential::Owned>>`
- `fn set_credentials(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::account_credential::Owned>) -> ::capnp::Result<()>`
- `fn init_credentials(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::account_credential::Owned>`
- `fn has_credentials(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::scan_params::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::scan_params::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::scan_params::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_plugin_data_dir(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_plugin_data_dir(self: &Self) -> bool`
- `fn get_accounts(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_accounts(self: &Self) -> bool`
- `fn get_page_size(self: Self) -> u32`
- `fn get_import_episodes(self: Self) -> bool`
- `fn get_import_plus_titles(self: Self) -> bool`
- `fn get_credentials(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::account_credential::Owned>>`
- `fn has_credentials(self: &Self) -> bool`

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



