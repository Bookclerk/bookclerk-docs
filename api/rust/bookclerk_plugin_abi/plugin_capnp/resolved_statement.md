**bookclerk_plugin_abi > plugin_capnp > resolved_statement**

# Module: plugin_capnp::resolved_statement

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::resolved_statement::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_statement_hash(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_statement_hash<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_statement_hash(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_statement_hash(self: &Self) -> bool`
- `fn get_output_columns(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_sql_type::Owned>>`
- `fn set_output_columns(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::named_sql_type::Owned>) -> ::capnp::Result<()>`
- `fn init_output_columns(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_sql_type::Owned>`
- `fn has_output_columns(self: &Self) -> bool`
- `fn get_physical_accesses(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::physical_access::Owned>>`
- `fn set_physical_accesses(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::physical_access::Owned>) -> ::capnp::Result<()>`
- `fn init_physical_accesses(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::physical_access::Owned>`
- `fn has_physical_accesses(self: &Self) -> bool`
- `fn get_assignments(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::resolved_assignment::Owned>>`
- `fn set_assignments(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::resolved_assignment::Owned>) -> ::capnp::Result<()>`
- `fn init_assignments(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::resolved_assignment::Owned>`
- `fn has_assignments(self: &Self) -> bool`
- `fn get_text_collate_sites(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::text_collate_site::Owned>>`
- `fn set_text_collate_sites(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::text_collate_site::Owned>) -> ::capnp::Result<()>`
- `fn init_text_collate_sites(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::text_collate_site::Owned>`
- `fn has_text_collate_sites(self: &Self) -> bool`
- `fn get_integer_arith_sites(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::integer_arith_site::Owned>>`
- `fn set_integer_arith_sites(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::integer_arith_site::Owned>) -> ::capnp::Result<()>`
- `fn init_integer_arith_sites(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::integer_arith_site::Owned>`
- `fn has_integer_arith_sites(self: &Self) -> bool`
- `fn get_functions(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_functions<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_functions(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_functions(self: &Self) -> bool`
- `fn get_schema_action(self: Self) -> ::capnp::Result<crate::plugin_capnp::schema_action::Builder<'a>>`
- `fn set_schema_action(self: & mut Self, value: crate::plugin_capnp::schema_action::Reader) -> ::capnp::Result<()>`
- `fn init_schema_action(self: Self) -> crate::plugin_capnp::schema_action::Builder<'a>`
- `fn has_schema_action(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`
- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`



## bookclerk_plugin_abi::plugin_capnp::resolved_statement::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Copy, OwnedStruct, Owned, Pipelined

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Owned`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::resolved_statement::Pipeline

*Struct*

**Methods:**

- `fn get_schema_action(self: &Self) -> crate::plugin_capnp::schema_action::Pipeline`

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::resolved_statement::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_statement_hash(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_statement_hash(self: &Self) -> bool`
- `fn get_output_columns(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::named_sql_type::Owned>>`
- `fn has_output_columns(self: &Self) -> bool`
- `fn get_physical_accesses(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::physical_access::Owned>>`
- `fn has_physical_accesses(self: &Self) -> bool`
- `fn get_assignments(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::resolved_assignment::Owned>>`
- `fn has_assignments(self: &Self) -> bool`
- `fn get_text_collate_sites(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::text_collate_site::Owned>>`
- `fn has_text_collate_sites(self: &Self) -> bool`
- `fn get_integer_arith_sites(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::integer_arith_site::Owned>>`
- `fn has_integer_arith_sites(self: &Self) -> bool`
- `fn get_functions(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_functions(self: &Self) -> bool`
- `fn get_schema_action(self: Self) -> ::capnp::Result<crate::plugin_capnp::schema_action::Reader<'a>>`
- `fn has_schema_action(self: &Self) -> bool`

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



