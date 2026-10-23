**bookclerk_plugin_abi > plugin_capnp > oidc_clients_ok**

# Module: plugin_capnp::oidc_clients_ok

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::oidc_clients_ok::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_clients(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::oidc_client_template::Owned>>`
- `fn set_clients(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::oidc_client_template::Owned>) -> ::capnp::Result<()>`
- `fn init_clients(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::oidc_client_template::Owned>`
- `fn has_clients(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`



## bookclerk_plugin_abi::plugin_capnp::oidc_clients_ok::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::oidc_clients_ok::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::oidc_clients_ok::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_clients(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::oidc_client_template::Owned>>`
- `fn has_clients(self: &Self) -> bool`

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



