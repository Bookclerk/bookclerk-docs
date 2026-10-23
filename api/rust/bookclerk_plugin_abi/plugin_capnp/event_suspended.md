**bookclerk_plugin_abi > plugin_capnp > event_suspended**

# Module: plugin_capnp::event_suspended

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::event_suspended::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_checkpoint_json(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_checkpoint_json<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_checkpoint_json(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_checkpoint_json(self: &Self) -> bool`
- `fn get_checkpoint_schema_version(self: Self) -> u32`
- `fn set_checkpoint_schema_version(self: & mut Self, value: u32)`
- `fn get_wake_at_unix_ms(self: Self) -> u64`
- `fn set_wake_at_unix_ms(self: & mut Self, value: u64)`
- `fn get_wake_on_event_type(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_wake_on_event_type<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_wake_on_event_type(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_wake_on_event_type(self: &Self) -> bool`
- `fn get_wake_on_filter_json(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_wake_on_filter_json<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_wake_on_filter_json(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_wake_on_filter_json(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::event_suspended::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::event_suspended::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::event_suspended::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_checkpoint_json(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_checkpoint_json(self: &Self) -> bool`
- `fn get_checkpoint_schema_version(self: Self) -> u32`
- `fn get_wake_at_unix_ms(self: Self) -> u64`
- `fn get_wake_on_event_type(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_wake_on_event_type(self: &Self) -> bool`
- `fn get_wake_on_filter_json(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_wake_on_filter_json(self: &Self) -> bool`

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
- **Clone**
  - `fn clone(self: &Self) -> Self`
- **IntoInternalStructReader**
  - `fn into_internal_struct_reader(self: Self) -> ::capnp::private::layout::StructReader<'a>`
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`



