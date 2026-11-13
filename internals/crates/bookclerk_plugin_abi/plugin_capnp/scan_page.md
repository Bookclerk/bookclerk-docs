# bookclerk_plugin_abi::plugin_capnp::scan_page

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

#### Reader::get_books

```rust
pub fn get_books(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_books

```rust
pub fn has_books(&self) -> bool
```

#### Reader::get_removed

```rust
pub fn get_removed(self) -> Result<Reader<'a>>
```

#### Reader::has_removed

```rust
pub fn has_removed(&self) -> bool
```

#### Reader::get_next_cursor

```rust
pub fn get_next_cursor(self) -> Result<Reader<'a>>
```

#### Reader::has_next_cursor

```rust
pub fn has_next_cursor(&self) -> bool
```

#### Reader::get_sync_token

```rust
pub fn get_sync_token(self) -> Result<Reader<'a>>
```

#### Reader::has_sync_token

```rust
pub fn has_sync_token(&self) -> bool
```

#### Reader::get_complete

```rust
pub fn get_complete(self) -> bool
```

#### Reader::get_store_pages

```rust
pub fn get_store_pages(self) -> u32
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

#### Builder::get_books

```rust
pub fn get_books(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_books

```rust
pub fn set_books(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_books

```rust
pub fn init_books(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_books

```rust
pub fn has_books(&self) -> bool
```

#### Builder::get_removed

```rust
pub fn get_removed(self) -> Result<Builder<'a>>
```

#### Builder::set_removed

```rust
pub fn set_removed(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_removed

```rust
pub fn init_removed(self, size: u32) -> Builder<'a>
```

#### Builder::has_removed

```rust
pub fn has_removed(&self) -> bool
```

#### Builder::get_next_cursor

```rust
pub fn get_next_cursor(self) -> Result<Builder<'a>>
```

#### Builder::set_next_cursor

```rust
pub fn set_next_cursor(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_next_cursor

```rust
pub fn init_next_cursor(self, size: u32) -> Builder<'a>
```

#### Builder::has_next_cursor

```rust
pub fn has_next_cursor(&self) -> bool
```

#### Builder::get_sync_token

```rust
pub fn get_sync_token(self) -> Result<Builder<'a>>
```

#### Builder::set_sync_token

```rust
pub fn set_sync_token(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_sync_token

```rust
pub fn init_sync_token(self, size: u32) -> Builder<'a>
```

#### Builder::has_sync_token

```rust
pub fn has_sync_token(&self) -> bool
```

#### Builder::get_complete

```rust
pub fn get_complete(self) -> bool
```

#### Builder::set_complete

```rust
pub fn set_complete(&mut self, value: bool)
```

#### Builder::get_store_pages

```rust
pub fn get_store_pages(self) -> u32
```

#### Builder::set_store_pages

```rust
pub fn set_store_pages(&mut self, value: u32)
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
