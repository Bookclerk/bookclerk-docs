# bookclerk_plugin_abi::plugin_capnp::scan_params

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

#### Reader::get_plugin_data_dir

```rust
pub fn get_plugin_data_dir(self) -> Result<Reader<'a>>
```

#### Reader::has_plugin_data_dir

```rust
pub fn has_plugin_data_dir(&self) -> bool
```

#### Reader::get_account_id

```rust
pub fn get_account_id(self) -> Result<Reader<'a>>
```

#### Reader::has_account_id

```rust
pub fn has_account_id(&self) -> bool
```

#### Reader::get_page_size

```rust
pub fn get_page_size(self) -> u32
```

#### Reader::get_import_episodes

```rust
pub fn get_import_episodes(self) -> bool
```

#### Reader::get_import_plus_titles

```rust
pub fn get_import_plus_titles(self) -> bool
```

#### Reader::get_credential

```rust
pub fn get_credential(self) -> Result<Reader<'a>>
```

#### Reader::has_credential

```rust
pub fn has_credential(&self) -> bool
```

#### Reader::get_cursor

```rust
pub fn get_cursor(self) -> Result<Reader<'a>>
```

#### Reader::has_cursor

```rust
pub fn has_cursor(&self) -> bool
```

#### Reader::get_sync_token

```rust
pub fn get_sync_token(self) -> Result<Reader<'a>>
```

#### Reader::has_sync_token

```rust
pub fn has_sync_token(&self) -> bool
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

#### Builder::get_plugin_data_dir

```rust
pub fn get_plugin_data_dir(self) -> Result<Builder<'a>>
```

#### Builder::set_plugin_data_dir

```rust
pub fn set_plugin_data_dir(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_plugin_data_dir

```rust
pub fn init_plugin_data_dir(self, size: u32) -> Builder<'a>
```

#### Builder::has_plugin_data_dir

```rust
pub fn has_plugin_data_dir(&self) -> bool
```

#### Builder::get_account_id

```rust
pub fn get_account_id(self) -> Result<Builder<'a>>
```

#### Builder::set_account_id

```rust
pub fn set_account_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_account_id

```rust
pub fn init_account_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_account_id

```rust
pub fn has_account_id(&self) -> bool
```

#### Builder::get_page_size

```rust
pub fn get_page_size(self) -> u32
```

#### Builder::set_page_size

```rust
pub fn set_page_size(&mut self, value: u32)
```

#### Builder::get_import_episodes

```rust
pub fn get_import_episodes(self) -> bool
```

#### Builder::set_import_episodes

```rust
pub fn set_import_episodes(&mut self, value: bool)
```

#### Builder::get_import_plus_titles

```rust
pub fn get_import_plus_titles(self) -> bool
```

#### Builder::set_import_plus_titles

```rust
pub fn set_import_plus_titles(&mut self, value: bool)
```

#### Builder::get_credential

```rust
pub fn get_credential(self) -> Result<Builder<'a>>
```

#### Builder::set_credential

```rust
pub fn set_credential(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_credential

```rust
pub fn init_credential(self) -> Builder<'a>
```

#### Builder::has_credential

```rust
pub fn has_credential(&self) -> bool
```

#### Builder::get_cursor

```rust
pub fn get_cursor(self) -> Result<Builder<'a>>
```

#### Builder::set_cursor

```rust
pub fn set_cursor(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cursor

```rust
pub fn init_cursor(self, size: u32) -> Builder<'a>
```

#### Builder::has_cursor

```rust
pub fn has_cursor(&self) -> bool
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

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_credential

```rust
pub fn get_credential(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
