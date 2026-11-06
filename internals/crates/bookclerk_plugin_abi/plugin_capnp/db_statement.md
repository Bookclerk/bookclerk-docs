# bookclerk_plugin_abi::plugin_capnp::db_statement

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

#### Reader::get_sql

```rust
pub fn get_sql(self) -> Result<Reader<'a>>
```

#### Reader::has_sql

```rust
pub fn has_sql(&self) -> bool
```

#### Reader::get_parameters

```rust
pub fn get_parameters(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_parameters

```rust
pub fn has_parameters(&self) -> bool
```

#### Reader::get_kind

```rust
pub fn get_kind(self) -> Result<DbStatementKind, NotInSchema>
```

#### Reader::get_max_rows

```rust
pub fn get_max_rows(self) -> u32
```

#### Reader::get_result_selection

```rust
pub fn get_result_selection(self) -> Result<DbResultSelection, NotInSchema>
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

#### Builder::get_sql

```rust
pub fn get_sql(self) -> Result<Builder<'a>>
```

#### Builder::set_sql

```rust
pub fn set_sql(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_sql

```rust
pub fn init_sql(self, size: u32) -> Builder<'a>
```

#### Builder::has_sql

```rust
pub fn has_sql(&self) -> bool
```

#### Builder::get_parameters

```rust
pub fn get_parameters(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_parameters

```rust
pub fn set_parameters(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_parameters

```rust
pub fn init_parameters(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_parameters

```rust
pub fn has_parameters(&self) -> bool
```

#### Builder::get_kind

```rust
pub fn get_kind(self) -> Result<DbStatementKind, NotInSchema>
```

#### Builder::set_kind

```rust
pub fn set_kind(&mut self, value: DbStatementKind)
```

#### Builder::get_max_rows

```rust
pub fn get_max_rows(self) -> u32
```

#### Builder::set_max_rows

```rust
pub fn set_max_rows(&mut self, value: u32)
```

#### Builder::get_result_selection

```rust
pub fn get_result_selection(self) -> Result<DbResultSelection, NotInSchema>
```

#### Builder::set_result_selection

```rust
pub fn set_result_selection(&mut self, value: DbResultSelection)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
