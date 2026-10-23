**bookclerk_plugin_abi > plugin_capnp > db_bootstrap_reply**

# Module: plugin_capnp::db_bootstrap_reply

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

## bookclerk_plugin_abi::plugin_capnp::db_bootstrap_reply::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn set_ok(self: & mut Self, value: crate::plugin_capnp::db_bootstrap::Reader) -> ::capnp::Result<()>`
- `fn init_ok(self: Self) -> crate::plugin_capnp::db_bootstrap::Builder<'a>`
- `fn has_ok(self: &Self) -> bool`
- `fn set_err(self: & mut Self, value: crate::plugin_capnp::plugin_error::Reader) -> ::capnp::Result<()>`
- `fn init_err(self: Self) -> crate::plugin_capnp::plugin_error::Builder<'a>`
- `fn has_err(self: &Self) -> bool`
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



## bookclerk_plugin_abi::plugin_capnp::db_bootstrap_reply::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::db_bootstrap_reply::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::db_bootstrap_reply::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn has_ok(self: &Self) -> bool`
- `fn has_err(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichReader<'a>, ::capnp::NotInSchema>`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`



## bookclerk_plugin_abi::plugin_capnp::db_bootstrap_reply::Which

*Enum*

**Generic Parameters:**
- A0
- A1

**Variants:**
- `Ok(A0)`
- `Err(A1)`



## bookclerk_plugin_abi::plugin_capnp::db_bootstrap_reply::WhichBuilder

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::db_bootstrap::Builder<'a>>, ::capnp::Result<crate::plugin_capnp::plugin_error::Builder<'a>>>`



## bookclerk_plugin_abi::plugin_capnp::db_bootstrap_reply::WhichReader

*Type Alias*: `Which<::capnp::Result<crate::plugin_capnp::db_bootstrap::Reader<'a>>, ::capnp::Result<crate::plugin_capnp::plugin_error::Reader<'a>>>`



