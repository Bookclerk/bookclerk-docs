# bookclerk_plugin_abi::plugin_capnp::database_adapter_config

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

#### Reader::get_settings

```rust
pub fn get_settings(self) -> Result<Reader<'a>>
```

#### Reader::has_settings

```rust
pub fn has_settings(&self) -> bool
```

#### Reader::get_binding

```rust
pub fn get_binding(self) -> Result<Reader<'a>>
```

#### Reader::has_binding

```rust
pub fn has_binding(&self) -> bool
```

#### Reader::get_instance_id

```rust
pub fn get_instance_id(self) -> Result<Reader<'a>>
```

#### Reader::has_instance_id

```rust
pub fn has_instance_id(&self) -> bool
```

#### Reader::get_open_existing

```rust
pub fn get_open_existing(self) -> bool
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

#### Builder::get_settings

```rust
pub fn get_settings(self) -> Result<Builder<'a>>
```

#### Builder::set_settings

```rust
pub fn set_settings(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_settings

```rust
pub fn init_settings(self) -> Builder<'a>
```

#### Builder::has_settings

```rust
pub fn has_settings(&self) -> bool
```

#### Builder::get_binding

```rust
pub fn get_binding(self) -> Result<Builder<'a>>
```

#### Builder::set_binding

```rust
pub fn set_binding(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_binding

```rust
pub fn init_binding(self, size: u32) -> Builder<'a>
```

#### Builder::has_binding

```rust
pub fn has_binding(&self) -> bool
```

#### Builder::get_instance_id

```rust
pub fn get_instance_id(self) -> Result<Builder<'a>>
```

#### Builder::set_instance_id

```rust
pub fn set_instance_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_instance_id

```rust
pub fn init_instance_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_instance_id

```rust
pub fn has_instance_id(&self) -> bool
```

#### Builder::get_open_existing

```rust
pub fn get_open_existing(self) -> bool
```

#### Builder::set_open_existing

```rust
pub fn set_open_existing(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_settings

```rust
pub fn get_settings(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
