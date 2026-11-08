# bookclerk_plugin_abi::plugin_capnp::execute_stream_frame

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

#### Reader::has_head

```rust
pub fn has_head(&self) -> bool
```

#### Reader::has_statement

```rust
pub fn has_statement(&self) -> bool
```

#### Reader::has_rows

```rust
pub fn has_rows(&self) -> bool
```

#### Reader::has_end

```rust
pub fn has_end(&self) -> bool
```

#### Reader::has_err

```rust
pub fn has_err(&self) -> bool
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

#### Builder::set_head

```rust
pub fn set_head(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_head

```rust
pub fn init_head(self) -> Builder<'a>
```

#### Builder::has_head

```rust
pub fn has_head(&self) -> bool
```

#### Builder::set_statement

```rust
pub fn set_statement(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_statement

```rust
pub fn init_statement(self) -> Builder<'a>
```

#### Builder::has_statement

```rust
pub fn has_statement(&self) -> bool
```

#### Builder::set_rows

```rust
pub fn set_rows(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_rows

```rust
pub fn init_rows(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_rows

```rust
pub fn has_rows(&self) -> bool
```

#### Builder::set_end

```rust
pub fn set_end(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_end

```rust
pub fn init_end(self) -> Builder<'a>
```

#### Builder::has_end

```rust
pub fn has_end(&self) -> bool
```

#### Builder::set_err

```rust
pub fn set_err(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_err

```rust
pub fn init_err(self) -> Builder<'a>
```

#### Builder::has_err

```rust
pub fn has_err(&self) -> bool
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
pub enum Which<A0, A1, A2, A3, A4> {
    Head(A0),
    Statement(A1),
    Rows(A2),
    End(A3),
    Err(A4),
}
```

## Type aliases

### WhichReader

```rust
pub type WhichReader<'a> = Which<Result<Reader<'a>>, Result<Reader<'a>>, Result<Reader<'a, Owned>>, Result<Reader<'a>>, Result<Reader<'a>>>;
```

### WhichBuilder

```rust
pub type WhichBuilder<'a> = Which<Result<Builder<'a>>, Result<Builder<'a>>, Result<Builder<'a, Owned>>, Result<Builder<'a>>, Result<Builder<'a>>>;
```
