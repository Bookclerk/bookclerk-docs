# bookclerk_plugin_abi::plugin_capnp::plugin_migration_op

| Item | Kind | Summary |
| --- | --- | --- |
| [`Owned`](#owned) | struct |  |
| [`Reader`](#reader) | struct |  |
| [`Builder`](#builder) | struct |  |
| [`Pipeline`](#pipeline) | struct |  |
| [`Which`](#which) | enum |  |
| [`WhichReader`](#whichreader) | type alias |  |
| [`WhichBuilder`](#whichbuilder) | type alias |  |

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

#### Reader::has_schema

```rust
pub fn has_schema(&self) -> bool
```

#### Reader::has_data

```rust
pub fn has_data(&self) -> bool
```

#### Reader::which

```rust
pub fn which(self) -> Result<WhichReader<'a>, NotInSchema>
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

#### Builder::set_schema

```rust
pub fn set_schema(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_schema

```rust
pub fn init_schema(self, size: u32) -> Builder<'a>
```

#### Builder::has_schema

```rust
pub fn has_schema(&self) -> bool
```

#### Builder::set_data

```rust
pub fn set_data(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_data

```rust
pub fn init_data(self, size: u32) -> Builder<'a>
```

#### Builder::has_data

```rust
pub fn has_data(&self) -> bool
```

#### Builder::which

```rust
pub fn which(self) -> Result<WhichBuilder<'a>, NotInSchema>
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`

## Enums

### Which

```rust
pub enum Which<A0, A1> {
    Schema(A0),
    Data(A1),
}
```

## Type aliases

### WhichReader

```rust
pub type WhichReader<'a> = Which<Result<Reader<'a>>, Result<Reader<'a>>>;
```

### WhichBuilder

```rust
pub type WhichBuilder<'a> = Which<Result<Builder<'a>>, Result<Builder<'a>>>;
```
