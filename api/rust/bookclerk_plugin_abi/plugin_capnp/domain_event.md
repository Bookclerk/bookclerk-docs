**bookclerk_plugin_abi > plugin_capnp > domain_event**

# Module: plugin_capnp::domain_event

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::domain_event::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_event_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_event_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_event_id(self: &Self) -> bool`
- `fn get_event_type(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_event_type<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_event_type(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_event_type(self: &Self) -> bool`
- `fn get_schema_version(self: Self) -> u32`
- `fn set_schema_version(self: & mut Self, value: u32)`
- `fn get_occurred_at_unix_ms(self: Self) -> u64`
- `fn set_occurred_at_unix_ms(self: & mut Self, value: u64)`
- `fn get_account_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_account_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_account_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_account_id(self: &Self) -> bool`
- `fn get_correlation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_correlation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_correlation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_correlation_id(self: &Self) -> bool`
- `fn get_causation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_causation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_causation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_causation_id(self: &Self) -> bool`
- `fn get_deduplication_key(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_deduplication_key<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_deduplication_key(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_deduplication_key(self: &Self) -> bool`
- `fn get_delivery_attempt(self: Self) -> u32`
- `fn set_delivery_attempt(self: & mut Self, value: u32)`
- `fn get_payload(self: Self) -> ::capnp::Result<::capnp::data::Builder<'a>>`
- `fn set_payload(self: & mut Self, value: ::capnp::data::Reader)`
- `fn init_payload(self: Self, size: u32) -> ::capnp::data::Builder<'a>`
- `fn has_payload(self: &Self) -> bool`
- `fn get_checkpoint_json(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_checkpoint_json<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_checkpoint_json(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_checkpoint_json(self: &Self) -> bool`
- `fn get_checkpoint_schema_version(self: Self) -> u32`
- `fn set_checkpoint_schema_version(self: & mut Self, value: u32)`
- `fn get_invocation_sequence(self: Self) -> u32`
- `fn set_invocation_sequence(self: & mut Self, value: u32)`
- `fn get_resume_pending(self: Self) -> bool`
- `fn set_resume_pending(self: & mut Self, value: bool)`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_source<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_source(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_source(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::domain_event::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::domain_event::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::domain_event::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_event_id(self: &Self) -> bool`
- `fn get_event_type(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_event_type(self: &Self) -> bool`
- `fn get_schema_version(self: Self) -> u32`
- `fn get_occurred_at_unix_ms(self: Self) -> u64`
- `fn get_account_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_account_id(self: &Self) -> bool`
- `fn get_correlation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_correlation_id(self: &Self) -> bool`
- `fn get_causation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_causation_id(self: &Self) -> bool`
- `fn get_deduplication_key(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_deduplication_key(self: &Self) -> bool`
- `fn get_delivery_attempt(self: Self) -> u32`
- `fn get_payload(self: Self) -> ::capnp::Result<::capnp::data::Reader<'a>>`
- `fn has_payload(self: &Self) -> bool`
- `fn get_checkpoint_json(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_checkpoint_json(self: &Self) -> bool`
- `fn get_checkpoint_schema_version(self: Self) -> u32`
- `fn get_invocation_sequence(self: Self) -> u32`
- `fn get_resume_pending(self: Self) -> bool`
- `fn get_source(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_source(self: &Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

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
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`



