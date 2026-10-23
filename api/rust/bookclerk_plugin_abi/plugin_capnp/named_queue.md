# bookclerk_plugin_abi::plugin_capnp::named_queue

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

#### Reader::get_name

```rust
pub fn get_name(self) -> Result<Reader<'a>>
```

#### Reader::has_name

```rust
pub fn has_name(&self) -> bool
```

#### Reader::get_queue

```rust
pub fn get_queue(self) -> Result<Reader<'a>>
```

#### Reader::has_queue

```rust
pub fn has_queue(&self) -> bool
```

#### Reader::get_producer

```rust
pub fn get_producer(self) -> Result<Client>
```

#### Reader::has_producer

```rust
pub fn has_producer(&self) -> bool
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

#### Builder::get_name

```rust
pub fn get_name(self) -> Result<Builder<'a>>
```

#### Builder::set_name

```rust
pub fn set_name(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_name

```rust
pub fn init_name(self, size: u32) -> Builder<'a>
```

#### Builder::has_name

```rust
pub fn has_name(&self) -> bool
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

#### Builder::get_producer

```rust
pub fn get_producer(self) -> Result<Client>
```

#### Builder::set_producer

```rust
pub fn set_producer(&mut self, value: Client)
```

#### Builder::has_producer

```rust
pub fn has_producer(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_producer

```rust
pub fn get_producer(&self) -> Client
```

Implements: `FromTypelessPipeline`
