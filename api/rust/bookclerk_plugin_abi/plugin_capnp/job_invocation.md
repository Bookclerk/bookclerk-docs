**bookclerk_plugin_abi > plugin_capnp > job_invocation**

# Module: plugin_capnp::job_invocation

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::job_invocation::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_payload_schema_version(self: Self) -> u32`
- `fn set_payload_schema_version(self: & mut Self, value: u32)`
- `fn get_invocation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_invocation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_invocation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_invocation_id(self: &Self) -> bool`
- `fn get_command_type(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_command_type<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_command_type(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_command_type(self: &Self) -> bool`
- `fn get_payload_json(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_payload_json<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_payload_json(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_payload_json(self: &Self) -> bool`
- `fn get_idempotency_key(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_idempotency_key<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_idempotency_key(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_idempotency_key(self: &Self) -> bool`
- `fn get_attempt(self: Self) -> u32`
- `fn set_attempt(self: & mut Self, value: u32)`
- `fn get_correlation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_correlation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_correlation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_correlation_id(self: &Self) -> bool`
- `fn get_causation_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_causation_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_causation_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_causation_id(self: &Self) -> bool`
- `fn get_deadline_unix_ms(self: Self) -> u64`
- `fn set_deadline_unix_ms(self: & mut Self, value: u64)`
- `fn get_checkpoint_json(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_checkpoint_json<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_checkpoint_json(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_checkpoint_json(self: &Self) -> bool`
- `fn get_checkpoint_schema_version(self: Self) -> u32`
- `fn set_checkpoint_schema_version(self: & mut Self, value: u32)`
- `fn get_invocation_sequence(self: Self) -> u32`
- `fn set_invocation_sequence(self: & mut Self, value: u32)`
- `fn get_step_id(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_step_id<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_step_id(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_step_id(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::job_invocation::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::job_invocation::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::job_invocation::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_payload_schema_version(self: Self) -> u32`
- `fn get_invocation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_invocation_id(self: &Self) -> bool`
- `fn get_command_type(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_command_type(self: &Self) -> bool`
- `fn get_payload_json(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_payload_json(self: &Self) -> bool`
- `fn get_idempotency_key(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_idempotency_key(self: &Self) -> bool`
- `fn get_attempt(self: Self) -> u32`
- `fn get_correlation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_correlation_id(self: &Self) -> bool`
- `fn get_causation_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_causation_id(self: &Self) -> bool`
- `fn get_deadline_unix_ms(self: Self) -> u64`
- `fn get_checkpoint_json(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_checkpoint_json(self: &Self) -> bool`
- `fn get_checkpoint_schema_version(self: Self) -> u32`
- `fn get_invocation_sequence(self: Self) -> u32`
- `fn get_step_id(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_step_id(self: &Self) -> bool`

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



