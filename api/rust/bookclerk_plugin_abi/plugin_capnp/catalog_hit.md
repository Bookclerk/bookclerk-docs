**bookclerk_plugin_abi > plugin_capnp > catalog_hit**

# Module: plugin_capnp::catalog_hit

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::catalog_hit::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
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
- `fn get_series_index(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_series_index<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_series_index(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_series_index(self: &Self) -> bool`
- `fn get_asin(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_asin<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_asin(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_asin(self: &Self) -> bool`
- `fn get_isbn(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_isbn<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_isbn(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_isbn(self: &Self) -> bool`
- `fn get_url(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_url<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_url(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_url(self: &Self) -> bool`
- `fn get_cover_url(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_cover_url<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_cover_url(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_cover_url(self: &Self) -> bool`
- `fn get_origin(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_origin<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_origin(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_origin(self: &Self) -> bool`
- `fn get_subtitle(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_subtitle<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_subtitle(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_subtitle(self: &Self) -> bool`
- `fn get_description(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_description<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_description(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_description(self: &Self) -> bool`
- `fn get_publisher(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_publisher<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_publisher(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_publisher(self: &Self) -> bool`
- `fn get_length_minutes(self: Self) -> i64`
- `fn set_length_minutes(self: & mut Self, value: i64)`
- `fn get_published_at(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_published_at<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_published_at(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_published_at(self: &Self) -> bool`
- `fn get_categories(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_categories<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_categories(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_categories(self: &Self) -> bool`
- `fn get_language(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_language<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_language(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_language(self: &Self) -> bool`
- `fn get_price_cents(self: Self) -> i64`
- `fn set_price_cents(self: & mut Self, value: i64)`
- `fn get_currency(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_currency<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_currency(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_currency(self: &Self) -> bool`
- `fn get_price_label(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_price_label<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_price_label(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_price_label(self: &Self) -> bool`
- `fn get_rating_overall(self: Self) -> f64`
- `fn set_rating_overall(self: & mut Self, value: f64)`
- `fn get_rating_count(self: Self) -> i64`
- `fn set_rating_count(self: & mut Self, value: i64)`
- `fn get_abridgement(self: Self) -> ::core::result::Result<crate::plugin_capnp::Abridgement, ::capnp::NotInSchema>`
- `fn set_abridgement(self: & mut Self, value: crate::plugin_capnp::Abridgement)`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::catalog_hit::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** OwnedStruct, Owned, Pipelined, Copy

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::catalog_hit::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::catalog_hit::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
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
- `fn get_series_index(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_series_index(self: &Self) -> bool`
- `fn get_asin(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_asin(self: &Self) -> bool`
- `fn get_isbn(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_isbn(self: &Self) -> bool`
- `fn get_url(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_url(self: &Self) -> bool`
- `fn get_cover_url(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_cover_url(self: &Self) -> bool`
- `fn get_origin(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_origin(self: &Self) -> bool`
- `fn get_subtitle(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_subtitle(self: &Self) -> bool`
- `fn get_description(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_description(self: &Self) -> bool`
- `fn get_publisher(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_publisher(self: &Self) -> bool`
- `fn get_length_minutes(self: Self) -> i64`
- `fn get_published_at(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_published_at(self: &Self) -> bool`
- `fn get_categories(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_categories(self: &Self) -> bool`
- `fn get_language(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_language(self: &Self) -> bool`
- `fn get_price_cents(self: Self) -> i64`
- `fn get_currency(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_currency(self: &Self) -> bool`
- `fn get_price_label(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_price_label(self: &Self) -> bool`
- `fn get_rating_overall(self: Self) -> f64`
- `fn get_rating_count(self: Self) -> i64`
- `fn get_abridgement(self: Self) -> ::core::result::Result<crate::plugin_capnp::Abridgement, ::capnp::NotInSchema>`

**Traits:** HasTypeId, Copy

**Trait Implementations:**

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
- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



