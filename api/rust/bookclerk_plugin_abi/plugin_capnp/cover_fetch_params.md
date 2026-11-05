# bookclerk_plugin_abi::plugin_capnp::cover_fetch_params

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

#### Reader::get_urls

```rust
pub fn get_urls(self) -> Result<Reader<'a>>
```

#### Reader::has_urls

```rust
pub fn has_urls(&self) -> bool
```

#### Reader::get_max_bytes

```rust
pub fn get_max_bytes(self) -> u32
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

#### Builder::get_urls

```rust
pub fn get_urls(self) -> Result<Builder<'a>>
```

#### Builder::set_urls

```rust
pub fn set_urls(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_urls

```rust
pub fn init_urls(self, size: u32) -> Builder<'a>
```

#### Builder::has_urls

```rust
pub fn has_urls(&self) -> bool
```

#### Builder::get_max_bytes

```rust
pub fn get_max_bytes(self) -> u32
```

#### Builder::set_max_bytes

```rust
pub fn set_max_bytes(&mut self, value: u32)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
