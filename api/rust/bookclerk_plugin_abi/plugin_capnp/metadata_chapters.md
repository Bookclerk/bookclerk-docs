# bookclerk_plugin_abi::plugin_capnp::metadata_chapters

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

#### Reader::get_found

```rust
pub fn get_found(self) -> bool
```

#### Reader::get_chapters

```rust
pub fn get_chapters(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_chapters

```rust
pub fn has_chapters(&self) -> bool
```

#### Reader::get_brand_intro_ms

```rust
pub fn get_brand_intro_ms(self) -> u64
```

#### Reader::get_brand_outro_ms

```rust
pub fn get_brand_outro_ms(self) -> u64
```

#### Reader::get_runtime_ms

```rust
pub fn get_runtime_ms(self) -> u64
```

#### Reader::get_accurate

```rust
pub fn get_accurate(self) -> bool
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

#### Builder::get_found

```rust
pub fn get_found(self) -> bool
```

#### Builder::set_found

```rust
pub fn set_found(&mut self, value: bool)
```

#### Builder::get_chapters

```rust
pub fn get_chapters(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_chapters

```rust
pub fn set_chapters(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_chapters

```rust
pub fn init_chapters(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_chapters

```rust
pub fn has_chapters(&self) -> bool
```

#### Builder::get_brand_intro_ms

```rust
pub fn get_brand_intro_ms(self) -> u64
```

#### Builder::set_brand_intro_ms

```rust
pub fn set_brand_intro_ms(&mut self, value: u64)
```

#### Builder::get_brand_outro_ms

```rust
pub fn get_brand_outro_ms(self) -> u64
```

#### Builder::set_brand_outro_ms

```rust
pub fn set_brand_outro_ms(&mut self, value: u64)
```

#### Builder::get_runtime_ms

```rust
pub fn get_runtime_ms(self) -> u64
```

#### Builder::set_runtime_ms

```rust
pub fn set_runtime_ms(&mut self, value: u64)
```

#### Builder::get_accurate

```rust
pub fn get_accurate(self) -> bool
```

#### Builder::set_accurate

```rust
pub fn set_accurate(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
