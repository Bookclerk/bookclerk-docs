# bookclerk_plugin_abi::plugin_capnp::kv_put_options

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

#### Reader::get_expiration_unix_ms

```rust
pub fn get_expiration_unix_ms(self) -> u64
```

#### Reader::get_expiration_ttl_seconds

```rust
pub fn get_expiration_ttl_seconds(self) -> u64
```

#### Reader::get_metadata

```rust
pub fn get_metadata(self) -> Result<Reader<'a>>
```

#### Reader::has_metadata

```rust
pub fn has_metadata(&self) -> bool
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

#### Builder::get_expiration_unix_ms

```rust
pub fn get_expiration_unix_ms(self) -> u64
```

#### Builder::set_expiration_unix_ms

```rust
pub fn set_expiration_unix_ms(&mut self, value: u64)
```

#### Builder::get_expiration_ttl_seconds

```rust
pub fn get_expiration_ttl_seconds(self) -> u64
```

#### Builder::set_expiration_ttl_seconds

```rust
pub fn set_expiration_ttl_seconds(&mut self, value: u64)
```

#### Builder::get_metadata

```rust
pub fn get_metadata(self) -> Result<Builder<'a>>
```

#### Builder::set_metadata

```rust
pub fn set_metadata(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_metadata

```rust
pub fn init_metadata(self, size: u32) -> Builder<'a>
```

#### Builder::has_metadata

```rust
pub fn has_metadata(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
