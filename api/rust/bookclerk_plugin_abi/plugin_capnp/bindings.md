**bookclerk_plugin_abi > plugin_capnp > bindings**

# Module: plugin_capnp::bindings

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::bindings::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_config(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Builder<'a>>`
- `fn set_config(self: & mut Self, value: crate::plugin_capnp::extensible_config::Reader) -> ::capnp::Result<()>`
- `fn init_config(self: Self) -> crate::plugin_capnp::extensible_config::Builder<'a>`
- `fn has_config(self: &Self) -> bool`
- `fn get_secrets(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Builder<'a>>`
- `fn set_secrets(self: & mut Self, value: crate::plugin_capnp::extensible_config::Reader) -> ::capnp::Result<()>`
- `fn init_secrets(self: Self) -> crate::plugin_capnp::extensible_config::Builder<'a>`
- `fn has_secrets(self: &Self) -> bool`
- `fn get_adapter(self: Self) -> ::capnp::Result<crate::plugin_capnp::database_adapter_config::Builder<'a>>`
- `fn set_adapter(self: & mut Self, value: crate::plugin_capnp::database_adapter_config::Reader) -> ::capnp::Result<()>`
- `fn init_adapter(self: Self) -> crate::plugin_capnp::database_adapter_config::Builder<'a>`
- `fn has_adapter(self: &Self) -> bool`
- `fn get_events(self: Self) -> ::capnp::Result<crate::plugin_capnp::event_publisher::Client>`
- `fn set_events(self: & mut Self, value: crate::plugin_capnp::event_publisher::Client)`
- `fn has_events(self: &Self) -> bool`
- `fn get_databases(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_database::Owned>>`
- `fn set_databases(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::named_database::Owned>) -> ::capnp::Result<()>`
- `fn init_databases(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_database::Owned>`
- `fn has_databases(self: &Self) -> bool`
- `fn get_cancel(self: Self) -> ::capnp::Result<crate::plugin_capnp::cancellation::Client>`
- `fn set_cancel(self: & mut Self, value: crate::plugin_capnp::cancellation::Client)`
- `fn has_cancel(self: &Self) -> bool`
- `fn get_storage(self: Self) -> ::capnp::Result<crate::plugin_capnp::destination::Client>`
- `fn set_storage(self: & mut Self, value: crate::plugin_capnp::destination::Client)`
- `fn has_storage(self: &Self) -> bool`
- `fn get_queues(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_queue::Owned>>`
- `fn set_queues(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::named_queue::Owned>) -> ::capnp::Result<()>`
- `fn init_queues(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_queue::Owned>`
- `fn has_queues(self: &Self) -> bool`
- `fn get_kv(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_kv::Owned>>`
- `fn set_kv(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::named_kv::Owned>) -> ::capnp::Result<()>`
- `fn init_kv(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_kv::Owned>`
- `fn has_kv(self: &Self) -> bool`

**Traits:** HasStructSize, HasTypeId

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::bindings::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::bindings::Pipeline

*Struct*

**Methods:**

- `fn get_config(self: &Self) -> crate::plugin_capnp::extensible_config::Pipeline`
- `fn get_secrets(self: &Self) -> crate::plugin_capnp::extensible_config::Pipeline`
- `fn get_adapter(self: &Self) -> crate::plugin_capnp::database_adapter_config::Pipeline`
- `fn get_events(self: &Self) -> crate::plugin_capnp::event_publisher::Client`
- `fn get_cancel(self: &Self) -> crate::plugin_capnp::cancellation::Client`
- `fn get_storage(self: &Self) -> crate::plugin_capnp::destination::Client`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::bindings::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_config(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Reader<'a>>`
- `fn has_config(self: &Self) -> bool`
- `fn get_secrets(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Reader<'a>>`
- `fn has_secrets(self: &Self) -> bool`
- `fn get_adapter(self: Self) -> ::capnp::Result<crate::plugin_capnp::database_adapter_config::Reader<'a>>`
- `fn has_adapter(self: &Self) -> bool`
- `fn get_events(self: Self) -> ::capnp::Result<crate::plugin_capnp::event_publisher::Client>`
- `fn has_events(self: &Self) -> bool`
- `fn get_databases(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::named_database::Owned>>`
- `fn has_databases(self: &Self) -> bool`
- `fn get_cancel(self: Self) -> ::capnp::Result<crate::plugin_capnp::cancellation::Client>`
- `fn has_cancel(self: &Self) -> bool`
- `fn get_storage(self: Self) -> ::capnp::Result<crate::plugin_capnp::destination::Client>`
- `fn has_storage(self: &Self) -> bool`
- `fn get_queues(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::named_queue::Owned>>`
- `fn has_queues(self: &Self) -> bool`
- `fn get_kv(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::named_kv::Owned>>`
- `fn has_kv(self: &Self) -> bool`

**Traits:** HasTypeId, Copy

**Trait Implementations:**

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
- **SetterInput**
  - `fn set_pointer_builder(pointer: ::capnp::private::layout::PointerBuilder, value: Self, canonicalize: bool) -> ::capnp::Result<()>`



