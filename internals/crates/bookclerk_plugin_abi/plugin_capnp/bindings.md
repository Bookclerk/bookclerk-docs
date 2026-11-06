# bookclerk_plugin_abi::plugin_capnp::bindings

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

#### Reader::get_config

```rust
pub fn get_config(self) -> Result<Reader<'a>>
```

#### Reader::has_config

```rust
pub fn has_config(&self) -> bool
```

#### Reader::get_secrets

```rust
pub fn get_secrets(self) -> Result<Reader<'a>>
```

#### Reader::has_secrets

```rust
pub fn has_secrets(&self) -> bool
```

#### Reader::get_adapter

```rust
pub fn get_adapter(self) -> Result<Reader<'a>>
```

#### Reader::has_adapter

```rust
pub fn has_adapter(&self) -> bool
```

#### Reader::get_events

```rust
pub fn get_events(self) -> Result<Client>
```

#### Reader::has_events

```rust
pub fn has_events(&self) -> bool
```

#### Reader::get_databases

```rust
pub fn get_databases(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_databases

```rust
pub fn has_databases(&self) -> bool
```

#### Reader::get_cancel

```rust
pub fn get_cancel(self) -> Result<Client>
```

#### Reader::has_cancel

```rust
pub fn has_cancel(&self) -> bool
```

#### Reader::get_storage

```rust
pub fn get_storage(self) -> Result<Client>
```

#### Reader::has_storage

```rust
pub fn has_storage(&self) -> bool
```

#### Reader::get_queues

```rust
pub fn get_queues(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_queues

```rust
pub fn has_queues(&self) -> bool
```

#### Reader::get_kv

```rust
pub fn get_kv(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_kv

```rust
pub fn has_kv(&self) -> bool
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

#### Builder::get_config

```rust
pub fn get_config(self) -> Result<Builder<'a>>
```

#### Builder::set_config

```rust
pub fn set_config(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_config

```rust
pub fn init_config(self) -> Builder<'a>
```

#### Builder::has_config

```rust
pub fn has_config(&self) -> bool
```

#### Builder::get_secrets

```rust
pub fn get_secrets(self) -> Result<Builder<'a>>
```

#### Builder::set_secrets

```rust
pub fn set_secrets(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_secrets

```rust
pub fn init_secrets(self) -> Builder<'a>
```

#### Builder::has_secrets

```rust
pub fn has_secrets(&self) -> bool
```

#### Builder::get_adapter

```rust
pub fn get_adapter(self) -> Result<Builder<'a>>
```

#### Builder::set_adapter

```rust
pub fn set_adapter(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_adapter

```rust
pub fn init_adapter(self) -> Builder<'a>
```

#### Builder::has_adapter

```rust
pub fn has_adapter(&self) -> bool
```

#### Builder::get_events

```rust
pub fn get_events(self) -> Result<Client>
```

#### Builder::set_events

```rust
pub fn set_events(&mut self, value: Client)
```

#### Builder::has_events

```rust
pub fn has_events(&self) -> bool
```

#### Builder::get_databases

```rust
pub fn get_databases(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_databases

```rust
pub fn set_databases(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_databases

```rust
pub fn init_databases(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_databases

```rust
pub fn has_databases(&self) -> bool
```

#### Builder::get_cancel

```rust
pub fn get_cancel(self) -> Result<Client>
```

#### Builder::set_cancel

```rust
pub fn set_cancel(&mut self, value: Client)
```

#### Builder::has_cancel

```rust
pub fn has_cancel(&self) -> bool
```

#### Builder::get_storage

```rust
pub fn get_storage(self) -> Result<Client>
```

#### Builder::set_storage

```rust
pub fn set_storage(&mut self, value: Client)
```

#### Builder::has_storage

```rust
pub fn has_storage(&self) -> bool
```

#### Builder::get_queues

```rust
pub fn get_queues(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_queues

```rust
pub fn set_queues(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_queues

```rust
pub fn init_queues(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_queues

```rust
pub fn has_queues(&self) -> bool
```

#### Builder::get_kv

```rust
pub fn get_kv(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_kv

```rust
pub fn set_kv(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_kv

```rust
pub fn init_kv(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_kv

```rust
pub fn has_kv(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_config

```rust
pub fn get_config(&self) -> Pipeline
```

#### Pipeline::get_secrets

```rust
pub fn get_secrets(&self) -> Pipeline
```

#### Pipeline::get_adapter

```rust
pub fn get_adapter(&self) -> Pipeline
```

#### Pipeline::get_events

```rust
pub fn get_events(&self) -> Client
```

#### Pipeline::get_cancel

```rust
pub fn get_cancel(&self) -> Client
```

#### Pipeline::get_storage

```rust
pub fn get_storage(&self) -> Client
```

Implements: `FromTypelessPipeline`
