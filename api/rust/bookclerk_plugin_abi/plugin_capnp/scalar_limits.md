# bookclerk_plugin_abi::plugin_capnp::scalar_limits

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

#### Reader::get_max_scalar_bytes

```rust
pub fn get_max_scalar_bytes(self) -> u32
```

#### Reader::get_max_stream_window_bytes

```rust
pub fn get_max_stream_window_bytes(self) -> u32
```

#### Reader::get_max_list_page

```rust
pub fn get_max_list_page(self) -> u32
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

#### Builder::get_max_scalar_bytes

```rust
pub fn get_max_scalar_bytes(self) -> u32
```

#### Builder::set_max_scalar_bytes

```rust
pub fn set_max_scalar_bytes(&mut self, value: u32)
```

#### Builder::get_max_stream_window_bytes

```rust
pub fn get_max_stream_window_bytes(self) -> u32
```

#### Builder::set_max_stream_window_bytes

```rust
pub fn set_max_stream_window_bytes(&mut self, value: u32)
```

#### Builder::get_max_list_page

```rust
pub fn get_max_list_page(self) -> u32
```

#### Builder::set_max_list_page

```rust
pub fn set_max_list_page(&mut self, value: u32)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
