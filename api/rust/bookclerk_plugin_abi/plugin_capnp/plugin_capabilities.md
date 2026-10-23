# bookclerk_plugin_abi::plugin_capnp::plugin_capabilities

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

#### Reader::get_entrypoints

```rust
pub fn get_entrypoints(self) -> Result<Reader<'a, Entrypoint>>
```

#### Reader::has_entrypoints

```rust
pub fn has_entrypoints(&self) -> bool
```

#### Reader::get_consumes

```rust
pub fn get_consumes(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_consumes

```rust
pub fn has_consumes(&self) -> bool
```

#### Reader::get_produces

```rust
pub fn get_produces(self) -> Result<Reader<'a>>
```

#### Reader::has_produces

```rust
pub fn has_produces(&self) -> bool
```

#### Reader::get_jobs

```rust
pub fn get_jobs(self) -> Result<Reader<'a>>
```

#### Reader::has_jobs

```rust
pub fn has_jobs(&self) -> bool
```

#### Reader::get_databases

```rust
pub fn get_databases(self) -> Result<Reader<'a>>
```

#### Reader::has_databases

```rust
pub fn has_databases(&self) -> bool
```

#### Reader::get_bindings

```rust
pub fn get_bindings(self) -> Result<Reader<'a>>
```

#### Reader::has_bindings

```rust
pub fn has_bindings(&self) -> bool
```

#### Reader::get_queue_consumers

```rust
pub fn get_queue_consumers(self) -> Result<Reader<'a>>
```

#### Reader::has_queue_consumers

```rust
pub fn has_queue_consumers(&self) -> bool
```

#### Reader::get_queue_producers

```rust
pub fn get_queue_producers(self) -> Result<Reader<'a>>
```

#### Reader::has_queue_producers

```rust
pub fn has_queue_producers(&self) -> bool
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

#### Builder::get_entrypoints

```rust
pub fn get_entrypoints(self) -> Result<Builder<'a, Entrypoint>>
```

#### Builder::set_entrypoints

```rust
pub fn set_entrypoints(
    &mut self,
    value: impl SetterInput<Owned<Entrypoint>>,
) -> Result<()>
```

#### Builder::init_entrypoints

```rust
pub fn init_entrypoints(self, size: u32) -> Builder<'a, Entrypoint>
```

#### Builder::has_entrypoints

```rust
pub fn has_entrypoints(&self) -> bool
```

#### Builder::get_consumes

```rust
pub fn get_consumes(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_consumes

```rust
pub fn set_consumes(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_consumes

```rust
pub fn init_consumes(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_consumes

```rust
pub fn has_consumes(&self) -> bool
```

#### Builder::get_produces

```rust
pub fn get_produces(self) -> Result<Builder<'a>>
```

#### Builder::set_produces

```rust
pub fn set_produces(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_produces

```rust
pub fn init_produces(self, size: u32) -> Builder<'a>
```

#### Builder::has_produces

```rust
pub fn has_produces(&self) -> bool
```

#### Builder::get_jobs

```rust
pub fn get_jobs(self) -> Result<Builder<'a>>
```

#### Builder::set_jobs

```rust
pub fn set_jobs(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_jobs

```rust
pub fn init_jobs(self, size: u32) -> Builder<'a>
```

#### Builder::has_jobs

```rust
pub fn has_jobs(&self) -> bool
```

#### Builder::get_databases

```rust
pub fn get_databases(self) -> Result<Builder<'a>>
```

#### Builder::set_databases

```rust
pub fn set_databases(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_databases

```rust
pub fn init_databases(self, size: u32) -> Builder<'a>
```

#### Builder::has_databases

```rust
pub fn has_databases(&self) -> bool
```

#### Builder::get_bindings

```rust
pub fn get_bindings(self) -> Result<Builder<'a>>
```

#### Builder::set_bindings

```rust
pub fn set_bindings(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_bindings

```rust
pub fn init_bindings(self, size: u32) -> Builder<'a>
```

#### Builder::has_bindings

```rust
pub fn has_bindings(&self) -> bool
```

#### Builder::get_queue_consumers

```rust
pub fn get_queue_consumers(self) -> Result<Builder<'a>>
```

#### Builder::set_queue_consumers

```rust
pub fn set_queue_consumers(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_queue_consumers

```rust
pub fn init_queue_consumers(self, size: u32) -> Builder<'a>
```

#### Builder::has_queue_consumers

```rust
pub fn has_queue_consumers(&self) -> bool
```

#### Builder::get_queue_producers

```rust
pub fn get_queue_producers(self) -> Result<Builder<'a>>
```

#### Builder::set_queue_producers

```rust
pub fn set_queue_producers(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_queue_producers

```rust
pub fn init_queue_producers(self, size: u32) -> Builder<'a>
```

#### Builder::has_queue_producers

```rust
pub fn has_queue_producers(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
