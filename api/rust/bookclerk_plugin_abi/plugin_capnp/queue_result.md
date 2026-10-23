**bookclerk_plugin_abi > plugin_capnp > queue_result**

# Module: plugin_capnp::queue_result

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

**Enums**

- [`Which`](#which)

**Type Aliases**

- [`WhichBuilder`](#whichbuilder)
- [`WhichReader`](#whichreader)

---

## bookclerk_plugin_abi::plugin_capnp::queue_result::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn set_ack(self: & mut Self, _value: ())`
- `fn set_retry(self: & mut Self, value: crate::plugin_capnp::queue_retry::Reader) -> ::capnp::Result<()>`
- `fn init_retry(self: Self) -> crate::plugin_capnp::queue_retry::Builder<'a>`
- `fn has_retry(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichBuilder<'a>, ::capnp::NotInSchema>`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::queue_result::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::queue_result::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::queue_result::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn has_retry(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichReader<'a>, ::capnp::NotInSchema>`

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



## bookclerk_plugin_abi::plugin_capnp::queue_result::Which

*Enum*

**Generic Parameters:**
- A0

**Variants:**
- `Ack(())`
- `Retry(A0)`



## bookclerk_plugin_abi::plugin_capnp::queue_result::WhichBuilder

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::queue_retry::Builder<'a>>>`



## bookclerk_plugin_abi::plugin_capnp::queue_result::WhichReader

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::queue_retry::Reader<'a>>>`



