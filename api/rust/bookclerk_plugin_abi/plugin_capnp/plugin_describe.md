# bookclerk_plugin_abi::plugin_capnp::plugin_describe

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

#### Reader::get_api_version

```rust
pub fn get_api_version(self) -> u32
```

#### Reader::get_id

```rust
pub fn get_id(self) -> Result<Reader<'a>>
```

#### Reader::has_id

```rust
pub fn has_id(&self) -> bool
```

#### Reader::get_display_name

```rust
pub fn get_display_name(self) -> Result<Reader<'a>>
```

#### Reader::has_display_name

```rust
pub fn has_display_name(&self) -> bool
```

#### Reader::get_rpc_features

```rust
pub fn get_rpc_features(self) -> Result<Reader<'a>>
```

#### Reader::has_rpc_features

```rust
pub fn has_rpc_features(&self) -> bool
```

#### Reader::get_scalar_limits

```rust
pub fn get_scalar_limits(self) -> Result<Reader<'a>>
```

#### Reader::has_scalar_limits

```rust
pub fn has_scalar_limits(&self) -> bool
```

#### Reader::get_capabilities

```rust
pub fn get_capabilities(self) -> Result<Reader<'a>>
```

#### Reader::has_capabilities

```rust
pub fn has_capabilities(&self) -> bool
```

#### Reader::get_portal_auth_mode

```rust
pub fn get_portal_auth_mode(self) -> Result<PortalAuthMode, NotInSchema>
```

#### Reader::get_password_env_var

```rust
pub fn get_password_env_var(self) -> Result<Reader<'a>>
```

#### Reader::has_password_env_var

```rust
pub fn has_password_env_var(&self) -> bool
```

#### Reader::get_aliases

```rust
pub fn get_aliases(self) -> Result<Reader<'a>>
```

#### Reader::has_aliases

```rust
pub fn has_aliases(&self) -> bool
```

#### Reader::get_sort_key

```rust
pub fn get_sort_key(self) -> u32
```

#### Reader::get_brand

```rust
pub fn get_brand(self) -> Result<Reader<'a>>
```

#### Reader::has_brand

```rust
pub fn has_brand(&self) -> bool
```

#### Reader::get_config_options

```rust
pub fn get_config_options(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_config_options

```rust
pub fn has_config_options(&self) -> bool
```

#### Reader::get_cli

```rust
pub fn get_cli(self) -> Result<Reader<'a>>
```

#### Reader::has_cli

```rust
pub fn has_cli(&self) -> bool
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

#### Builder::get_api_version

```rust
pub fn get_api_version(self) -> u32
```

#### Builder::set_api_version

```rust
pub fn set_api_version(&mut self, value: u32)
```

#### Builder::get_id

```rust
pub fn get_id(self) -> Result<Builder<'a>>
```

#### Builder::set_id

```rust
pub fn set_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_id

```rust
pub fn init_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_id

```rust
pub fn has_id(&self) -> bool
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

#### Builder::get_rpc_features

```rust
pub fn get_rpc_features(self) -> Result<Builder<'a>>
```

#### Builder::set_rpc_features

```rust
pub fn set_rpc_features(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_rpc_features

```rust
pub fn init_rpc_features(self, size: u32) -> Builder<'a>
```

#### Builder::has_rpc_features

```rust
pub fn has_rpc_features(&self) -> bool
```

#### Builder::get_scalar_limits

```rust
pub fn get_scalar_limits(self) -> Result<Builder<'a>>
```

#### Builder::set_scalar_limits

```rust
pub fn set_scalar_limits(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_scalar_limits

```rust
pub fn init_scalar_limits(self) -> Builder<'a>
```

#### Builder::has_scalar_limits

```rust
pub fn has_scalar_limits(&self) -> bool
```

#### Builder::get_capabilities

```rust
pub fn get_capabilities(self) -> Result<Builder<'a>>
```

#### Builder::set_capabilities

```rust
pub fn set_capabilities(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_capabilities

```rust
pub fn init_capabilities(self) -> Builder<'a>
```

#### Builder::has_capabilities

```rust
pub fn has_capabilities(&self) -> bool
```

#### Builder::get_portal_auth_mode

```rust
pub fn get_portal_auth_mode(self) -> Result<PortalAuthMode, NotInSchema>
```

#### Builder::set_portal_auth_mode

```rust
pub fn set_portal_auth_mode(&mut self, value: PortalAuthMode)
```

#### Builder::get_password_env_var

```rust
pub fn get_password_env_var(self) -> Result<Builder<'a>>
```

#### Builder::set_password_env_var

```rust
pub fn set_password_env_var(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_password_env_var

```rust
pub fn init_password_env_var(self, size: u32) -> Builder<'a>
```

#### Builder::has_password_env_var

```rust
pub fn has_password_env_var(&self) -> bool
```

#### Builder::get_aliases

```rust
pub fn get_aliases(self) -> Result<Builder<'a>>
```

#### Builder::set_aliases

```rust
pub fn set_aliases(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_aliases

```rust
pub fn init_aliases(self, size: u32) -> Builder<'a>
```

#### Builder::has_aliases

```rust
pub fn has_aliases(&self) -> bool
```

#### Builder::get_sort_key

```rust
pub fn get_sort_key(self) -> u32
```

#### Builder::set_sort_key

```rust
pub fn set_sort_key(&mut self, value: u32)
```

#### Builder::get_brand

```rust
pub fn get_brand(self) -> Result<Builder<'a>>
```

#### Builder::set_brand

```rust
pub fn set_brand(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_brand

```rust
pub fn init_brand(self) -> Builder<'a>
```

#### Builder::has_brand

```rust
pub fn has_brand(&self) -> bool
```

#### Builder::get_config_options

```rust
pub fn get_config_options(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_config_options

```rust
pub fn set_config_options(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_config_options

```rust
pub fn init_config_options(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_config_options

```rust
pub fn has_config_options(&self) -> bool
```

#### Builder::get_cli

```rust
pub fn get_cli(self) -> Result<Builder<'a>>
```

#### Builder::set_cli

```rust
pub fn set_cli(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_cli

```rust
pub fn init_cli(self) -> Builder<'a>
```

#### Builder::has_cli

```rust
pub fn has_cli(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_scalar_limits

```rust
pub fn get_scalar_limits(&self) -> Pipeline
```

#### Pipeline::get_capabilities

```rust
pub fn get_capabilities(&self) -> Pipeline
```

#### Pipeline::get_brand

```rust
pub fn get_brand(&self) -> Pipeline
```

#### Pipeline::get_cli

```rust
pub fn get_cli(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
