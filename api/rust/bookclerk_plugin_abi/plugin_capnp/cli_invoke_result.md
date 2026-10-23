**bookclerk_plugin_abi > plugin_capnp > cli_invoke_result**

# Module: plugin_capnp::cli_invoke_result

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::cli_invoke_result::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_exit_code(self: Self) -> i32`
- `fn set_exit_code(self: & mut Self, value: i32)`
- `fn get_stdout(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_stdout<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_stdout(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_stdout(self: &Self) -> bool`
- `fn get_stderr(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_stderr<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_stderr(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_stderr(self: &Self) -> bool`
- `fn get_payload(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Builder<'a>>`
- `fn set_payload(self: & mut Self, value: crate::plugin_capnp::extensible_config::Reader) -> ::capnp::Result<()>`
- `fn init_payload(self: Self) -> crate::plugin_capnp::extensible_config::Builder<'a>`
- `fn has_payload(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::cli_invoke_result::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::cli_invoke_result::Pipeline

*Struct*

**Methods:**

- `fn get_payload(self: &Self) -> crate::plugin_capnp::extensible_config::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::cli_invoke_result::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_exit_code(self: Self) -> i32`
- `fn get_stdout(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_stdout(self: &Self) -> bool`
- `fn get_stderr(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_stderr(self: &Self) -> bool`
- `fn get_payload(self: Self) -> ::capnp::Result<crate::plugin_capnp::extensible_config::Reader<'a>>`
- `fn has_payload(self: &Self) -> bool`

**Traits:** Copy, HasTypeId

**Trait Implementations:**

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
- **Debug**
  - `fn fmt(self: &Self, f: & mut ::core::fmt::Formatter) -> ::core::result::Result<(), ::core::fmt::Error>`
- **Imbue**
  - `fn imbue(self: & mut Self, cap_table: &'a ::capnp::private::layout::CapTable)`



