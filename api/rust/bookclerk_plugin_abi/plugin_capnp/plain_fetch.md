# bookclerk_plugin_abi::plugin_capnp::plain_fetch

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

#### Reader::get_parts

```rust
pub fn get_parts(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_parts

```rust
pub fn has_parts(&self) -> bool
```

#### Reader::get_m4b_path

```rust
pub fn get_m4b_path(self) -> Result<Reader<'a>>
```

#### Reader::has_m4b_path

```rust
pub fn has_m4b_path(&self) -> bool
```

#### Reader::get_cover_path

```rust
pub fn get_cover_path(self) -> Result<Reader<'a>>
```

#### Reader::has_cover_path

```rust
pub fn has_cover_path(&self) -> bool
```

#### Reader::get_chapters

```rust
pub fn get_chapters(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_chapters

```rust
pub fn has_chapters(&self) -> bool
```

#### Reader::get_pdf_url

```rust
pub fn get_pdf_url(self) -> Result<Reader<'a>>
```

#### Reader::has_pdf_url

```rust
pub fn has_pdf_url(&self) -> bool
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

#### Builder::get_parts

```rust
pub fn get_parts(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_parts

```rust
pub fn set_parts(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_parts

```rust
pub fn init_parts(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_parts

```rust
pub fn has_parts(&self) -> bool
```

#### Builder::get_m4b_path

```rust
pub fn get_m4b_path(self) -> Result<Builder<'a>>
```

#### Builder::set_m4b_path

```rust
pub fn set_m4b_path(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_m4b_path

```rust
pub fn init_m4b_path(self, size: u32) -> Builder<'a>
```

#### Builder::has_m4b_path

```rust
pub fn has_m4b_path(&self) -> bool
```

#### Builder::get_cover_path

```rust
pub fn get_cover_path(self) -> Result<Builder<'a>>
```

#### Builder::set_cover_path

```rust
pub fn set_cover_path(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cover_path

```rust
pub fn init_cover_path(self, size: u32) -> Builder<'a>
```

#### Builder::has_cover_path

```rust
pub fn has_cover_path(&self) -> bool
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

#### Builder::get_pdf_url

```rust
pub fn get_pdf_url(self) -> Result<Builder<'a>>
```

#### Builder::set_pdf_url

```rust
pub fn set_pdf_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_pdf_url

```rust
pub fn init_pdf_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_pdf_url

```rust
pub fn has_pdf_url(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
