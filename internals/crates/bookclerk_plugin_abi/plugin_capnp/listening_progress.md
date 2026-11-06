# bookclerk_plugin_abi::plugin_capnp::listening_progress

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

#### Reader::get_external_user_id

```rust
pub fn get_external_user_id(self) -> Result<Reader<'a>>
```

#### Reader::has_external_user_id

```rust
pub fn has_external_user_id(&self) -> bool
```

#### Reader::get_external_item_id

```rust
pub fn get_external_item_id(self) -> Result<Reader<'a>>
```

#### Reader::has_external_item_id

```rust
pub fn has_external_item_id(&self) -> bool
```

#### Reader::get_identity_id

```rust
pub fn get_identity_id(self) -> i64
```

#### Reader::get_title

```rust
pub fn get_title(self) -> Result<Reader<'a>>
```

#### Reader::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Reader::get_authors

```rust
pub fn get_authors(self) -> Result<Reader<'a>>
```

#### Reader::has_authors

```rust
pub fn has_authors(&self) -> bool
```

#### Reader::get_asin

```rust
pub fn get_asin(self) -> Result<Reader<'a>>
```

#### Reader::has_asin

```rust
pub fn has_asin(&self) -> bool
```

#### Reader::get_isbn

```rust
pub fn get_isbn(self) -> Result<Reader<'a>>
```

#### Reader::has_isbn

```rust
pub fn has_isbn(&self) -> bool
```

#### Reader::get_progress

```rust
pub fn get_progress(self) -> f64
```

#### Reader::get_current_time_seconds

```rust
pub fn get_current_time_seconds(self) -> f64
```

#### Reader::get_duration_seconds

```rust
pub fn get_duration_seconds(self) -> f64
```

#### Reader::get_is_finished

```rust
pub fn get_is_finished(self) -> bool
```

#### Reader::get_last_listened_at_unix_ms

```rust
pub fn get_last_listened_at_unix_ms(self) -> u64
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

#### Builder::get_external_user_id

```rust
pub fn get_external_user_id(self) -> Result<Builder<'a>>
```

#### Builder::set_external_user_id

```rust
pub fn set_external_user_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_external_user_id

```rust
pub fn init_external_user_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_external_user_id

```rust
pub fn has_external_user_id(&self) -> bool
```

#### Builder::get_external_item_id

```rust
pub fn get_external_item_id(self) -> Result<Builder<'a>>
```

#### Builder::set_external_item_id

```rust
pub fn set_external_item_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_external_item_id

```rust
pub fn init_external_item_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_external_item_id

```rust
pub fn has_external_item_id(&self) -> bool
```

#### Builder::get_identity_id

```rust
pub fn get_identity_id(self) -> i64
```

#### Builder::set_identity_id

```rust
pub fn set_identity_id(&mut self, value: i64)
```

#### Builder::get_title

```rust
pub fn get_title(self) -> Result<Builder<'a>>
```

#### Builder::set_title

```rust
pub fn set_title(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_title

```rust
pub fn init_title(self, size: u32) -> Builder<'a>
```

#### Builder::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Builder::get_authors

```rust
pub fn get_authors(self) -> Result<Builder<'a>>
```

#### Builder::set_authors

```rust
pub fn set_authors(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_authors

```rust
pub fn init_authors(self, size: u32) -> Builder<'a>
```

#### Builder::has_authors

```rust
pub fn has_authors(&self) -> bool
```

#### Builder::get_asin

```rust
pub fn get_asin(self) -> Result<Builder<'a>>
```

#### Builder::set_asin

```rust
pub fn set_asin(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_asin

```rust
pub fn init_asin(self, size: u32) -> Builder<'a>
```

#### Builder::has_asin

```rust
pub fn has_asin(&self) -> bool
```

#### Builder::get_isbn

```rust
pub fn get_isbn(self) -> Result<Builder<'a>>
```

#### Builder::set_isbn

```rust
pub fn set_isbn(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_isbn

```rust
pub fn init_isbn(self, size: u32) -> Builder<'a>
```

#### Builder::has_isbn

```rust
pub fn has_isbn(&self) -> bool
```

#### Builder::get_progress

```rust
pub fn get_progress(self) -> f64
```

#### Builder::set_progress

```rust
pub fn set_progress(&mut self, value: f64)
```

#### Builder::get_current_time_seconds

```rust
pub fn get_current_time_seconds(self) -> f64
```

#### Builder::set_current_time_seconds

```rust
pub fn set_current_time_seconds(&mut self, value: f64)
```

#### Builder::get_duration_seconds

```rust
pub fn get_duration_seconds(self) -> f64
```

#### Builder::set_duration_seconds

```rust
pub fn set_duration_seconds(&mut self, value: f64)
```

#### Builder::get_is_finished

```rust
pub fn get_is_finished(self) -> bool
```

#### Builder::set_is_finished

```rust
pub fn set_is_finished(&mut self, value: bool)
```

#### Builder::get_last_listened_at_unix_ms

```rust
pub fn get_last_listened_at_unix_ms(self) -> u64
```

#### Builder::set_last_listened_at_unix_ms

```rust
pub fn set_last_listened_at_unix_ms(&mut self, value: u64)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
