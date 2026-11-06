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

#### Reader::get_accounts

```rust
pub fn get_accounts(self) -> Result<Reader<'a>>
```

#### Reader::has_accounts

```rust
pub fn has_accounts(&self) -> bool
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

#### Reader::get_credentials

```rust
pub fn get_credentials(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_credentials

```rust
pub fn has_credentials(&self) -> bool
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

#### Builder::get_accounts

```rust
pub fn get_accounts(self) -> Result<Builder<'a>>
```

#### Builder::set_accounts

```rust
pub fn set_accounts(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_accounts

```rust
pub fn init_accounts(self, size: u32) -> Builder<'a>
```

#### Builder::has_accounts

```rust
pub fn has_accounts(&self) -> bool
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

#### Builder::get_credentials

```rust
pub fn get_credentials(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_credentials

```rust
pub fn set_credentials(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_credentials

```rust
pub fn init_credentials(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_credentials

```rust
pub fn has_credentials(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
