**bookclerk_plugin_abi > plugin_capnp > expand_candidates_params**

# Module: plugin_capnp::expand_candidates_params

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::expand_candidates_params::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_source<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_source(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_source(self: &Self) -> bool`
- `fn get_product_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_product_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_product_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_product_id(self: &Self) -> bool`
- `fn get_title(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_title<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_title(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_title(self: &Self) -> bool`
- `fn get_authors(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_authors<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_authors(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_authors(self: &Self) -> bool`
- `fn get_narrators(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_narrators<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_narrators(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_narrators(self: &Self) -> bool`
- `fn get_series(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_series<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_series(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_series(self: &Self) -> bool`
- `fn get_series_asin(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_series_asin<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_series_asin(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_series_asin(self: &Self) -> bool`
- `fn get_asin(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_asin<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_asin(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_asin(self: &Self) -> bool`
- `fn get_isbn(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_isbn<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_isbn(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_isbn(self: &Self) -> bool`
- `fn get_region(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_region<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_region(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_region(self: &Self) -> bool`
- `fn get_limit(self: Self) -> u32`
- `fn set_limit(self: & mut Self, value: u32)`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::expand_candidates_params::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::expand_candidates_params::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::expand_candidates_params::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_source(self: &Self) -> bool`
- `fn get_product_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_product_id(self: &Self) -> bool`
- `fn get_title(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_title(self: &Self) -> bool`
- `fn get_authors(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_authors(self: &Self) -> bool`
- `fn get_narrators(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_narrators(self: &Self) -> bool`
- `fn get_series(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_series(self: &Self) -> bool`
- `fn get_series_asin(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_series_asin(self: &Self) -> bool`
- `fn get_asin(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_asin(self: &Self) -> bool`
- `fn get_isbn(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_isbn(self: &Self) -> bool`
- `fn get_region(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_region(self: &Self) -> bool`
- `fn get_limit(self: Self) -> u32`

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



