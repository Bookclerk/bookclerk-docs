# bookclerk_plugin_abi::plugin_capnp::cover_image

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

#### Reader::get_url

```rust
pub fn get_url(self) -> Result<Reader<'a>>
```

#### Reader::has_url

```rust
pub fn has_url(&self) -> bool
```

#### Reader::get_stored

```rust
pub fn get_stored(self) -> bool
```

#### Reader::get_content_type

```rust
pub fn get_content_type(self) -> Result<Reader<'a>>
```

#### Reader::has_content_type

```rust
pub fn has_content_type(&self) -> bool
```

#### Reader::get_error

```rust
pub fn get_error(self) -> Result<Reader<'a>>
```

#### Reader::has_error

```rust
pub fn has_error(&self) -> bool
```

#### Reader::get_not_modified

```rust
pub fn get_not_modified(self) -> bool
```

#### Reader::get_etag

```rust
pub fn get_etag(self) -> Result<Reader<'a>>
```

#### Reader::has_etag

```rust
pub fn has_etag(&self) -> bool
```

#### Reader::get_last_modified

```rust
pub fn get_last_modified(self) -> Result<Reader<'a>>
```

#### Reader::has_last_modified

```rust
pub fn has_last_modified(&self) -> bool
```

#### Reader::get_cache_control

```rust
pub fn get_cache_control(self) -> Result<Reader<'a>>
```

#### Reader::has_cache_control

```rust
pub fn has_cache_control(&self) -> bool
```

#### Reader::get_expires

```rust
pub fn get_expires(self) -> Result<Reader<'a>>
```

#### Reader::has_expires

```rust
pub fn has_expires(&self) -> bool
```

#### Reader::get_date

```rust
pub fn get_date(self) -> Result<Reader<'a>>
```

#### Reader::has_date

```rust
pub fn has_date(&self) -> bool
```

#### Reader::get_age

```rust
pub fn get_age(self) -> Result<Reader<'a>>
```

#### Reader::has_age

```rust
pub fn has_age(&self) -> bool
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

#### Builder::get_url

```rust
pub fn get_url(self) -> Result<Builder<'a>>
```

#### Builder::set_url

```rust
pub fn set_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_url

```rust
pub fn init_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_url

```rust
pub fn has_url(&self) -> bool
```

#### Builder::get_stored

```rust
pub fn get_stored(self) -> bool
```

#### Builder::set_stored

```rust
pub fn set_stored(&mut self, value: bool)
```

#### Builder::get_content_type

```rust
pub fn get_content_type(self) -> Result<Builder<'a>>
```

#### Builder::set_content_type

```rust
pub fn set_content_type(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_content_type

```rust
pub fn init_content_type(self, size: u32) -> Builder<'a>
```

#### Builder::has_content_type

```rust
pub fn has_content_type(&self) -> bool
```

#### Builder::get_error

```rust
pub fn get_error(self) -> Result<Builder<'a>>
```

#### Builder::set_error

```rust
pub fn set_error(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_error

```rust
pub fn init_error(self, size: u32) -> Builder<'a>
```

#### Builder::has_error

```rust
pub fn has_error(&self) -> bool
```

#### Builder::get_not_modified

```rust
pub fn get_not_modified(self) -> bool
```

#### Builder::set_not_modified

```rust
pub fn set_not_modified(&mut self, value: bool)
```

#### Builder::get_etag

```rust
pub fn get_etag(self) -> Result<Builder<'a>>
```

#### Builder::set_etag

```rust
pub fn set_etag(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_etag

```rust
pub fn init_etag(self, size: u32) -> Builder<'a>
```

#### Builder::has_etag

```rust
pub fn has_etag(&self) -> bool
```

#### Builder::get_last_modified

```rust
pub fn get_last_modified(self) -> Result<Builder<'a>>
```

#### Builder::set_last_modified

```rust
pub fn set_last_modified(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_last_modified

```rust
pub fn init_last_modified(self, size: u32) -> Builder<'a>
```

#### Builder::has_last_modified

```rust
pub fn has_last_modified(&self) -> bool
```

#### Builder::get_cache_control

```rust
pub fn get_cache_control(self) -> Result<Builder<'a>>
```

#### Builder::set_cache_control

```rust
pub fn set_cache_control(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cache_control

```rust
pub fn init_cache_control(self, size: u32) -> Builder<'a>
```

#### Builder::has_cache_control

```rust
pub fn has_cache_control(&self) -> bool
```

#### Builder::get_expires

```rust
pub fn get_expires(self) -> Result<Builder<'a>>
```

#### Builder::set_expires

```rust
pub fn set_expires(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_expires

```rust
pub fn init_expires(self, size: u32) -> Builder<'a>
```

#### Builder::has_expires

```rust
pub fn has_expires(&self) -> bool
```

#### Builder::get_date

```rust
pub fn get_date(self) -> Result<Builder<'a>>
```

#### Builder::set_date

```rust
pub fn set_date(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_date

```rust
pub fn init_date(self, size: u32) -> Builder<'a>
```

#### Builder::has_date

```rust
pub fn has_date(&self) -> bool
```

#### Builder::get_age

```rust
pub fn get_age(self) -> Result<Builder<'a>>
```

#### Builder::set_age

```rust
pub fn set_age(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_age

```rust
pub fn init_age(self, size: u32) -> Builder<'a>
```

#### Builder::has_age

```rust
pub fn has_age(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
