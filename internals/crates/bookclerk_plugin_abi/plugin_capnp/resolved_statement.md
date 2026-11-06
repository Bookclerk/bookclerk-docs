# bookclerk_plugin_abi::plugin_capnp::resolved_statement

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

#### Reader::get_statement_hash

```rust
pub fn get_statement_hash(self) -> Result<Reader<'a>>
```

#### Reader::has_statement_hash

```rust
pub fn has_statement_hash(&self) -> bool
```

#### Reader::get_output_columns

```rust
pub fn get_output_columns(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_output_columns

```rust
pub fn has_output_columns(&self) -> bool
```

#### Reader::get_physical_accesses

```rust
pub fn get_physical_accesses(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_physical_accesses

```rust
pub fn has_physical_accesses(&self) -> bool
```

#### Reader::get_assignments

```rust
pub fn get_assignments(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_assignments

```rust
pub fn has_assignments(&self) -> bool
```

#### Reader::get_text_collate_sites

```rust
pub fn get_text_collate_sites(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_text_collate_sites

```rust
pub fn has_text_collate_sites(&self) -> bool
```

#### Reader::get_integer_arith_sites

```rust
pub fn get_integer_arith_sites(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_integer_arith_sites

```rust
pub fn has_integer_arith_sites(&self) -> bool
```

#### Reader::get_functions

```rust
pub fn get_functions(self) -> Result<Reader<'a>>
```

#### Reader::has_functions

```rust
pub fn has_functions(&self) -> bool
```

#### Reader::get_schema_action

```rust
pub fn get_schema_action(self) -> Result<Reader<'a>>
```

#### Reader::has_schema_action

```rust
pub fn has_schema_action(&self) -> bool
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

#### Builder::get_statement_hash

```rust
pub fn get_statement_hash(self) -> Result<Builder<'a>>
```

#### Builder::set_statement_hash

```rust
pub fn set_statement_hash(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_statement_hash

```rust
pub fn init_statement_hash(self, size: u32) -> Builder<'a>
```

#### Builder::has_statement_hash

```rust
pub fn has_statement_hash(&self) -> bool
```

#### Builder::get_output_columns

```rust
pub fn get_output_columns(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_output_columns

```rust
pub fn set_output_columns(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_output_columns

```rust
pub fn init_output_columns(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_output_columns

```rust
pub fn has_output_columns(&self) -> bool
```

#### Builder::get_physical_accesses

```rust
pub fn get_physical_accesses(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_physical_accesses

```rust
pub fn set_physical_accesses(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_physical_accesses

```rust
pub fn init_physical_accesses(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_physical_accesses

```rust
pub fn has_physical_accesses(&self) -> bool
```

#### Builder::get_assignments

```rust
pub fn get_assignments(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_assignments

```rust
pub fn set_assignments(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_assignments

```rust
pub fn init_assignments(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_assignments

```rust
pub fn has_assignments(&self) -> bool
```

#### Builder::get_text_collate_sites

```rust
pub fn get_text_collate_sites(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_text_collate_sites

```rust
pub fn set_text_collate_sites(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_text_collate_sites

```rust
pub fn init_text_collate_sites(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_text_collate_sites

```rust
pub fn has_text_collate_sites(&self) -> bool
```

#### Builder::get_integer_arith_sites

```rust
pub fn get_integer_arith_sites(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_integer_arith_sites

```rust
pub fn set_integer_arith_sites(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_integer_arith_sites

```rust
pub fn init_integer_arith_sites(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_integer_arith_sites

```rust
pub fn has_integer_arith_sites(&self) -> bool
```

#### Builder::get_functions

```rust
pub fn get_functions(self) -> Result<Builder<'a>>
```

#### Builder::set_functions

```rust
pub fn set_functions(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_functions

```rust
pub fn init_functions(self, size: u32) -> Builder<'a>
```

#### Builder::has_functions

```rust
pub fn has_functions(&self) -> bool
```

#### Builder::get_schema_action

```rust
pub fn get_schema_action(self) -> Result<Builder<'a>>
```

#### Builder::set_schema_action

```rust
pub fn set_schema_action(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_schema_action

```rust
pub fn init_schema_action(self) -> Builder<'a>
```

#### Builder::has_schema_action

```rust
pub fn has_schema_action(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_schema_action

```rust
pub fn get_schema_action(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
