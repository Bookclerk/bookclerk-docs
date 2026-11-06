# bookclerk_plugin_abi::plugin_capnp::db_value

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

#### Reader::has_text

```rust
pub fn has_text(&self) -> bool
```

#### Reader::has_bytes

```rust
pub fn has_bytes(&self) -> bool
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

#### Builder::set_null

```rust
pub fn set_null(&mut self, value: DbType)
```

#### Builder::set_boolean

```rust
pub fn set_boolean(&mut self, value: bool)
```

#### Builder::set_int64

```rust
pub fn set_int64(&mut self, value: i64)
```

#### Builder::set_float64

```rust
pub fn set_float64(&mut self, value: f64)
```

#### Builder::set_text

```rust
pub fn set_text(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_text

```rust
pub fn init_text(self, size: u32) -> Builder<'a>
```

#### Builder::has_text

```rust
pub fn has_text(&self) -> bool
```

#### Builder::set_bytes

```rust
pub fn set_bytes(&mut self, value: Reader<'_>)
```

#### Builder::init_bytes

```rust
pub fn init_bytes(self, size: u32) -> Builder<'a>
```

#### Builder::has_bytes

```rust
pub fn has_bytes(&self) -> bool
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
    Null(Result<DbType, NotInSchema>),
    Boolean(bool),
    Int64(i64),
    Float64(f64),
    Text(A0),
    Bytes(A1),
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
