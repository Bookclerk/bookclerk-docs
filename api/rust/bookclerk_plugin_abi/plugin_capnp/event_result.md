# bookclerk_plugin_abi::plugin_capnp::event_result

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

#### Reader::has_ack

```rust
pub fn has_ack(&self) -> bool
```

#### Reader::has_retry

```rust
pub fn has_retry(&self) -> bool
```

#### Reader::has_reject

```rust
pub fn has_reject(&self) -> bool
```

#### Reader::has_dead_letter

```rust
pub fn has_dead_letter(&self) -> bool
```

#### Reader::has_suspended

```rust
pub fn has_suspended(&self) -> bool
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

#### Builder::set_ack

```rust
pub fn set_ack(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_ack

```rust
pub fn init_ack(self) -> Builder<'a>
```

#### Builder::has_ack

```rust
pub fn has_ack(&self) -> bool
```

#### Builder::set_retry

```rust
pub fn set_retry(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_retry

```rust
pub fn init_retry(self) -> Builder<'a>
```

#### Builder::has_retry

```rust
pub fn has_retry(&self) -> bool
```

#### Builder::set_reject

```rust
pub fn set_reject(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_reject

```rust
pub fn init_reject(self) -> Builder<'a>
```

#### Builder::has_reject

```rust
pub fn has_reject(&self) -> bool
```

#### Builder::set_dead_letter

```rust
pub fn set_dead_letter(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_dead_letter

```rust
pub fn init_dead_letter(self) -> Builder<'a>
```

#### Builder::has_dead_letter

```rust
pub fn has_dead_letter(&self) -> bool
```

#### Builder::set_suspended

```rust
pub fn set_suspended(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_suspended

```rust
pub fn init_suspended(self) -> Builder<'a>
```

#### Builder::has_suspended

```rust
pub fn has_suspended(&self) -> bool
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
    Ack(A0),
    Retry(A1),
    Reject(A2),
    DeadLetter(A3),
    Suspended(A4),
}
```

## Type aliases

### WhichReader

```rust
pub type WhichReader<'a> = Which<Result<Reader<'a>>, Result<Reader<'a>>, Result<Reader<'a>>, Result<Reader<'a>>, Result<Reader<'a>>>;
```

### WhichBuilder

```rust
pub type WhichBuilder<'a> = Which<Result<Builder<'a>>, Result<Builder<'a>>, Result<Builder<'a>>, Result<Builder<'a>>, Result<Builder<'a>>>;
```
