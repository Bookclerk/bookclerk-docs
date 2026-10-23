# bookclerk_plugin_abi::plugin_capnp::oidc_client_template

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

#### Reader::get_client_id

```rust
pub fn get_client_id(self) -> Result<Reader<'a>>
```

#### Reader::has_client_id

```rust
pub fn has_client_id(&self) -> bool
```

#### Reader::get_display_name

```rust
pub fn get_display_name(self) -> Result<Reader<'a>>
```

#### Reader::has_display_name

```rust
pub fn has_display_name(&self) -> bool
```

#### Reader::get_callback_path

```rust
pub fn get_callback_path(self) -> Result<Reader<'a>>
```

#### Reader::has_callback_path

```rust
pub fn has_callback_path(&self) -> bool
```

#### Reader::get_public_client

```rust
pub fn get_public_client(self) -> bool
```

#### Reader::get_default_scopes

```rust
pub fn get_default_scopes(self) -> Result<Reader<'a>>
```

#### Reader::has_default_scopes

```rust
pub fn has_default_scopes(&self) -> bool
```

#### Reader::get_issue_refresh_token

```rust
pub fn get_issue_refresh_token(self) -> bool
```

#### Reader::get_origin_config_key

```rust
pub fn get_origin_config_key(self) -> Result<Reader<'a>>
```

#### Reader::has_origin_config_key

```rust
pub fn has_origin_config_key(&self) -> bool
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

#### Builder::get_client_id

```rust
pub fn get_client_id(self) -> Result<Builder<'a>>
```

#### Builder::set_client_id

```rust
pub fn set_client_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_client_id

```rust
pub fn init_client_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_client_id

```rust
pub fn has_client_id(&self) -> bool
```

#### Builder::get_display_name

```rust
pub fn get_display_name(self) -> Result<Builder<'a>>
```

#### Builder::set_display_name

```rust
pub fn set_display_name(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_display_name

```rust
pub fn init_display_name(self, size: u32) -> Builder<'a>
```

#### Builder::has_display_name

```rust
pub fn has_display_name(&self) -> bool
```

#### Builder::get_callback_path

```rust
pub fn get_callback_path(self) -> Result<Builder<'a>>
```

#### Builder::set_callback_path

```rust
pub fn set_callback_path(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_callback_path

```rust
pub fn init_callback_path(self, size: u32) -> Builder<'a>
```

#### Builder::has_callback_path

```rust
pub fn has_callback_path(&self) -> bool
```

#### Builder::get_public_client

```rust
pub fn get_public_client(self) -> bool
```

#### Builder::set_public_client

```rust
pub fn set_public_client(&mut self, value: bool)
```

#### Builder::get_default_scopes

```rust
pub fn get_default_scopes(self) -> Result<Builder<'a>>
```

#### Builder::set_default_scopes

```rust
pub fn set_default_scopes(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_default_scopes

```rust
pub fn init_default_scopes(self, size: u32) -> Builder<'a>
```

#### Builder::has_default_scopes

```rust
pub fn has_default_scopes(&self) -> bool
```

#### Builder::get_issue_refresh_token

```rust
pub fn get_issue_refresh_token(self) -> bool
```

#### Builder::set_issue_refresh_token

```rust
pub fn set_issue_refresh_token(&mut self, value: bool)
```

#### Builder::get_origin_config_key

```rust
pub fn get_origin_config_key(self) -> Result<Builder<'a>>
```

#### Builder::set_origin_config_key

```rust
pub fn set_origin_config_key(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_origin_config_key

```rust
pub fn init_origin_config_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_origin_config_key

```rust
pub fn has_origin_config_key(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
