**bookclerk_plugin_abi > plugin_capnp > table_constraint**

# Module: plugin_capnp::table_constraint

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

## bookclerk_plugin_abi::plugin_capnp::table_constraint::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn set_primary_key<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_primary_key(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_primary_key(self: &Self) -> bool`
- `fn set_unique<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_unique(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_unique(self: &Self) -> bool`
- `fn set_check<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_check(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_check(self: &Self) -> bool`
- `fn set_foreign_key(self: & mut Self, value: crate::plugin_capnp::foreign_key_constraint::Reader) -> ::capnp::Result<()>`
- `fn init_foreign_key(self: Self) -> crate::plugin_capnp::foreign_key_constraint::Builder<'a>`
- `fn has_foreign_key(self: &Self) -> bool`
- `fn which(self: Self) -> ::core::result::Result<WhichBuilder<'a>, ::capnp::NotInSchema>`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::table_constraint::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::table_constraint::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::table_constraint::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn has_primary_key(self: &Self) -> bool`
- `fn has_unique(self: &Self) -> bool`
- `fn has_check(self: &Self) -> bool`
- `fn has_foreign_key(self: &Self) -> bool`
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
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`



## bookclerk_plugin_abi::plugin_capnp::table_constraint::Which

*Enum*

**Generic Parameters:**
- A0
- A1
- A2
- A3

**Variants:**
- `PrimaryKey(A0)`
- `Unique(A1)`
- `Check(A2)`
- `ForeignKey(A3)`



## bookclerk_plugin_abi::plugin_capnp::table_constraint::WhichBuilder

*Type Alias*: `Which<::capnp::Result<::capnp::text_list::Builder<'a>>, ::capnp::Result<::capnp::text_list::Builder<'a>>, ::capnp::Result<::capnp::text::Builder<'a>>, ::capnp::Result<crate::plugin_capnp::foreign_key_constraint::Builder<'a>>>`



## bookclerk_plugin_abi::plugin_capnp::table_constraint::WhichReader

*Type Alias*: `Which<::capnp::Result<::capnp::text_list::Reader<'a>>, ::capnp::Result<::capnp::text_list::Reader<'a>>, ::capnp::Result<::capnp::text::Reader<'a>>, ::capnp::Result<crate::plugin_capnp::foreign_key_constraint::Reader<'a>>>`



