**bookclerk_plugin_abi > plugin_capnp > plugin_event**

# Module: plugin_capnp::plugin_event

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::plugin_event::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_type(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_event_type<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_event_type(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_event_type(self: &Self) -> bool`
- `fn get_schema_version(self: Self) -> u32`
- `fn set_schema_version(self: & mut Self, value: u32)`
- `fn get_deduplication_key(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_deduplication_key<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_deduplication_key(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_deduplication_key(self: &Self) -> bool`
- `fn get_payload(self: Self) -> ::capnp::Result<::capnp::data::Builder<'a>>`
- `fn set_payload(self: & mut Self, value: ::capnp::data::Reader)`
- `fn init_payload(self: Self, size: u32) -> ::capnp::data::Builder<'a>`
- `fn has_payload(self: &Self) -> bool`
- `fn get_occurred_at_unix_ms(self: Self) -> u64`
- `fn set_occurred_at_unix_ms(self: & mut Self, value: u64)`
- `fn get_correlation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_correlation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_correlation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_correlation_id(self: &Self) -> bool`
- `fn get_causation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_causation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_causation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_causation_id(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::plugin_event::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::plugin_event::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::plugin_event::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_type(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_event_type(self: &Self) -> bool`
- `fn get_schema_version(self: Self) -> u32`
- `fn get_deduplication_key(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_deduplication_key(self: &Self) -> bool`
- `fn get_payload(self: Self) -> ::capnp::Result<::capnp::data::Reader<'a>>`
- `fn has_payload(self: &Self) -> bool`
- `fn get_occurred_at_unix_ms(self: Self) -> u64`
- `fn get_correlation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_correlation_id(self: &Self) -> bool`
- `fn get_causation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_causation_id(self: &Self) -> bool`

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



