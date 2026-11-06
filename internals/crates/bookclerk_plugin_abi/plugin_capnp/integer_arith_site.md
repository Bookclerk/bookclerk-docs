# bookclerk_plugin_abi::plugin_capnp::integer_arith_site

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

#### Reader::get_full

```rust
pub fn get_full(self) -> Result<Reader<'a>>
```

#### Reader::has_full

```rust
pub fn has_full(&self) -> bool
```

#### Reader::get_lhs

```rust
pub fn get_lhs(self) -> Result<Reader<'a>>
```

#### Reader::has_lhs

```rust
pub fn has_lhs(&self) -> bool
```

#### Reader::get_rhs

```rust
pub fn get_rhs(self) -> Result<Reader<'a>>
```

#### Reader::has_rhs

```rust
pub fn has_rhs(&self) -> bool
```

#### Reader::get_kind

```rust
pub fn get_kind(self) -> Result<IntegerArithKind, NotInSchema>
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

#### Builder::get_full

```rust
pub fn get_full(self) -> Result<Builder<'a>>
```

#### Builder::set_full

```rust
pub fn set_full(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_full

```rust
pub fn init_full(self) -> Builder<'a>
```

#### Builder::has_full

```rust
pub fn has_full(&self) -> bool
```

#### Builder::get_lhs

```rust
pub fn get_lhs(self) -> Result<Builder<'a>>
```

#### Builder::set_lhs

```rust
pub fn set_lhs(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_lhs

```rust
pub fn init_lhs(self) -> Builder<'a>
```

#### Builder::has_lhs

```rust
pub fn has_lhs(&self) -> bool
```

#### Builder::get_rhs

```rust
pub fn get_rhs(self) -> Result<Builder<'a>>
```

#### Builder::set_rhs

```rust
pub fn set_rhs(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_rhs

```rust
pub fn init_rhs(self) -> Builder<'a>
```

#### Builder::has_rhs

```rust
pub fn has_rhs(&self) -> bool
```

#### Builder::get_kind

```rust
pub fn get_kind(self) -> Result<IntegerArithKind, NotInSchema>
```

#### Builder::set_kind

```rust
pub fn set_kind(&mut self, value: IntegerArithKind)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_full

```rust
pub fn get_full(&self) -> Pipeline
```

#### Pipeline::get_lhs

```rust
pub fn get_lhs(&self) -> Pipeline
```

#### Pipeline::get_rhs

```rust
pub fn get_rhs(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
