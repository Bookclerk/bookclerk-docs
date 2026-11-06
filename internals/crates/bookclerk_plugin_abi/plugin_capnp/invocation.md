# bookclerk_plugin_abi::plugin_capnp::invocation

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

#### Reader::get_id

```rust
pub fn get_id(self) -> Result<Reader<'a>>
```

#### Reader::has_id

```rust
pub fn has_id(&self) -> bool
```

#### Reader::get_account_id

```rust
pub fn get_account_id(self) -> Result<Reader<'a>>
```

#### Reader::has_account_id

```rust
pub fn has_account_id(&self) -> bool
```

#### Reader::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
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

#### Builder::get_account_id

```rust
pub fn get_account_id(self) -> Result<Builder<'a>>
```

#### Builder::set_account_id

```rust
pub fn set_account_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_account_id

```rust
pub fn init_account_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_account_id

```rust
pub fn has_account_id(&self) -> bool
```

#### Builder::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
```

#### Builder::set_deadline_unix_ms

```rust
pub fn set_deadline_unix_ms(&mut self, value: u64)
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
