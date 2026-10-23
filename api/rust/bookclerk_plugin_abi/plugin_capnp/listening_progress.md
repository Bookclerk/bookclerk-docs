**bookclerk_plugin_abi > plugin_capnp > listening_progress**

# Module: plugin_capnp::listening_progress

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::listening_progress::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_external_user_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_external_user_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_external_user_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_external_user_id(self: &Self) -> bool`
- `fn get_external_item_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_external_item_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_external_item_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_external_item_id(self: &Self) -> bool`
- `fn get_identity_id(self: Self) -> i64`
- `fn set_identity_id(self: & mut Self, value: i64)`
- `fn get_title(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_title<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_title(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_title(self: &Self) -> bool`
- `fn get_authors(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_authors<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_authors(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_authors(self: &Self) -> bool`
- `fn get_asin(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_asin<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_asin(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_asin(self: &Self) -> bool`
- `fn get_isbn(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_isbn<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_isbn(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_isbn(self: &Self) -> bool`
- `fn get_progress(self: Self) -> f64`
- `fn set_progress(self: & mut Self, value: f64)`
- `fn get_current_time_seconds(self: Self) -> f64`
- `fn set_current_time_seconds(self: & mut Self, value: f64)`
- `fn get_duration_seconds(self: Self) -> f64`
- `fn set_duration_seconds(self: & mut Self, value: f64)`
- `fn get_is_finished(self: Self) -> bool`
- `fn set_is_finished(self: & mut Self, value: bool)`
- `fn get_last_listened_at_unix_ms(self: Self) -> u64`
- `fn set_last_listened_at_unix_ms(self: & mut Self, value: u64)`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::listening_progress::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::listening_progress::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::listening_progress::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_external_user_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_external_user_id(self: &Self) -> bool`
- `fn get_external_item_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_external_item_id(self: &Self) -> bool`
- `fn get_identity_id(self: Self) -> i64`
- `fn get_title(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_title(self: &Self) -> bool`
- `fn get_authors(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_authors(self: &Self) -> bool`
- `fn get_asin(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_asin(self: &Self) -> bool`
- `fn get_isbn(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_isbn(self: &Self) -> bool`
- `fn get_progress(self: Self) -> f64`
- `fn get_current_time_seconds(self: Self) -> f64`
- `fn get_duration_seconds(self: Self) -> f64`
- `fn get_is_finished(self: Self) -> bool`
- `fn get_last_listened_at_unix_ms(self: Self) -> u64`

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



