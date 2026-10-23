# bookclerk_plugin_abi::plugin_capnp::queue_batch

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

#### Reader::get_queue

```rust
pub fn get_queue(self) -> Result<Reader<'a>>
```

#### Reader::has_queue

```rust
pub fn has_queue(&self) -> bool
```

#### Reader::get_messages

```rust
pub fn get_messages(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_messages

```rust
pub fn has_messages(&self) -> bool
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

#### Builder::get_queue

```rust
pub fn get_queue(self) -> Result<Builder<'a>>
```

#### Builder::set_queue

```rust
pub fn set_queue(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_queue

```rust
pub fn init_queue(self, size: u32) -> Builder<'a>
```

#### Builder::has_queue

```rust
pub fn has_queue(&self) -> bool
```

#### Builder::get_messages

```rust
pub fn get_messages(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_messages

```rust
pub fn set_messages(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_messages

```rust
pub fn init_messages(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_messages

```rust
pub fn has_messages(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
