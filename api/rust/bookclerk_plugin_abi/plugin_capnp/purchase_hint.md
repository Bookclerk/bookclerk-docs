**bookclerk_plugin_abi > plugin_capnp > purchase_hint**

# Module: plugin_capnp::purchase_hint

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::purchase_hint::Builder

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
- `fn get_url(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_url<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_url(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_url(self: &Self) -> bool`
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
- `fn get_list_price_cents(self: Self) -> i64`
- `fn set_list_price_cents(self: & mut Self, value: i64)`
- `fn get_list_price_label(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_list_price_label<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_list_price_label(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_list_price_label(self: &Self) -> bool`
- `fn get_member_price_cents(self: Self) -> i64`
- `fn set_member_price_cents(self: & mut Self, value: i64)`
- `fn get_member_price_label(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_member_price_label<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_member_price_label(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_member_price_label(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



## bookclerk_plugin_abi::plugin_capnp::purchase_hint::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::purchase_hint::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::purchase_hint::Reader

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
- `fn get_url(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_url(self: &Self) -> bool`
- `fn get_price_cents(self: Self) -> i64`
- `fn get_currency(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_currency(self: &Self) -> bool`
- `fn get_price_label(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_price_label(self: &Self) -> bool`
- `fn get_list_price_cents(self: Self) -> i64`
- `fn get_list_price_label(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_list_price_label(self: &Self) -> bool`
- `fn get_member_price_cents(self: Self) -> i64`
- `fn get_member_price_label(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_member_price_label(self: &Self) -> bool`

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



