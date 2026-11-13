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

#### Reader::get_media_kind

```rust
pub fn get_media_kind(self) -> Result<MediaKind, NotInSchema>
```

#### Reader::get_files

```rust
pub fn get_files(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_files

```rust
pub fn has_files(&self) -> bool
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

#### Reader::get_refreshed_credentials

```rust
pub fn get_refreshed_credentials(self) -> Result<Reader<'a>>
```

#### Reader::has_refreshed_credentials

```rust
pub fn has_refreshed_credentials(&self) -> bool
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

#### Builder::get_media_kind

```rust
pub fn get_media_kind(self) -> Result<MediaKind, NotInSchema>
```

#### Builder::set_media_kind

```rust
pub fn set_media_kind(&mut self, value: MediaKind)
```

#### Builder::get_files

```rust
pub fn get_files(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_files

```rust
pub fn set_files(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_files

```rust
pub fn init_files(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_files

```rust
pub fn has_files(&self) -> bool
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

#### Builder::get_refreshed_credentials

```rust
pub fn get_refreshed_credentials(self) -> Result<Builder<'a>>
```

#### Builder::set_refreshed_credentials

```rust
pub fn set_refreshed_credentials(&mut self, value: Reader<'_>)
```

#### Builder::init_refreshed_credentials

```rust
pub fn init_refreshed_credentials(self, size: u32) -> Builder<'a>
```

#### Builder::has_refreshed_credentials

```rust
pub fn has_refreshed_credentials(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
