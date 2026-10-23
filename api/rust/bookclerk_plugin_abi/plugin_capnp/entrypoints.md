# bookclerk_plugin_abi::plugin_capnp::entrypoints

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

#### Reader::get_event_consumer

```rust
pub fn get_event_consumer(self) -> Result<Client>
```

#### Reader::has_event_consumer

```rust
pub fn has_event_consumer(&self) -> bool
```

#### Reader::get_job_runner

```rust
pub fn get_job_runner(self) -> Result<Client>
```

#### Reader::has_job_runner

```rust
pub fn has_job_runner(&self) -> bool
```

#### Reader::get_storefront

```rust
pub fn get_storefront(self) -> Result<Client>
```

#### Reader::has_storefront

```rust
pub fn has_storefront(&self) -> bool
```

#### Reader::get_storage

```rust
pub fn get_storage(self) -> Result<Client>
```

#### Reader::has_storage

```rust
pub fn has_storage(&self) -> bool
```

#### Reader::get_database_adapter

```rust
pub fn get_database_adapter(self) -> Result<Client>
```

#### Reader::has_database_adapter

```rust
pub fn has_database_adapter(&self) -> bool
```

#### Reader::get_remote_library

```rust
pub fn get_remote_library(self) -> Result<Client>
```

#### Reader::has_remote_library

```rust
pub fn has_remote_library(&self) -> bool
```

#### Reader::get_cli

```rust
pub fn get_cli(self) -> Result<Client>
```

#### Reader::has_cli

```rust
pub fn has_cli(&self) -> bool
```

#### Reader::get_oidc

```rust
pub fn get_oidc(self) -> Result<Client>
```

#### Reader::has_oidc

```rust
pub fn has_oidc(&self) -> bool
```

#### Reader::get_queue_consumer

```rust
pub fn get_queue_consumer(self) -> Result<Client>
```

#### Reader::has_queue_consumer

```rust
pub fn has_queue_consumer(&self) -> bool
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

#### Builder::get_event_consumer

```rust
pub fn get_event_consumer(self) -> Result<Client>
```

#### Builder::set_event_consumer

```rust
pub fn set_event_consumer(&mut self, value: Client)
```

#### Builder::has_event_consumer

```rust
pub fn has_event_consumer(&self) -> bool
```

#### Builder::get_job_runner

```rust
pub fn get_job_runner(self) -> Result<Client>
```

#### Builder::set_job_runner

```rust
pub fn set_job_runner(&mut self, value: Client)
```

#### Builder::has_job_runner

```rust
pub fn has_job_runner(&self) -> bool
```

#### Builder::get_storefront

```rust
pub fn get_storefront(self) -> Result<Client>
```

#### Builder::set_storefront

```rust
pub fn set_storefront(&mut self, value: Client)
```

#### Builder::has_storefront

```rust
pub fn has_storefront(&self) -> bool
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

#### Builder::get_database_adapter

```rust
pub fn get_database_adapter(self) -> Result<Client>
```

#### Builder::set_database_adapter

```rust
pub fn set_database_adapter(&mut self, value: Client)
```

#### Builder::has_database_adapter

```rust
pub fn has_database_adapter(&self) -> bool
```

#### Builder::get_remote_library

```rust
pub fn get_remote_library(self) -> Result<Client>
```

#### Builder::set_remote_library

```rust
pub fn set_remote_library(&mut self, value: Client)
```

#### Builder::has_remote_library

```rust
pub fn has_remote_library(&self) -> bool
```

#### Builder::get_cli

```rust
pub fn get_cli(self) -> Result<Client>
```

#### Builder::set_cli

```rust
pub fn set_cli(&mut self, value: Client)
```

#### Builder::has_cli

```rust
pub fn has_cli(&self) -> bool
```

#### Builder::get_oidc

```rust
pub fn get_oidc(self) -> Result<Client>
```

#### Builder::set_oidc

```rust
pub fn set_oidc(&mut self, value: Client)
```

#### Builder::has_oidc

```rust
pub fn has_oidc(&self) -> bool
```

#### Builder::get_queue_consumer

```rust
pub fn get_queue_consumer(self) -> Result<Client>
```

#### Builder::set_queue_consumer

```rust
pub fn set_queue_consumer(&mut self, value: Client)
```

#### Builder::has_queue_consumer

```rust
pub fn has_queue_consumer(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_event_consumer

```rust
pub fn get_event_consumer(&self) -> Client
```

#### Pipeline::get_job_runner

```rust
pub fn get_job_runner(&self) -> Client
```

#### Pipeline::get_storefront

```rust
pub fn get_storefront(&self) -> Client
```

#### Pipeline::get_storage

```rust
pub fn get_storage(&self) -> Client
```

#### Pipeline::get_database_adapter

```rust
pub fn get_database_adapter(&self) -> Client
```

#### Pipeline::get_remote_library

```rust
pub fn get_remote_library(&self) -> Client
```

#### Pipeline::get_cli

```rust
pub fn get_cli(&self) -> Client
```

#### Pipeline::get_oidc

```rust
pub fn get_oidc(&self) -> Client
```

#### Pipeline::get_queue_consumer

```rust
pub fn get_queue_consumer(&self) -> Client
```

Implements: `FromTypelessPipeline`
