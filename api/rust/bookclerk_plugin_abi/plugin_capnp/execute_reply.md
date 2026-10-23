# bookclerk_plugin_abi::plugin_capnp::execute_reply

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

#### Reader::get_statements

```rust
pub fn get_statements(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_statements

```rust
pub fn has_statements(&self) -> bool
```

#### Reader::get_timing

```rust
pub fn get_timing(self) -> Result<Reader<'a>>
```

#### Reader::has_timing

```rust
pub fn has_timing(&self) -> bool
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

#### Builder::get_timing

```rust
pub fn get_timing(self) -> Result<Builder<'a>>
```

#### Builder::set_timing

```rust
pub fn set_timing(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_timing

```rust
pub fn init_timing(self) -> Builder<'a>
```

#### Builder::has_timing

```rust
pub fn has_timing(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_timing

```rust
pub fn get_timing(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
