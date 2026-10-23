**bookclerk_plugin_abi > plugin_capnp > plugin_capabilities**

# Module: plugin_capnp::plugin_capabilities

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::plugin_capabilities::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_entrypoints(self: Self) -> ::capnp::Result<::capnp::enum_list::Builder<'a, crate::plugin_capnp::Entrypoint>>`
- `fn set_entrypoints<impl ::capnp::traits::SetterInput<::capnp::enum_list::Owned<crate::plugin_capnp::Entrypoint>>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_entrypoints(self: Self, size: u32) -> ::capnp::enum_list::Builder<'a, crate::plugin_capnp::Entrypoint>`
- `fn has_entrypoints(self: &Self) -> bool`
- `fn get_consumes(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::event_consumer_spec::Owned>>`
- `fn set_consumes(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::event_consumer_spec::Owned>) -> ::capnp::Result<()>`
- `fn init_consumes(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::event_consumer_spec::Owned>`
- `fn has_consumes(self: &Self) -> bool`
- `fn get_produces(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_produces<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_produces(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_produces(self: &Self) -> bool`
- `fn get_jobs(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_jobs<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_jobs(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_jobs(self: &Self) -> bool`
- `fn get_databases(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_databases<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_databases(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_databases(self: &Self) -> bool`
- `fn get_bindings(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_bindings<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_bindings(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_bindings(self: &Self) -> bool`
- `fn get_queue_consumers(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_queue_consumers<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_queue_consumers(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_queue_consumers(self: &Self) -> bool`
- `fn get_queue_producers(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_queue_producers<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_queue_producers(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_queue_producers(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::plugin_capabilities::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Copy, Pipelined, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::plugin_capabilities::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::plugin_capabilities::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_entrypoints(self: Self) -> ::capnp::Result<::capnp::enum_list::Reader<'a, crate::plugin_capnp::Entrypoint>>`
- `fn has_entrypoints(self: &Self) -> bool`
- `fn get_consumes(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::event_consumer_spec::Owned>>`
- `fn has_consumes(self: &Self) -> bool`
- `fn get_produces(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_produces(self: &Self) -> bool`
- `fn get_jobs(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_jobs(self: &Self) -> bool`
- `fn get_databases(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_databases(self: &Self) -> bool`
- `fn get_bindings(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_bindings(self: &Self) -> bool`
- `fn get_queue_consumers(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_queue_consumers(self: &Self) -> bool`
- `fn get_queue_producers(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_queue_producers(self: &Self) -> bool`

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



