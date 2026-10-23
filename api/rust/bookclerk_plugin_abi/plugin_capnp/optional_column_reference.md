**bookclerk_plugin_abi > plugin_capnp > optional_column_reference**

# Module: plugin_capnp::optional_column_reference

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

## bookclerk_plugin_abi::plugin_capnp::optional_column_reference::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn set_none(self: & mut Self, _value: ())`
- `fn set_some(self: & mut Self, value: crate::plugin_capnp::column_reference::Reader) -> ::capnp::Result<()>`
- `fn init_some(self: Self) -> crate::plugin_capnp::column_reference::Builder<'a>`
- `fn has_some(self: &Self) -> bool`
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



## bookclerk_plugin_abi::plugin_capnp::optional_column_reference::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::optional_column_reference::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::optional_column_reference::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn has_some(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichReader<'a>, ::capnp::NotInSchema>`

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



## bookclerk_plugin_abi::plugin_capnp::optional_column_reference::Which

*Enum*

**Generic Parameters:**
- A0

**Variants:**
- `None(())`
- `Some(A0)`



## bookclerk_plugin_abi::plugin_capnp::optional_column_reference::WhichBuilder

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::column_reference::Builder<'a>>>`



## bookclerk_plugin_abi::plugin_capnp::optional_column_reference::WhichReader

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::column_reference::Reader<'a>>>`



