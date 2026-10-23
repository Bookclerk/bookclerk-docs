**bookclerk_plugin_abi > plugin_capnp > job_controller**

# Module: plugin_capnp::job_controller

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::job_controller::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_invocation(self: Self) -> ::capnp::Result<crate::plugin_capnp::job_invocation::Builder<'a>>`
- `fn set_invocation(self: & mut Self, value: crate::plugin_capnp::job_invocation::Reader) -> ::capnp::Result<()>`
- `fn init_invocation(self: Self) -> crate::plugin_capnp::job_invocation::Builder<'a>`
- `fn has_invocation(self: &Self) -> bool`
- `fn get_input(self: Self) -> ::capnp::Result<crate::plugin_capnp::source::Client>`
- `fn set_input(self: & mut Self, value: crate::plugin_capnp::source::Client)`
- `fn has_input(self: &Self) -> bool`
- `fn get_output(self: Self) -> ::capnp::Result<crate::plugin_capnp::destination::Client>`
- `fn set_output(self: & mut Self, value: crate::plugin_capnp::destination::Client)`
- `fn has_output(self: &Self) -> bool`
- `fn get_progress(self: Self) -> ::capnp::Result<crate::plugin_capnp::progress_sink::Client>`
- `fn set_progress(self: & mut Self, value: crate::plugin_capnp::progress_sink::Client)`
- `fn has_progress(self: &Self) -> bool`
- `fn get_cancel(self: Self) -> ::capnp::Result<crate::plugin_capnp::cancellation::Client>`
- `fn set_cancel(self: & mut Self, value: crate::plugin_capnp::cancellation::Client)`
- `fn has_cancel(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::job_controller::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::job_controller::Pipeline

*Struct*

**Methods:**

- `fn get_invocation(self: &Self) -> crate::plugin_capnp::job_invocation::Pipeline`
- `fn get_input(self: &Self) -> crate::plugin_capnp::source::Client`
- `fn get_output(self: &Self) -> crate::plugin_capnp::destination::Client`
- `fn get_progress(self: &Self) -> crate::plugin_capnp::progress_sink::Client`
- `fn get_cancel(self: &Self) -> crate::plugin_capnp::cancellation::Client`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::job_controller::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_invocation(self: Self) -> ::capnp::Result<crate::plugin_capnp::job_invocation::Reader<'a>>`
- `fn has_invocation(self: &Self) -> bool`
- `fn get_input(self: Self) -> ::capnp::Result<crate::plugin_capnp::source::Client>`
- `fn has_input(self: &Self) -> bool`
- `fn get_output(self: Self) -> ::capnp::Result<crate::plugin_capnp::destination::Client>`
- `fn has_output(self: &Self) -> bool`
- `fn get_progress(self: Self) -> ::capnp::Result<crate::plugin_capnp::progress_sink::Client>`
- `fn has_progress(self: &Self) -> bool`
- `fn get_cancel(self: Self) -> ::capnp::Result<crate::plugin_capnp::cancellation::Client>`
- `fn has_cancel(self: &Self) -> bool`

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



