**bookclerk_plugin_abi > plugin_capnp > entrypoints**

# Module: plugin_capnp::entrypoints

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::entrypoints::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_consumer(self: Self) -> ::capnp::Result<crate::plugin_capnp::event_consumer::Client>`
- `fn set_event_consumer(self: & mut Self, value: crate::plugin_capnp::event_consumer::Client)`
- `fn has_event_consumer(self: &Self) -> bool`
- `fn get_job_runner(self: Self) -> ::capnp::Result<crate::plugin_capnp::job_runner::Client>`
- `fn set_job_runner(self: & mut Self, value: crate::plugin_capnp::job_runner::Client)`
- `fn has_job_runner(self: &Self) -> bool`
- `fn get_storefront(self: Self) -> ::capnp::Result<crate::plugin_capnp::content_source::Client>`
- `fn set_storefront(self: & mut Self, value: crate::plugin_capnp::content_source::Client)`
- `fn has_storefront(self: &Self) -> bool`
- `fn get_storage(self: Self) -> ::capnp::Result<crate::plugin_capnp::destination::Client>`
- `fn set_storage(self: & mut Self, value: crate::plugin_capnp::destination::Client)`
- `fn has_storage(self: &Self) -> bool`
- `fn get_database_adapter(self: Self) -> ::capnp::Result<crate::plugin_capnp::database::Client>`
- `fn set_database_adapter(self: & mut Self, value: crate::plugin_capnp::database::Client)`
- `fn has_database_adapter(self: &Self) -> bool`
- `fn get_remote_library(self: Self) -> ::capnp::Result<crate::plugin_capnp::remote_library::Client>`
- `fn set_remote_library(self: & mut Self, value: crate::plugin_capnp::remote_library::Client)`
- `fn has_remote_library(self: &Self) -> bool`
- `fn get_cli(self: Self) -> ::capnp::Result<crate::plugin_capnp::plugin_cli::Client>`
- `fn set_cli(self: & mut Self, value: crate::plugin_capnp::plugin_cli::Client)`
- `fn has_cli(self: &Self) -> bool`
- `fn get_oidc(self: Self) -> ::capnp::Result<crate::plugin_capnp::oidc::Client>`
- `fn set_oidc(self: & mut Self, value: crate::plugin_capnp::oidc::Client)`
- `fn has_oidc(self: &Self) -> bool`
- `fn get_queue_consumer(self: Self) -> ::capnp::Result<crate::plugin_capnp::queue_consumer::Client>`
- `fn set_queue_consumer(self: & mut Self, value: crate::plugin_capnp::queue_consumer::Client)`
- `fn has_queue_consumer(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::entrypoints::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::entrypoints::Pipeline

*Struct*

**Methods:**

- `fn get_event_consumer(self: &Self) -> crate::plugin_capnp::event_consumer::Client`
- `fn get_job_runner(self: &Self) -> crate::plugin_capnp::job_runner::Client`
- `fn get_storefront(self: &Self) -> crate::plugin_capnp::content_source::Client`
- `fn get_storage(self: &Self) -> crate::plugin_capnp::destination::Client`
- `fn get_database_adapter(self: &Self) -> crate::plugin_capnp::database::Client`
- `fn get_remote_library(self: &Self) -> crate::plugin_capnp::remote_library::Client`
- `fn get_cli(self: &Self) -> crate::plugin_capnp::plugin_cli::Client`
- `fn get_oidc(self: &Self) -> crate::plugin_capnp::oidc::Client`
- `fn get_queue_consumer(self: &Self) -> crate::plugin_capnp::queue_consumer::Client`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::entrypoints::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_event_consumer(self: Self) -> ::capnp::Result<crate::plugin_capnp::event_consumer::Client>`
- `fn has_event_consumer(self: &Self) -> bool`
- `fn get_job_runner(self: Self) -> ::capnp::Result<crate::plugin_capnp::job_runner::Client>`
- `fn has_job_runner(self: &Self) -> bool`
- `fn get_storefront(self: Self) -> ::capnp::Result<crate::plugin_capnp::content_source::Client>`
- `fn has_storefront(self: &Self) -> bool`
- `fn get_storage(self: Self) -> ::capnp::Result<crate::plugin_capnp::destination::Client>`
- `fn has_storage(self: &Self) -> bool`
- `fn get_database_adapter(self: Self) -> ::capnp::Result<crate::plugin_capnp::database::Client>`
- `fn has_database_adapter(self: &Self) -> bool`
- `fn get_remote_library(self: Self) -> ::capnp::Result<crate::plugin_capnp::remote_library::Client>`
- `fn has_remote_library(self: &Self) -> bool`
- `fn get_cli(self: Self) -> ::capnp::Result<crate::plugin_capnp::plugin_cli::Client>`
- `fn has_cli(self: &Self) -> bool`
- `fn get_oidc(self: Self) -> ::capnp::Result<crate::plugin_capnp::oidc::Client>`
- `fn has_oidc(self: &Self) -> bool`
- `fn get_queue_consumer(self: Self) -> ::capnp::Result<crate::plugin_capnp::queue_consumer::Client>`
- `fn has_queue_consumer(self: &Self) -> bool`

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



