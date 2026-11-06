# bookclerk_plugin_abi::plugin_capnp::queue_message

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

#### Reader::get_timestamp_unix_ms

```rust
pub fn get_timestamp_unix_ms(self) -> u64
```

#### Reader::get_body

```rust
pub fn get_body(self) -> Result<Reader<'a>>
```

#### Reader::has_body

```rust
pub fn has_body(&self) -> bool
```

#### Reader::get_content_type

```rust
pub fn get_content_type(self) -> Result<Reader<'a>>
```

#### Reader::has_content_type

```rust
pub fn has_content_type(&self) -> bool
```

#### Reader::get_attempts

```rust
pub fn get_attempts(self) -> u32
```

#### Reader::get_source

```rust
pub fn get_source(self) -> Result<Reader<'a>>
```

#### Reader::has_source

```rust
pub fn has_source(&self) -> bool
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

#### Builder::get_timestamp_unix_ms

```rust
pub fn get_timestamp_unix_ms(self) -> u64
```

#### Builder::set_timestamp_unix_ms

```rust
pub fn set_timestamp_unix_ms(&mut self, value: u64)
```

#### Builder::get_body

```rust
pub fn get_body(self) -> Result<Builder<'a>>
```

#### Builder::set_body

```rust
pub fn set_body(&mut self, value: Reader<'_>)
```

#### Builder::init_body

```rust
pub fn init_body(self, size: u32) -> Builder<'a>
```

#### Builder::has_body

```rust
pub fn has_body(&self) -> bool
```

#### Builder::get_content_type

```rust
pub fn get_content_type(self) -> Result<Builder<'a>>
```

#### Builder::set_content_type

```rust
pub fn set_content_type(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_content_type

```rust
pub fn init_content_type(self, size: u32) -> Builder<'a>
```

#### Builder::has_content_type

```rust
pub fn has_content_type(&self) -> bool
```

#### Builder::get_attempts

```rust
pub fn get_attempts(self) -> u32
```

#### Builder::set_attempts

```rust
pub fn set_attempts(&mut self, value: u32)
```

#### Builder::get_source

```rust
pub fn get_source(self) -> Result<Builder<'a>>
```

#### Builder::set_source

```rust
pub fn set_source(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_source

```rust
pub fn init_source(self, size: u32) -> Builder<'a>
```

#### Builder::has_source

```rust
pub fn has_source(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
