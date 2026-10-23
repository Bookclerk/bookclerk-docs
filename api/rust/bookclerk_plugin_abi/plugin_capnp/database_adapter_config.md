**bookclerk_plugin_abi > plugin_capnp > database_adapter_config**

# Module: plugin_capnp::database_adapter_config

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::database_adapter_config::Builder

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
- `fn get_settings(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Builder<'a>>`
- `fn set_settings(self: & mut Self, value: crate::plugin_capnp::extensible_config::Reader) -> ::capnp::Result<()>`
- `fn init_settings(self: Self) -> crate::plugin_capnp::extensible_config::Builder<'a>`
- `fn has_settings(self: &Self) -> bool`
- `fn get_binding(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_binding<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_binding(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_binding(self: &Self) -> bool`
- `fn get_instance_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_instance_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_instance_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_instance_id(self: &Self) -> bool`
- `fn get_open_existing(self: Self) -> bool`
- `fn set_open_existing(self: & mut Self, value: bool)`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::database_adapter_config::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::database_adapter_config::Pipeline

*Struct*

**Methods:**

- `fn get_settings(self: &Self) -> crate::plugin_capnp::extensible_config::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::database_adapter_config::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_plugin_data_dir(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_plugin_data_dir(self: &Self) -> bool`
- `fn get_settings(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Reader<'a>>`
- `fn has_settings(self: &Self) -> bool`
- `fn get_binding(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_binding(self: &Self) -> bool`
- `fn get_instance_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_instance_id(self: &Self) -> bool`
- `fn get_open_existing(self: Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

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
- **Clone**
  - `fn clone(self: &Self) -> Self`



