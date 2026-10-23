**bookclerk_plugin_abi > plugin_capnp > kv_namespace > put_params**

# Module: plugin_capnp::kv_namespace::put_params

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::kv_namespace::put_params::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_key(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_key<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_key(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_key(self: &Self) -> bool`
- `fn get_value(self: Self) -> ::capnp::Result<::capnp::data::Builder<'a>>`
- `fn set_value(self: & mut Self, value: ::capnp::data::Reader)`
- `fn init_value(self: Self, size: u32) -> ::capnp::data::Builder<'a>`
- `fn has_value(self: &Self) -> bool`
- `fn get_options(self: Self) -> ::capnp::Result<crate::plugin_capnp::kv_put_options::Builder<'a>>`
- `fn set_options(self: & mut Self, value: crate::plugin_capnp::kv_put_options::Reader) -> ::capnp::Result<()>`
- `fn init_options(self: Self) -> crate::plugin_capnp::kv_put_options::Builder<'a>`
- `fn has_options(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::put_params::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::put_params::Pipeline

*Struct*

**Methods:**

- `fn get_options(self: &Self) -> crate::plugin_capnp::kv_put_options::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::kv_namespace::put_params::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_key(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_key(self: &Self) -> bool`
- `fn get_value(self: Self) -> ::capnp::Result<::capnp::data::Reader<'a>>`
- `fn has_value(self: &Self) -> bool`
- `fn get_options(self: Self) -> ::capnp::Result<crate::plugin_capnp::kv_put_options::Reader<'a>>`
- `fn has_options(self: &Self) -> bool`

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



