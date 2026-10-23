**bookclerk_plugin_abi > plugin_capnp > create_table_schema**

# Module: plugin_capnp::create_table_schema

## Contents

**Structs**

- [`Builder`](#builder)
- [`Owned`](#owned)
- [`Pipeline`](#pipeline)
- [`Reader`](#reader)

---

## bookclerk_plugin_abi::plugin_capnp::create_table_schema::Builder

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn into_reader(self: Self) -> Reader<'a>`
- `fn reborrow(self: & mut Self) -> Builder`
- `fn reborrow_as_reader(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_table(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_table<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_table(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_table(self: &Self) -> bool`
- `fn get_columns(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_sql_type::Owned>>`
- `fn set_columns(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::named_sql_type::Owned>) -> ::capnp::Result<()>`
- `fn init_columns(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::named_sql_type::Owned>`
- `fn has_columns(self: &Self) -> bool`
- `fn get_identity_column(self: Self) -> ::capnp::Result<::capnp::text::Builder<'a>>`
- `fn set_identity_column<impl ::capnp::traits::SetterInput<::capnp::text::Owned>>(self: & mut Self, value: impl Trait)`
- `fn init_identity_column(self: Self, size: u32) -> ::capnp::text::Builder<'a>`
- `fn has_identity_column(self: &Self) -> bool`
- `fn get_column_not_null(self: Self) -> ::capnp::Result<::capnp::primitive_list::Builder<'a, bool>>`
- `fn set_column_not_null<impl ::capnp::traits::SetterInput<::capnp::primitive_list::Owned<bool>>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_column_not_null(self: Self, size: u32) -> ::capnp::primitive_list::Builder<'a, bool>`
- `fn has_column_not_null(self: &Self) -> bool`
- `fn get_column_unique(self: Self) -> ::capnp::Result<::capnp::primitive_list::Builder<'a, bool>>`
- `fn set_column_unique<impl ::capnp::traits::SetterInput<::capnp::primitive_list::Owned<bool>>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_column_unique(self: Self, size: u32) -> ::capnp::primitive_list::Builder<'a, bool>`
- `fn has_column_unique(self: &Self) -> bool`
- `fn get_column_primary_key(self: Self) -> ::capnp::Result<::capnp::primitive_list::Builder<'a, bool>>`
- `fn set_column_primary_key<impl ::capnp::traits::SetterInput<::capnp::primitive_list::Owned<bool>>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_column_primary_key(self: Self, size: u32) -> ::capnp::primitive_list::Builder<'a, bool>`
- `fn has_column_primary_key(self: &Self) -> bool`
- `fn get_column_defaults(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_column_defaults<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_column_defaults(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_column_defaults(self: &Self) -> bool`
- `fn get_column_checks(self: Self) -> ::capnp::Result<::capnp::text_list::Builder<'a>>`
- `fn set_column_checks<impl ::capnp::traits::SetterInput<::capnp::text_list::Owned>>(self: & mut Self, value: impl Trait) -> ::capnp::Result<()>`
- `fn init_column_checks(self: Self, size: u32) -> ::capnp::text_list::Builder<'a>`
- `fn has_column_checks(self: &Self) -> bool`
- `fn get_column_references(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::optional_column_reference::Owned>>`
- `fn set_column_references(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::optional_column_reference::Owned>) -> ::capnp::Result<()>`
- `fn init_column_references(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::optional_column_reference::Owned>`
- `fn has_column_references(self: &Self) -> bool`
- `fn get_table_constraints(self: Self) -> ::capnp::Result<::capnp::struct_list::Builder<'a, crate::plugin_capnp::table_constraint::Owned>>`
- `fn set_table_constraints(self: & mut Self, value: ::capnp::struct_list::Reader<crate::plugin_capnp::table_constraint::Owned>) -> ::capnp::Result<()>`
- `fn init_table_constraints(self: Self, size: u32) -> ::capnp::struct_list::Builder<'a, crate::plugin_capnp::table_constraint::Owned>`
- `fn has_table_constraints(self: &Self) -> bool`

**Traits:** HasTypeId, HasStructSize

**Trait Implementations:**

- **ImbueMut**
  - `fn imbue_mut(self: & mut Self, cap_table: &'a  mut ::capnp::private::layout::CapTable)`
- **FromPointerBuilder**
  - `fn init_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, _size: u32) -> Self`
  - `fn get_from_pointer(builder: ::capnp::private::layout::PointerBuilder<'a>, default: ::core::option::Option<&'a [::capnp::Word]>) -> ::capnp::Result<Self>`
- **From**
  - `fn from(builder: ::capnp::private::layout::StructBuilder<'a>) -> Self`



## bookclerk_plugin_abi::plugin_capnp::create_table_schema::Owned

*Struct*

**Tuple Struct**: `()`

**Traits:** Owned, Pipelined, Copy, OwnedStruct

**Trait Implementations:**

- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **Clone**
  - `fn clone(self: &Self) -> Owned`



## bookclerk_plugin_abi::plugin_capnp::create_table_schema::Pipeline

*Struct*

**Trait Implementations:**

- **FromTypelessPipeline**
  - `fn new(typeless: ::capnp::any_pointer::Pipeline) -> Self`



## bookclerk_plugin_abi::plugin_capnp::create_table_schema::Reader

*Struct*

**Generic Parameters:**
- 'a

**Methods:**

- `fn reborrow(self: &Self) -> Reader`
- `fn total_size(self: &Self) -> ::capnp::Result<::capnp::MessageSize>`
- `fn get_table(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_table(self: &Self) -> bool`
- `fn get_columns(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::named_sql_type::Owned>>`
- `fn has_columns(self: &Self) -> bool`
- `fn get_identity_column(self: Self) -> ::capnp::Result<::capnp::text::Reader<'a>>`
- `fn has_identity_column(self: &Self) -> bool`
- `fn get_column_not_null(self: Self) -> ::capnp::Result<::capnp::primitive_list::Reader<'a, bool>>`
- `fn has_column_not_null(self: &Self) -> bool`
- `fn get_column_unique(self: Self) -> ::capnp::Result<::capnp::primitive_list::Reader<'a, bool>>`
- `fn has_column_unique(self: &Self) -> bool`
- `fn get_column_primary_key(self: Self) -> ::capnp::Result<::capnp::primitive_list::Reader<'a, bool>>`
- `fn has_column_primary_key(self: &Self) -> bool`
- `fn get_column_defaults(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_column_defaults(self: &Self) -> bool`
- `fn get_column_checks(self: Self) -> ::capnp::Result<::capnp::text_list::Reader<'a>>`
- `fn has_column_checks(self: &Self) -> bool`
- `fn get_column_references(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::optional_column_reference::Owned>>`
- `fn has_column_references(self: &Self) -> bool`
- `fn get_table_constraints(self: Self) -> ::capnp::Result<::capnp::struct_list::Reader<'a, crate::plugin_capnp::table_constraint::Owned>>`
- `fn has_table_constraints(self: &Self) -> bool`

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



