# bookclerk_plugin_abi::plugin_capnp::ai_text_embeddings_output

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

#### Reader::get_dimensions

```rust
pub fn get_dimensions(self) -> u32
```

#### Reader::get_vectors

```rust
pub fn get_vectors(self) -> Result<Reader<'a>>
```

#### Reader::has_vectors

```rust
pub fn has_vectors(&self) -> bool
```

#### Reader::get_revision

```rust
pub fn get_revision(self) -> Result<Reader<'a>>
```

#### Reader::has_revision

```rust
pub fn has_revision(&self) -> bool
```

#### Reader::get_fallbacks

```rust
pub fn get_fallbacks(self) -> Result<Reader<'a, u32>>
```

#### Reader::has_fallbacks

```rust
pub fn has_fallbacks(&self) -> bool
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

#### Builder::get_dimensions

```rust
pub fn get_dimensions(self) -> u32
```

#### Builder::set_dimensions

```rust
pub fn set_dimensions(&mut self, value: u32)
```

#### Builder::get_vectors

```rust
pub fn get_vectors(self) -> Result<Builder<'a>>
```

#### Builder::set_vectors

```rust
pub fn set_vectors(&mut self, value: Reader<'_>)
```

#### Builder::init_vectors

```rust
pub fn init_vectors(self, size: u32) -> Builder<'a>
```

#### Builder::has_vectors

```rust
pub fn has_vectors(&self) -> bool
```

#### Builder::get_revision

```rust
pub fn get_revision(self) -> Result<Builder<'a>>
```

#### Builder::set_revision

```rust
pub fn set_revision(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_revision

```rust
pub fn init_revision(self, size: u32) -> Builder<'a>
```

#### Builder::has_revision

```rust
pub fn has_revision(&self) -> bool
```

#### Builder::get_fallbacks

```rust
pub fn get_fallbacks(self) -> Result<Builder<'a, u32>>
```

#### Builder::set_fallbacks

```rust
pub fn set_fallbacks(&mut self, value: impl SetterInput<Owned<u32>>) -> Result<()>
```

#### Builder::init_fallbacks

```rust
pub fn init_fallbacks(self, size: u32) -> Builder<'a, u32>
```

#### Builder::has_fallbacks

```rust
pub fn has_fallbacks(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
