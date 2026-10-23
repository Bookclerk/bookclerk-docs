**bookclerk_plugin_abi > plugin_capnp > plain_fetch**

# Module: plugin_capnp::plain_fetch

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::plain_fetch::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_parts(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::plain_part::Owned>>`
- `fn set_parts(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::plain_part::Owned>) -> ::capnp::Result<()>`
- `fn init_parts(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::plain_part::Owned>`
- `fn has_parts(self: &Self) -> bool`
- `fn get_m4b_path(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_m4b_path<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_m4b_path(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_m4b_path(self: &Self) -> bool`
- `fn get_cover_path(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_cover_path<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_cover_path(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_cover_path(self: &Self) -> bool`
- `fn get_chapters(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::chapter_marker::Owned>>`
- `fn set_chapters(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::chapter_marker::Owned>) -> ::capnp::Result<()>`
- `fn init_chapters(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::chapter_marker::Owned>`
- `fn has_chapters(self: &Self) -> bool`
- `fn get_pdf_url(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_pdf_url<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_pdf_url(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_pdf_url(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::plain_fetch::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::plain_fetch::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::plain_fetch::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_parts(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::plain_part::Owned>>`
- `fn has_parts(self: &Self) -> bool`
- `fn get_m4b_path(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_m4b_path(self: &Self) -> bool`
- `fn get_cover_path(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_cover_path(self: &Self) -> bool`
- `fn get_chapters(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::chapter_marker::Owned>>`
- `fn has_chapters(self: &Self) -> bool`
- `fn get_pdf_url(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_pdf_url(self: &Self) -> bool`

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



