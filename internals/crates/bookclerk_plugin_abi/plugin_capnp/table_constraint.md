# bookclerk_plugin_abi::plugin_capnp::table_constraint

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

#### Reader::has_primary_key

```rust
pub fn has_primary_key(&self) -> bool
```

#### Reader::has_unique

```rust
pub fn has_unique(&self) -> bool
```

#### Reader::has_check

```rust
pub fn has_check(&self) -> bool
```

#### Reader::has_foreign_key

```rust
pub fn has_foreign_key(&self) -> bool
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

#### Builder::set_primary_key

```rust
pub fn set_primary_key(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_primary_key

```rust
pub fn init_primary_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_primary_key

```rust
pub fn has_primary_key(&self) -> bool
```

#### Builder::set_unique

```rust
pub fn set_unique(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_unique

```rust
pub fn init_unique(self, size: u32) -> Builder<'a>
```

#### Builder::has_unique

```rust
pub fn has_unique(&self) -> bool
```

#### Builder::set_check

```rust
pub fn set_check(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_check

```rust
pub fn init_check(self, size: u32) -> Builder<'a>
```

#### Builder::has_check

```rust
pub fn has_check(&self) -> bool
```

#### Builder::set_foreign_key

```rust
pub fn set_foreign_key(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_foreign_key

```rust
pub fn init_foreign_key(self) -> Builder<'a>
```

#### Builder::has_foreign_key

```rust
pub fn has_foreign_key(&self) -> bool
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
pub enum Which<A0, A1, A2, A3> {
    PrimaryKey(A0),
    Unique(A1),
    Check(A2),
    ForeignKey(A3),
}
```

## Type aliases

### WhichReader

```rust
pub type WhichReader<'a> = Which<Result<Reader<'a>>, Result<Reader<'a>>, Result<Reader<'a>>, Result<Reader<'a>>>;
```

### WhichBuilder

```rust
pub type WhichBuilder<'a> = Which<Result<Builder<'a>>, Result<Builder<'a>>, Result<Builder<'a>>, Result<Builder<'a>>>;
```
