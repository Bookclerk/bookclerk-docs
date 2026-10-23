# bookclerk_plugin_abi::plugin_capnp::foreign_key_constraint

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

#### Reader::get_columns

```rust
pub fn get_columns(self) -> Result<Reader<'a>>
```

#### Reader::has_columns

```rust
pub fn has_columns(&self) -> bool
```

#### Reader::get_ref_table

```rust
pub fn get_ref_table(self) -> Result<Reader<'a>>
```

#### Reader::has_ref_table

```rust
pub fn has_ref_table(&self) -> bool
```

#### Reader::get_ref_columns

```rust
pub fn get_ref_columns(self) -> Result<Reader<'a>>
```

#### Reader::has_ref_columns

```rust
pub fn has_ref_columns(&self) -> bool
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

#### Builder::get_columns

```rust
pub fn get_columns(self) -> Result<Builder<'a>>
```

#### Builder::set_columns

```rust
pub fn set_columns(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_columns

```rust
pub fn init_columns(self, size: u32) -> Builder<'a>
```

#### Builder::has_columns

```rust
pub fn has_columns(&self) -> bool
```

#### Builder::get_ref_table

```rust
pub fn get_ref_table(self) -> Result<Builder<'a>>
```

#### Builder::set_ref_table

```rust
pub fn set_ref_table(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_ref_table

```rust
pub fn init_ref_table(self, size: u32) -> Builder<'a>
```

#### Builder::has_ref_table

```rust
pub fn has_ref_table(&self) -> bool
```

#### Builder::get_ref_columns

```rust
pub fn get_ref_columns(self) -> Result<Builder<'a>>
```

#### Builder::set_ref_columns

```rust
pub fn set_ref_columns(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_ref_columns

```rust
pub fn init_ref_columns(self, size: u32) -> Builder<'a>
```

#### Builder::has_ref_columns

```rust
pub fn has_ref_columns(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
