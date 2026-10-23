**bookclerk_plugin_abi > plugin_capnp > db_value**

# Module: plugin_capnp::db_value

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

## bookclerk_plugin_abi::plugin_capnp::db_value::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn set_null(self: & mut Self, value: crate::plugin_capnp::DbType)`
- `fn set_boolean(self: & mut Self, value: bool)`
- `fn set_int64(self: & mut Self, value: i64)`
- `fn set_float64(self: & mut Self, value: f64)`
- `fn set_text<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_text(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_text(self: &Self) -> bool`
- `fn set_bytes(self: & mut Self, value: ::capnp::data::Reader)`
- `fn init_bytes(self: Self, size: u32) -> ::capnp::data::Builder<'a>`
- `fn has_bytes(self: &Self) -> bool`
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



## bookclerk_plugin_abi::plugin_capnp::db_value::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::db_value::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::db_value::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn has_text(self: &Self) -> bool`
- `fn has_bytes(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichReader<'a>, ::capnp::NotInSchema>`

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
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::db_value::Which

*Enum*

**Generic Parameters:**
- A0
- A1

**Variants:**
- `Null(::core::result::Result<crate::plugin_capnp::DbType, ::capnp::NotInSchema>)`
- `Boolean(bool)`
- `Int64(i64)`
- `Float64(f64)`
- `Text(A0)`
- `Bytes(A1)`



## bookclerk_plugin_abi::plugin_capnp::db_value::WhichBuilder

*Type Alias*: `Which<::capnp::Result<::capnp::text::Builder<'a>>, ::capnp::Result<::capnp::data::Builder<'a>>>`



## bookclerk_plugin_abi::plugin_capnp::db_value::WhichReader

*Type Alias*: `Which<::capnp::Result<::capnp::text::Reader<'a>>, ::capnp::Result<::capnp::data::Reader<'a>>>`



