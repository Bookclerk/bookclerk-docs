**bookclerk_plugin_abi > plugin_capnp > source_account**

# Module: plugin_capnp::source_account

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::source_account::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_account_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_account_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_account_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_account_id(self: &Self) -> bool`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_source<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_source(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_source(self: &Self) -> bool`
- `fn get_marketplace(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_marketplace<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_marketplace(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_marketplace(self: &Self) -> bool`
- `fn get_label(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_label<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_label(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_label(self: &Self) -> bool`
- `fn get_scan_enabled(self: Self) -> bool`
- `fn set_scan_enabled(self: & mut Self, value: bool)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::source_account::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::source_account::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::source_account::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_account_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_account_id(self: &Self) -> bool`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_source(self: &Self) -> bool`
- `fn get_marketplace(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_marketplace(self: &Self) -> bool`
- `fn get_label(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_label(self: &Self) -> bool`
- `fn get_scan_enabled(self: Self) -> bool`

**Traits:** Copy, HasTypeId

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



