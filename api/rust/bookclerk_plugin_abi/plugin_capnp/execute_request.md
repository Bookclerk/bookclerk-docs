# bookclerk_plugin_abi::plugin_capnp::execute_request

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

#### Reader::get_operation_id

```rust
pub fn get_operation_id(self) -> Result<Reader<'a>>
```

#### Reader::has_operation_id

```rust
pub fn has_operation_id(&self) -> bool
```

#### Reader::get_request_hash

```rust
pub fn get_request_hash(self) -> Result<Reader<'a>>
```

#### Reader::has_request_hash

```rust
pub fn has_request_hash(&self) -> bool
```

#### Reader::get_statements

```rust
pub fn get_statements(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_statements

```rust
pub fn has_statements(&self) -> bool
```

#### Reader::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
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

#### Builder::get_operation_id

```rust
pub fn get_operation_id(self) -> Result<Builder<'a>>
```

#### Builder::set_operation_id

```rust
pub fn set_operation_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_operation_id

```rust
pub fn init_operation_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_operation_id

```rust
pub fn has_operation_id(&self) -> bool
```

#### Builder::get_request_hash

```rust
pub fn get_request_hash(self) -> Result<Builder<'a>>
```

#### Builder::set_request_hash

```rust
pub fn set_request_hash(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_request_hash

```rust
pub fn init_request_hash(self, size: u32) -> Builder<'a>
```

#### Builder::has_request_hash

```rust
pub fn has_request_hash(&self) -> bool
```

#### Builder::get_statements

```rust
pub fn get_statements(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_statements

```rust
pub fn set_statements(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_statements

```rust
pub fn init_statements(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_statements

```rust
pub fn has_statements(&self) -> bool
```

#### Builder::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
```

#### Builder::set_deadline_unix_ms

```rust
pub fn set_deadline_unix_ms(&mut self, value: u64)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
