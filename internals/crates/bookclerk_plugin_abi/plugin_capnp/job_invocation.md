# bookclerk_plugin_abi::plugin_capnp::job_invocation

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

#### Reader::get_payload_schema_version

```rust
pub fn get_payload_schema_version(self) -> u32
```

#### Reader::get_invocation_id

```rust
pub fn get_invocation_id(self) -> Result<Reader<'a>>
```

#### Reader::has_invocation_id

```rust
pub fn has_invocation_id(&self) -> bool
```

#### Reader::get_command_type

```rust
pub fn get_command_type(self) -> Result<Reader<'a>>
```

#### Reader::has_command_type

```rust
pub fn has_command_type(&self) -> bool
```

#### Reader::get_payload_json

```rust
pub fn get_payload_json(self) -> Result<Reader<'a>>
```

#### Reader::has_payload_json

```rust
pub fn has_payload_json(&self) -> bool
```

#### Reader::get_idempotency_key

```rust
pub fn get_idempotency_key(self) -> Result<Reader<'a>>
```

#### Reader::has_idempotency_key

```rust
pub fn has_idempotency_key(&self) -> bool
```

#### Reader::get_attempt

```rust
pub fn get_attempt(self) -> u32
```

#### Reader::get_correlation_id

```rust
pub fn get_correlation_id(self) -> Result<Reader<'a>>
```

#### Reader::has_correlation_id

```rust
pub fn has_correlation_id(&self) -> bool
```

#### Reader::get_causation_id

```rust
pub fn get_causation_id(self) -> Result<Reader<'a>>
```

#### Reader::has_causation_id

```rust
pub fn has_causation_id(&self) -> bool
```

#### Reader::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
```

#### Reader::get_checkpoint_json

```rust
pub fn get_checkpoint_json(self) -> Result<Reader<'a>>
```

#### Reader::has_checkpoint_json

```rust
pub fn has_checkpoint_json(&self) -> bool
```

#### Reader::get_checkpoint_schema_version

```rust
pub fn get_checkpoint_schema_version(self) -> u32
```

#### Reader::get_invocation_sequence

```rust
pub fn get_invocation_sequence(self) -> u32
```

#### Reader::get_step_id

```rust
pub fn get_step_id(self) -> Result<Reader<'a>>
```

#### Reader::has_step_id

```rust
pub fn has_step_id(&self) -> bool
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

#### Builder::get_payload_schema_version

```rust
pub fn get_payload_schema_version(self) -> u32
```

#### Builder::set_payload_schema_version

```rust
pub fn set_payload_schema_version(&mut self, value: u32)
```

#### Builder::get_invocation_id

```rust
pub fn get_invocation_id(self) -> Result<Builder<'a>>
```

#### Builder::set_invocation_id

```rust
pub fn set_invocation_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_invocation_id

```rust
pub fn init_invocation_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_invocation_id

```rust
pub fn has_invocation_id(&self) -> bool
```

#### Builder::get_command_type

```rust
pub fn get_command_type(self) -> Result<Builder<'a>>
```

#### Builder::set_command_type

```rust
pub fn set_command_type(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_command_type

```rust
pub fn init_command_type(self, size: u32) -> Builder<'a>
```

#### Builder::has_command_type

```rust
pub fn has_command_type(&self) -> bool
```

#### Builder::get_payload_json

```rust
pub fn get_payload_json(self) -> Result<Builder<'a>>
```

#### Builder::set_payload_json

```rust
pub fn set_payload_json(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_payload_json

```rust
pub fn init_payload_json(self, size: u32) -> Builder<'a>
```

#### Builder::has_payload_json

```rust
pub fn has_payload_json(&self) -> bool
```

#### Builder::get_idempotency_key

```rust
pub fn get_idempotency_key(self) -> Result<Builder<'a>>
```

#### Builder::set_idempotency_key

```rust
pub fn set_idempotency_key(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_idempotency_key

```rust
pub fn init_idempotency_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_idempotency_key

```rust
pub fn has_idempotency_key(&self) -> bool
```

#### Builder::get_attempt

```rust
pub fn get_attempt(self) -> u32
```

#### Builder::set_attempt

```rust
pub fn set_attempt(&mut self, value: u32)
```

#### Builder::get_correlation_id

```rust
pub fn get_correlation_id(self) -> Result<Builder<'a>>
```

#### Builder::set_correlation_id

```rust
pub fn set_correlation_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_correlation_id

```rust
pub fn init_correlation_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_correlation_id

```rust
pub fn has_correlation_id(&self) -> bool
```

#### Builder::get_causation_id

```rust
pub fn get_causation_id(self) -> Result<Builder<'a>>
```

#### Builder::set_causation_id

```rust
pub fn set_causation_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_causation_id

```rust
pub fn init_causation_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_causation_id

```rust
pub fn has_causation_id(&self) -> bool
```

#### Builder::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
```

#### Builder::set_deadline_unix_ms

```rust
pub fn set_deadline_unix_ms(&mut self, value: u64)
```

#### Builder::get_checkpoint_json

```rust
pub fn get_checkpoint_json(self) -> Result<Builder<'a>>
```

#### Builder::set_checkpoint_json

```rust
pub fn set_checkpoint_json(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_checkpoint_json

```rust
pub fn init_checkpoint_json(self, size: u32) -> Builder<'a>
```

#### Builder::has_checkpoint_json

```rust
pub fn has_checkpoint_json(&self) -> bool
```

#### Builder::get_checkpoint_schema_version

```rust
pub fn get_checkpoint_schema_version(self) -> u32
```

#### Builder::set_checkpoint_schema_version

```rust
pub fn set_checkpoint_schema_version(&mut self, value: u32)
```

#### Builder::get_invocation_sequence

```rust
pub fn get_invocation_sequence(self) -> u32
```

#### Builder::set_invocation_sequence

```rust
pub fn set_invocation_sequence(&mut self, value: u32)
```

#### Builder::get_step_id

```rust
pub fn get_step_id(self) -> Result<Builder<'a>>
```

#### Builder::set_step_id

```rust
pub fn set_step_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_step_id

```rust
pub fn init_step_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_step_id

```rust
pub fn has_step_id(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
