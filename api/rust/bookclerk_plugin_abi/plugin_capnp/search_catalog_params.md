**bookclerk_plugin_abi > plugin_capnp > search_catalog_params**

# Module: plugin_capnp::search_catalog_params

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::search_catalog_params::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_query(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_query<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_query(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_query(self: &Self) -> bool`
- `fn get_region(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_region<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_region(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_region(self: &Self) -> bool`
- `fn get_limit(self: Self) -> u32`
- `fn set_limit(self: & mut Self, value: u32)`
- `fn get_page(self: Self) -> u32`
- `fn set_page(self: & mut Self, value: u32)`
- `fn get_sort(self: Self) -> ::core::result::Result<crate::plugin_capnp::CatalogSort, ::capnp::NotInSchema>`
- `fn set_sort(self: & mut Self, value: crate::plugin_capnp::CatalogSort)`
- `fn get_field(self: Self) -> ::core::result::Result<crate::plugin_capnp::CatalogField, ::capnp::NotInSchema>`
- `fn set_field(self: & mut Self, value: crate::plugin_capnp::CatalogField)`
- `fn get_language(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_language<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_language(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_language(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::search_catalog_params::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::search_catalog_params::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::search_catalog_params::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_query(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_query(self: &Self) -> bool`
- `fn get_region(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_region(self: &Self) -> bool`
- `fn get_limit(self: Self) -> u32`
- `fn get_page(self: Self) -> u32`
- `fn get_sort(self: Self) -> ::core::result::Result<crate::plugin_capnp::CatalogSort, ::capnp::NotInSchema>`
- `fn get_field(self: Self) -> ::core::result::Result<crate::plugin_capnp::CatalogField, ::capnp::NotInSchema>`
- `fn get_language(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_language(self: &Self) -> bool`

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



