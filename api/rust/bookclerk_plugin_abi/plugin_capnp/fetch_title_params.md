# bookclerk_plugin_abi::plugin_capnp::fetch_title_params

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

#### Reader::get_title_id

```rust
pub fn get_title_id(self) -> Result<Reader<'a>>
```

#### Reader::has_title_id

```rust
pub fn has_title_id(&self) -> bool
```

#### Reader::get_cache_dir

```rust
pub fn get_cache_dir(self) -> Result<Reader<'a>>
```

#### Reader::has_cache_dir

```rust
pub fn has_cache_dir(&self) -> bool
```

#### Reader::get_credentials

```rust
pub fn get_credentials(self) -> Result<Reader<'a>>
```

#### Reader::has_credentials

```rust
pub fn has_credentials(&self) -> bool
```

#### Reader::get_source_config

```rust
pub fn get_source_config(self) -> Result<Reader<'a>>
```

#### Reader::has_source_config

```rust
pub fn has_source_config(&self) -> bool
```

#### Reader::get_fetch

```rust
pub fn get_fetch(self) -> Result<Reader<'a>>
```

#### Reader::has_fetch

```rust
pub fn has_fetch(&self) -> bool
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

#### Builder::get_title_id

```rust
pub fn get_title_id(self) -> Result<Builder<'a>>
```

#### Builder::set_title_id

```rust
pub fn set_title_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_title_id

```rust
pub fn init_title_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_title_id

```rust
pub fn has_title_id(&self) -> bool
```

#### Builder::get_cache_dir

```rust
pub fn get_cache_dir(self) -> Result<Builder<'a>>
```

#### Builder::set_cache_dir

```rust
pub fn set_cache_dir(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cache_dir

```rust
pub fn init_cache_dir(self, size: u32) -> Builder<'a>
```

#### Builder::has_cache_dir

```rust
pub fn has_cache_dir(&self) -> bool
```

#### Builder::get_credentials

```rust
pub fn get_credentials(self) -> Result<Builder<'a>>
```

#### Builder::set_credentials

```rust
pub fn set_credentials(&mut self, value: Reader<'_>)
```

#### Builder::init_credentials

```rust
pub fn init_credentials(self, size: u32) -> Builder<'a>
```

#### Builder::has_credentials

```rust
pub fn has_credentials(&self) -> bool
```

#### Builder::get_source_config

```rust
pub fn get_source_config(self) -> Result<Builder<'a>>
```

#### Builder::set_source_config

```rust
pub fn set_source_config(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_source_config

```rust
pub fn init_source_config(self) -> Builder<'a>
```

#### Builder::has_source_config

```rust
pub fn has_source_config(&self) -> bool
```

#### Builder::get_fetch

```rust
pub fn get_fetch(self) -> Result<Builder<'a>>
```

#### Builder::set_fetch

```rust
pub fn set_fetch(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_fetch

```rust
pub fn init_fetch(self) -> Builder<'a>
```

#### Builder::has_fetch

```rust
pub fn has_fetch(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_source_config

```rust
pub fn get_source_config(&self) -> Pipeline
```

#### Pipeline::get_fetch

```rust
pub fn get_fetch(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
