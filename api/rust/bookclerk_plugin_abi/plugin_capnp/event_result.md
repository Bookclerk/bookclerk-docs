**bookclerk_plugin_abi > plugin_capnp > event_result**

# Module: plugin_capnp::event_result

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

## bookclerk_plugin_abi::plugin_capnp::event_result::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn set_ack(self: & mut Self, value: crate::plugin_capnp::event_ack::Reader) -> ::capnp::Result<()>`
- `fn init_ack(self: Self) -> crate::plugin_capnp::event_ack::Builder<'a>`
- `fn has_ack(self: &Self) -> bool`
- `fn set_retry(self: & mut Self, value: crate::plugin_capnp::event_retry::Reader) -> ::capnp::Result<()>`
- `fn init_retry(self: Self) -> crate::plugin_capnp::event_retry::Builder<'a>`
- `fn has_retry(self: &Self) -> bool`
- `fn set_reject(self: & mut Self, value: crate::plugin_capnp::event_reject::Reader) -> ::capnp::Result<()>`
- `fn init_reject(self: Self) -> crate::plugin_capnp::event_reject::Builder<'a>`
- `fn has_reject(self: &Self) -> bool`
- `fn set_dead_letter(self: & mut Self, value: crate::plugin_capnp::event_dead_letter::Reader) -> ::capnp::Result<()>`
- `fn init_dead_letter(self: Self) -> crate::plugin_capnp::event_dead_letter::Builder<'a>`
- `fn has_dead_letter(self: &Self) -> bool`
- `fn set_suspended(self: & mut Self, value: crate::plugin_capnp::event_suspended::Reader) -> ::capnp::Result<()>`
- `fn init_suspended(self: Self) -> crate::plugin_capnp::event_suspended::Builder<'a>`
- `fn has_suspended(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichBuilder<'a>, ::capnp::NotInSchema>`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::event_result::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::event_result::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::event_result::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn has_ack(self: &Self) -> bool`
- `fn has_retry(self: &Self) -> bool`
- `fn has_reject(self: &Self) -> bool`
- `fn has_dead_letter(self: &Self) -> bool`
- `fn has_suspended(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichReader<'a>, ::capnp::NotInSchema>`

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
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`



## bookclerk_plugin_abi::plugin_capnp::event_result::Which

*Enum*

**Generic Parameters:**
- A0
- A1
- A2
- A3
- A4

**Variants:**
- `Ack(A0)`
- `Retry(A1)`
- `Reject(A2)`
- `DeadLetter(A3)`
- `Suspended(A4)`



## bookclerk_plugin_abi::plugin_capnp::event_result::WhichBuilder

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::event_ack::Builder<'a>>, ::capnp::Result<crate::plugin_capnp::event_retry::Builder<'a>>, ::capnp::Result<crate::plugin_capnp::event_reject::Builder<'a>>, ::capnp::Result<crate::plugin_capnp::event_dead_letter::Builder<'a>>, ::capnp::Result<crate::plugin_capnp::event_suspended::Builder<'a>>>`



## bookclerk_plugin_abi::plugin_capnp::event_result::WhichReader

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::event_ack::Reader<'a>>, ::capnp::Result<crate::plugin_capnp::event_retry::Reader<'a>>, ::capnp::Result<crate::plugin_capnp::event_reject::Reader<'a>>, ::capnp::Result<crate::plugin_capnp::event_dead_letter::Reader<'a>>, ::capnp::Result<crate::plugin_capnp::event_suspended::Reader<'a>>>`



