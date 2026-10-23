**bookclerk_plugin_abi > plugin_capnp > fetch_title_params**

# Module: plugin_capnp::fetch_title_params

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::fetch_title_params::Builder

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
- `fn get_account_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_account_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_account_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_account_id(self: &Self) -> bool`
- `fn get_title_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_title_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_title_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_title_id(self: &Self) -> bool`
- `fn get_cache_dir(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_cache_dir<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_cache_dir(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_cache_dir(self: &Self) -> bool`
- `fn get_credentials(self: Self) -> ::capnp::Result<::capnp::data::Builder<'a>>`
- `fn set_credentials(self: & mut Self, value: ::capnp::data::Reader)`
- `fn init_credentials(self: Self, size: u32) -> ::capnp::data::Builder<'a>`
- `fn has_credentials(self: &Self) -> bool`
- `fn get_source_config(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Builder<'a>>`
- `fn set_source_config(self: & mut Self, value: crate::plugin_capnp::extensible_config::Reader) -> ::capnp::Result<()>`
- `fn init_source_config(self: Self) -> crate::plugin_capnp::extensible_config::Builder<'a>`
- `fn has_source_config(self: &Self) -> bool`
- `fn get_fetch(self: Self) -> ::capnp::Result<crate::plugin_capnp::fetch_options::Builder<'a>>`
- `fn set_fetch(self: & mut Self, value: crate::plugin_capnp::fetch_options::Reader) -> ::capnp::Result<()>`
- `fn init_fetch(self: Self) -> crate::plugin_capnp::fetch_options::Builder<'a>`
- `fn has_fetch(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::fetch_title_params::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, Pipelined, OwnedStruct, Owned

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::fetch_title_params::Pipeline

*Struct*

**Methods:**

- `fn get_source_config(self: &Self) -> crate::plugin_capnp::extensible_config::Pipeline`
- `fn get_fetch(self: &Self) -> crate::plugin_capnp::fetch_options::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::fetch_title_params::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_plugin_data_dir(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_plugin_data_dir(self: &Self) -> bool`
- `fn get_account_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_account_id(self: &Self) -> bool`
- `fn get_title_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_title_id(self: &Self) -> bool`
- `fn get_cache_dir(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_cache_dir(self: &Self) -> bool`
- `fn get_credentials(self: Self) -> ::capnp::Result<::capnp::data::Reader<'a>>`
- `fn has_credentials(self: &Self) -> bool`
- `fn get_source_config(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Reader<'a>>`
- `fn has_source_config(self: &Self) -> bool`
- `fn get_fetch(self: Self) -> ::capnp::Result<crate::plugin_capnp::fetch_options::Reader<'a>>`
- `fn has_fetch(self: &Self) -> bool`

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



