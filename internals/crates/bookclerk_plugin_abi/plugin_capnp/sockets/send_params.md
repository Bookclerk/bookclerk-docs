# bookclerk_plugin_abi::plugin_capnp::sockets::send_params

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

#### Reader::get_socket

```rust
pub fn get_socket(self) -> Result<Reader<'a>>
```

#### Reader::has_socket

```rust
pub fn has_socket(&self) -> bool
```

#### Reader::get_frame

```rust
pub fn get_frame(self) -> Result<Reader<'a>>
```

#### Reader::has_frame

```rust
pub fn has_frame(&self) -> bool
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

#### Builder::get_socket

```rust
pub fn get_socket(self) -> Result<Builder<'a>>
```

#### Builder::set_socket

```rust
pub fn set_socket(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_socket

```rust
pub fn init_socket(self, size: u32) -> Builder<'a>
```

#### Builder::has_socket

```rust
pub fn has_socket(&self) -> bool
```

#### Builder::get_frame

```rust
pub fn get_frame(self) -> Result<Builder<'a>>
```

#### Builder::set_frame

```rust
pub fn set_frame(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_frame

```rust
pub fn init_frame(self) -> Builder<'a>
```

#### Builder::has_frame

```rust
pub fn has_frame(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_frame

```rust
pub fn get_frame(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
