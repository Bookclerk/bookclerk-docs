# bookclerk_plugin_abi::plugin_capnp::login_params

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

#### Reader::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Reader<'a>>
```

#### Reader::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
```

#### Reader::get_label

```rust
pub fn get_label(self) -> Result<Reader<'a>>
```

#### Reader::has_label

```rust
pub fn has_label(&self) -> bool
```

#### Reader::get_email

```rust
pub fn get_email(self) -> Result<Reader<'a>>
```

#### Reader::has_email

```rust
pub fn has_email(&self) -> bool
```

#### Reader::get_password

```rust
pub fn get_password(self) -> Result<Reader<'a>>
```

#### Reader::has_password

```rust
pub fn has_password(&self) -> bool
```

#### Reader::get_force

```rust
pub fn get_force(self) -> bool
```

#### Reader::get_callback_bind

```rust
pub fn get_callback_bind(self) -> Result<Reader<'a>>
```

#### Reader::has_callback_bind

```rust
pub fn has_callback_bind(&self) -> bool
```

#### Reader::get_callback_ipc

```rust
pub fn get_callback_ipc(self) -> Result<Reader<'a>>
```

#### Reader::has_callback_ipc

```rust
pub fn has_callback_ipc(&self) -> bool
```

#### Reader::get_callback_public_base

```rust
pub fn get_callback_public_base(self) -> Result<Reader<'a>>
```

#### Reader::has_callback_public_base

```rust
pub fn has_callback_public_base(&self) -> bool
```

#### Reader::get_external

```rust
pub fn get_external(self) -> bool
```

#### Reader::get_response_url

```rust
pub fn get_response_url(self) -> Result<Reader<'a>>
```

#### Reader::has_response_url

```rust
pub fn has_response_url(&self) -> bool
```

#### Reader::get_show_qr

```rust
pub fn get_show_qr(self) -> bool
```

#### Reader::get_timeout_secs

```rust
pub fn get_timeout_secs(self) -> u64
```

#### Reader::get_extra

```rust
pub fn get_extra(self) -> Result<Reader<'a>>
```

#### Reader::has_extra

```rust
pub fn has_extra(&self) -> bool
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

#### Builder::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Builder<'a>>
```

#### Builder::set_marketplace

```rust
pub fn set_marketplace(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_marketplace

```rust
pub fn init_marketplace(self, size: u32) -> Builder<'a>
```

#### Builder::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
```

#### Builder::get_label

```rust
pub fn get_label(self) -> Result<Builder<'a>>
```

#### Builder::set_label

```rust
pub fn set_label(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_label

```rust
pub fn init_label(self, size: u32) -> Builder<'a>
```

#### Builder::has_label

```rust
pub fn has_label(&self) -> bool
```

#### Builder::get_email

```rust
pub fn get_email(self) -> Result<Builder<'a>>
```

#### Builder::set_email

```rust
pub fn set_email(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_email

```rust
pub fn init_email(self, size: u32) -> Builder<'a>
```

#### Builder::has_email

```rust
pub fn has_email(&self) -> bool
```

#### Builder::get_password

```rust
pub fn get_password(self) -> Result<Builder<'a>>
```

#### Builder::set_password

```rust
pub fn set_password(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_password

```rust
pub fn init_password(self, size: u32) -> Builder<'a>
```

#### Builder::has_password

```rust
pub fn has_password(&self) -> bool
```

#### Builder::get_force

```rust
pub fn get_force(self) -> bool
```

#### Builder::set_force

```rust
pub fn set_force(&mut self, value: bool)
```

#### Builder::get_callback_bind

```rust
pub fn get_callback_bind(self) -> Result<Builder<'a>>
```

#### Builder::set_callback_bind

```rust
pub fn set_callback_bind(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_callback_bind

```rust
pub fn init_callback_bind(self, size: u32) -> Builder<'a>
```

#### Builder::has_callback_bind

```rust
pub fn has_callback_bind(&self) -> bool
```

#### Builder::get_callback_ipc

```rust
pub fn get_callback_ipc(self) -> Result<Builder<'a>>
```

#### Builder::set_callback_ipc

```rust
pub fn set_callback_ipc(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_callback_ipc

```rust
pub fn init_callback_ipc(self, size: u32) -> Builder<'a>
```

#### Builder::has_callback_ipc

```rust
pub fn has_callback_ipc(&self) -> bool
```

#### Builder::get_callback_public_base

```rust
pub fn get_callback_public_base(self) -> Result<Builder<'a>>
```

#### Builder::set_callback_public_base

```rust
pub fn set_callback_public_base(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_callback_public_base

```rust
pub fn init_callback_public_base(self, size: u32) -> Builder<'a>
```

#### Builder::has_callback_public_base

```rust
pub fn has_callback_public_base(&self) -> bool
```

#### Builder::get_external

```rust
pub fn get_external(self) -> bool
```

#### Builder::set_external

```rust
pub fn set_external(&mut self, value: bool)
```

#### Builder::get_response_url

```rust
pub fn get_response_url(self) -> Result<Builder<'a>>
```

#### Builder::set_response_url

```rust
pub fn set_response_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_response_url

```rust
pub fn init_response_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_response_url

```rust
pub fn has_response_url(&self) -> bool
```

#### Builder::get_show_qr

```rust
pub fn get_show_qr(self) -> bool
```

#### Builder::set_show_qr

```rust
pub fn set_show_qr(&mut self, value: bool)
```

#### Builder::get_timeout_secs

```rust
pub fn get_timeout_secs(self) -> u64
```

#### Builder::set_timeout_secs

```rust
pub fn set_timeout_secs(&mut self, value: u64)
```

#### Builder::get_extra

```rust
pub fn get_extra(self) -> Result<Builder<'a>>
```

#### Builder::set_extra

```rust
pub fn set_extra(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_extra

```rust
pub fn init_extra(self) -> Builder<'a>
```

#### Builder::has_extra

```rust
pub fn has_extra(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_extra

```rust
pub fn get_extra(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
