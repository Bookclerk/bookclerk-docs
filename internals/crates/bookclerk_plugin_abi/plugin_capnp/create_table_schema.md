# bookclerk_plugin_abi::plugin_capnp::create_table_schema

| Item | Kind | Summary |
| --- | --- | --- |
| [`Owned`](#owned) | struct |  |
| [`Reader`](#reader) | struct |  |
| [`Builder`](#builder) | struct |  |
| [`Pipeline`](#pipeline) | struct |  |

## Structs

### Owned

```rust
pub struct Owned(/* private */);
```

Implements: `Clone`, `Copy`, `Introspect`, `Owned`, `OwnedStruct`, `Pipelined`, `SetterInput<Owned>`

### Reader

```rust
pub struct Reader<'a> {
    /* private fields */
}
```

#### Reader::reborrow

```rust
pub fn reborrow(&self) -> Reader<'_>
```

#### Reader::total_size

```rust
pub fn total_size(&self) -> Result<MessageSize>
```

#### Reader::get_table

```rust
pub fn get_table(self) -> Result<Reader<'a>>
```

#### Reader::has_table

```rust
pub fn has_table(&self) -> bool
```

#### Reader::get_columns

```rust
pub fn get_columns(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_columns

```rust
pub fn has_columns(&self) -> bool
```

#### Reader::get_identity_column

```rust
pub fn get_identity_column(self) -> Result<Reader<'a>>
```

#### Reader::has_identity_column

```rust
pub fn has_identity_column(&self) -> bool
```

#### Reader::get_column_not_null

```rust
pub fn get_column_not_null(self) -> Result<Reader<'a, bool>>
```

#### Reader::has_column_not_null

```rust
pub fn has_column_not_null(&self) -> bool
```

#### Reader::get_column_unique

```rust
pub fn get_column_unique(self) -> Result<Reader<'a, bool>>
```

#### Reader::has_column_unique

```rust
pub fn has_column_unique(&self) -> bool
```

#### Reader::get_column_primary_key

```rust
pub fn get_column_primary_key(self) -> Result<Reader<'a, bool>>
```

#### Reader::has_column_primary_key

```rust
pub fn has_column_primary_key(&self) -> bool
```

#### Reader::get_column_defaults

```rust
pub fn get_column_defaults(self) -> Result<Reader<'a>>
```

#### Reader::has_column_defaults

```rust
pub fn has_column_defaults(&self) -> bool
```

#### Reader::get_column_checks

```rust
pub fn get_column_checks(self) -> Result<Reader<'a>>
```

#### Reader::has_column_checks

```rust
pub fn has_column_checks(&self) -> bool
```

#### Reader::get_column_references

```rust
pub fn get_column_references(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_column_references

```rust
pub fn has_column_references(&self) -> bool
```

#### Reader::get_table_constraints

```rust
pub fn get_table_constraints(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_table_constraints

```rust
pub fn has_table_constraints(&self) -> bool
```

Implements: `Clone`, `Copy`, `Debug`, `From<Reader<'a>>`, `From<StructReader<'a>>`, `FromPointerReader<'a>`, `HasTypeId`, `Imbue<'a>`, `IntoInternalStructReader<'a>`, `SetterInput<Owned>`

### Builder

```rust
pub struct Builder<'a> {
    /* private fields */
}
```

#### Builder::into_reader

```rust
pub fn into_reader(self) -> Reader<'a>
```

#### Builder::reborrow

```rust
pub fn reborrow(&mut self) -> Builder<'_>
```

#### Builder::reborrow_as_reader

```rust
pub fn reborrow_as_reader(&self) -> Reader<'_>
```

#### Builder::total_size

```rust
pub fn total_size(&self) -> Result<MessageSize>
```

#### Builder::get_table

```rust
pub fn get_table(self) -> Result<Builder<'a>>
```

#### Builder::set_table

```rust
pub fn set_table(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_table

```rust
pub fn init_table(self, size: u32) -> Builder<'a>
```

#### Builder::has_table

```rust
pub fn has_table(&self) -> bool
```

#### Builder::get_columns

```rust
pub fn get_columns(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_columns

```rust
pub fn set_columns(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_columns

```rust
pub fn init_columns(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_columns

```rust
pub fn has_columns(&self) -> bool
```

#### Builder::get_identity_column

```rust
pub fn get_identity_column(self) -> Result<Builder<'a>>
```

#### Builder::set_identity_column

```rust
pub fn set_identity_column(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_identity_column

```rust
pub fn init_identity_column(self, size: u32) -> Builder<'a>
```

#### Builder::has_identity_column

```rust
pub fn has_identity_column(&self) -> bool
```

#### Builder::get_column_not_null

```rust
pub fn get_column_not_null(self) -> Result<Builder<'a, bool>>
```

#### Builder::set_column_not_null

```rust
pub fn set_column_not_null(&mut self, value: impl SetterInput<Owned<bool>>) -> Result<()>
```

#### Builder::init_column_not_null

```rust
pub fn init_column_not_null(self, size: u32) -> Builder<'a, bool>
```

#### Builder::has_column_not_null

```rust
pub fn has_column_not_null(&self) -> bool
```

#### Builder::get_column_unique

```rust
pub fn get_column_unique(self) -> Result<Builder<'a, bool>>
```

#### Builder::set_column_unique

```rust
pub fn set_column_unique(&mut self, value: impl SetterInput<Owned<bool>>) -> Result<()>
```

#### Builder::init_column_unique

```rust
pub fn init_column_unique(self, size: u32) -> Builder<'a, bool>
```

#### Builder::has_column_unique

```rust
pub fn has_column_unique(&self) -> bool
```

#### Builder::get_column_primary_key

```rust
pub fn get_column_primary_key(self) -> Result<Builder<'a, bool>>
```

#### Builder::set_column_primary_key

```rust
pub fn set_column_primary_key(
    &mut self,
    value: impl SetterInput<Owned<bool>>,
) -> Result<()>
```

#### Builder::init_column_primary_key

```rust
pub fn init_column_primary_key(self, size: u32) -> Builder<'a, bool>
```

#### Builder::has_column_primary_key

```rust
pub fn has_column_primary_key(&self) -> bool
```

#### Builder::get_column_defaults

```rust
pub fn get_column_defaults(self) -> Result<Builder<'a>>
```

#### Builder::set_column_defaults

```rust
pub fn set_column_defaults(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_column_defaults

```rust
pub fn init_column_defaults(self, size: u32) -> Builder<'a>
```

#### Builder::has_column_defaults

```rust
pub fn has_column_defaults(&self) -> bool
```

#### Builder::get_column_checks

```rust
pub fn get_column_checks(self) -> Result<Builder<'a>>
```

#### Builder::set_column_checks

```rust
pub fn set_column_checks(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_column_checks

```rust
pub fn init_column_checks(self, size: u32) -> Builder<'a>
```

#### Builder::has_column_checks

```rust
pub fn has_column_checks(&self) -> bool
```

#### Builder::get_column_references

```rust
pub fn get_column_references(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_column_references

```rust
pub fn set_column_references(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_column_references

```rust
pub fn init_column_references(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_column_references

```rust
pub fn has_column_references(&self) -> bool
```

#### Builder::get_table_constraints

```rust
pub fn get_table_constraints(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_table_constraints

```rust
pub fn set_table_constraints(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_table_constraints

```rust
pub fn init_table_constraints(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_table_constraints

```rust
pub fn has_table_constraints(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
