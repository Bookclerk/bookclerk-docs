# bookclerk_plugin_abi::plugin_capnp::resolved_assignment

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

#### Reader::get_column

```rust
pub fn get_column(self) -> Result<Reader<'a>>
```

#### Reader::has_column

```rust
pub fn has_column(&self) -> bool
```

#### Reader::get_dest

```rust
pub fn get_dest(self) -> Result<ResolvedSqlType, NotInSchema>
```

#### Reader::get_source

```rust
pub fn get_source(self) -> Result<ResolvedSqlType, NotInSchema>
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

#### Builder::get_column

```rust
pub fn get_column(self) -> Result<Builder<'a>>
```

#### Builder::set_column

```rust
pub fn set_column(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_column

```rust
pub fn init_column(self, size: u32) -> Builder<'a>
```

#### Builder::has_column

```rust
pub fn has_column(&self) -> bool
```

#### Builder::get_dest

```rust
pub fn get_dest(self) -> Result<ResolvedSqlType, NotInSchema>
```

#### Builder::set_dest

```rust
pub fn set_dest(&mut self, value: ResolvedSqlType)
```

#### Builder::get_source

```rust
pub fn get_source(self) -> Result<ResolvedSqlType, NotInSchema>
```

#### Builder::set_source

```rust
pub fn set_source(&mut self, value: ResolvedSqlType)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
