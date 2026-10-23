**bookclerk_plugin_abi > plugin_capnp > scalar_limits**

# Module: plugin_capnp::scalar_limits

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::scalar_limits::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_max_scalar_bytes(self: Self) -> u32`
- `fn set_max_scalar_bytes(self: & mut Self, value: u32)`
- `fn get_max_stream_window_bytes(self: Self) -> u32`
- `fn set_max_stream_window_bytes(self: & mut Self, value: u32)`
- `fn get_max_list_page(self: Self) -> u32`
- `fn set_max_list_page(self: & mut Self, value: u32)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::scalar_limits::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** OwnedStruct, Pipelined, Copy, Owned

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::scalar_limits::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::scalar_limits::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_max_scalar_bytes(self: Self) -> u32`
- `fn get_max_stream_window_bytes(self: Self) -> u32`
- `fn get_max_list_page(self: Self) -> u32`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`



