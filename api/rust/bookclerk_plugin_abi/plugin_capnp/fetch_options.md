**bookclerk_plugin_abi > plugin_capnp > fetch_options**

# Module: plugin_capnp::fetch_options

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::fetch_options::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_widevine(self: Self) -> bool`
- `fn set_widevine(self: & mut Self, value: bool)`
- `fn get_xhe_aac(self: Self) -> bool`
- `fn set_xhe_aac(self: & mut Self, value: bool)`
- `fn get_widevine_cdm_path(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_widevine_cdm_path<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_widevine_cdm_path(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_widevine_cdm_path(self: &Self) -> bool`
- `fn get_widevine_cdm_provider(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_widevine_cdm_provider<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_widevine_cdm_provider(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_widevine_cdm_provider(self: &Self) -> bool`
- `fn get_download_cover(self: Self) -> bool`
- `fn set_download_cover(self: & mut Self, value: bool)`
- `fn get_download_pdf(self: Self) -> bool`
- `fn set_download_pdf(self: & mut Self, value: bool)`
- `fn get_cover_size(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_cover_size<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_cover_size(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_cover_size(self: &Self) -> bool`
- `fn get_chapter_layout(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_chapter_layout<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_chapter_layout(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_chapter_layout(self: &Self) -> bool`
- `fn get_strip_audible_brand_audio(self: Self) -> bool`
- `fn set_strip_audible_brand_audio(self: & mut Self, value: bool)`
- `fn get_download_clips_bookmarks(self: Self) -> bool`
- `fn set_download_clips_bookmarks(self: & mut Self, value: bool)`
- `fn get_retain_aax_file(self: Self) -> bool`
- `fn set_retain_aax_file(self: & mut Self, value: bool)`
- `fn get_download_speed_limit_kbps(self: Self) -> u32`
- `fn set_download_speed_limit_kbps(self: & mut Self, value: u32)`
- `fn get_save_metadata_json(self: Self) -> bool`
- `fn set_save_metadata_json(self: & mut Self, value: bool)`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::fetch_options::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::fetch_options::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::fetch_options::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_widevine(self: Self) -> bool`
- `fn get_xhe_aac(self: Self) -> bool`
- `fn get_widevine_cdm_path(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_widevine_cdm_path(self: &Self) -> bool`
- `fn get_widevine_cdm_provider(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_widevine_cdm_provider(self: &Self) -> bool`
- `fn get_download_cover(self: Self) -> bool`
- `fn get_download_pdf(self: Self) -> bool`
- `fn get_cover_size(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_cover_size(self: &Self) -> bool`
- `fn get_chapter_layout(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_chapter_layout(self: &Self) -> bool`
- `fn get_strip_audible_brand_audio(self: Self) -> bool`
- `fn get_download_clips_bookmarks(self: Self) -> bool`
- `fn get_retain_aax_file(self: Self) -> bool`
- `fn get_download_speed_limit_kbps(self: Self) -> u32`
- `fn get_save_metadata_json(self: Self) -> bool`

**Traits:** HasTypeId, Copy

**Trait Implementations:**

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
- **From**
  - `fn from(reader: ::capnp::private::layout::StructReader<'a>) -> Self`
- **FromPointerReader**
  - `fn get_from_pointer(reader: &::capnp::private::layout::PointerReader<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



