# bookclerk_plugin_abi::plugin_capnp::event_read_request

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

#### Reader::get_after

```rust
pub fn get_after(self) -> Result<Reader<'a>>
```

#### Reader::has_after

```rust
pub fn has_after(&self) -> bool
```

#### Reader::get_since_unix_ms

```rust
pub fn get_since_unix_ms(self) -> u64
```

#### Reader::get_types

```rust
pub fn get_types(self) -> Result<Reader<'a>>
```

#### Reader::has_types

```rust
pub fn has_types(&self) -> bool
```

#### Reader::get_limit

```rust
pub fn get_limit(self) -> u32
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

#### Builder::get_after

```rust
pub fn get_after(self) -> Result<Builder<'a>>
```

#### Builder::set_after

```rust
pub fn set_after(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_after

```rust
pub fn init_after(self, size: u32) -> Builder<'a>
```

#### Builder::has_after

```rust
pub fn has_after(&self) -> bool
```

#### Builder::get_since_unix_ms

```rust
pub fn get_since_unix_ms(self) -> u64
```

#### Builder::set_since_unix_ms

```rust
pub fn set_since_unix_ms(&mut self, value: u64)
```

#### Builder::get_types

```rust
pub fn get_types(self) -> Result<Builder<'a>>
```

#### Builder::set_types

```rust
pub fn set_types(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_types

```rust
pub fn init_types(self, size: u32) -> Builder<'a>
```

#### Builder::has_types

```rust
pub fn has_types(&self) -> bool
```

#### Builder::get_limit

```rust
pub fn get_limit(self) -> u32
```

#### Builder::set_limit

```rust
pub fn set_limit(&mut self, value: u32)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
