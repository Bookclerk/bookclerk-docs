# bookclerk_plugin_abi::plugin_capnp::job_outcome

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

#### Reader::has_completed

```rust
pub fn has_completed(&self) -> bool
```

#### Reader::has_retryable

```rust
pub fn has_retryable(&self) -> bool
```

#### Reader::has_rejected

```rust
pub fn has_rejected(&self) -> bool
```

#### Reader::has_cancelled

```rust
pub fn has_cancelled(&self) -> bool
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

#### Builder::set_completed

```rust
pub fn set_completed(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_completed

```rust
pub fn init_completed(self) -> Builder<'a>
```

#### Builder::has_completed

```rust
pub fn has_completed(&self) -> bool
```

#### Builder::set_retryable

```rust
pub fn set_retryable(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_retryable

```rust
pub fn init_retryable(self) -> Builder<'a>
```

#### Builder::has_retryable

```rust
pub fn has_retryable(&self) -> bool
```

#### Builder::set_rejected

```rust
pub fn set_rejected(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_rejected

```rust
pub fn init_rejected(self) -> Builder<'a>
```

#### Builder::has_rejected

```rust
pub fn has_rejected(&self) -> bool
```

#### Builder::set_cancelled

```rust
pub fn set_cancelled(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_cancelled

```rust
pub fn init_cancelled(self) -> Builder<'a>
```

#### Builder::has_cancelled

```rust
pub fn has_cancelled(&self) -> bool
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
    Completed(A0),
    Retryable(A1),
    Rejected(A2),
    Cancelled(A3),
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
