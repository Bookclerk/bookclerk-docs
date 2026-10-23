# bookclerk_plugin_abi::plugin_capnp::plugin_event

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

#### Reader::get_event_type

```rust
pub fn get_event_type(self) -> Result<Reader<'a>>
```

#### Reader::has_event_type

```rust
pub fn has_event_type(&self) -> bool
```

#### Reader::get_schema_version

```rust
pub fn get_schema_version(self) -> u32
```

#### Reader::get_deduplication_key

```rust
pub fn get_deduplication_key(self) -> Result<Reader<'a>>
```

#### Reader::has_deduplication_key

```rust
pub fn has_deduplication_key(&self) -> bool
```

#### Reader::get_payload

```rust
pub fn get_payload(self) -> Result<Reader<'a>>
```

#### Reader::has_payload

```rust
pub fn has_payload(&self) -> bool
```

#### Reader::get_occurred_at_unix_ms

```rust
pub fn get_occurred_at_unix_ms(self) -> u64
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

#### Builder::get_event_type

```rust
pub fn get_event_type(self) -> Result<Builder<'a>>
```

#### Builder::set_event_type

```rust
pub fn set_event_type(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_event_type

```rust
pub fn init_event_type(self, size: u32) -> Builder<'a>
```

#### Builder::has_event_type

```rust
pub fn has_event_type(&self) -> bool
```

#### Builder::get_schema_version

```rust
pub fn get_schema_version(self) -> u32
```

#### Builder::set_schema_version

```rust
pub fn set_schema_version(&mut self, value: u32)
```

#### Builder::get_deduplication_key

```rust
pub fn get_deduplication_key(self) -> Result<Builder<'a>>
```

#### Builder::set_deduplication_key

```rust
pub fn set_deduplication_key(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_deduplication_key

```rust
pub fn init_deduplication_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_deduplication_key

```rust
pub fn has_deduplication_key(&self) -> bool
```

#### Builder::get_payload

```rust
pub fn get_payload(self) -> Result<Builder<'a>>
```

#### Builder::set_payload

```rust
pub fn set_payload(&mut self, value: Reader<'_>)
```

#### Builder::init_payload

```rust
pub fn init_payload(self, size: u32) -> Builder<'a>
```

#### Builder::has_payload

```rust
pub fn has_payload(&self) -> bool
```

#### Builder::get_occurred_at_unix_ms

```rust
pub fn get_occurred_at_unix_ms(self) -> u64
```

#### Builder::set_occurred_at_unix_ms

```rust
pub fn set_occurred_at_unix_ms(&mut self, value: u64)
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

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
