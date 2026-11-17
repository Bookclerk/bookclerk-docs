# bookclerk_plugin_abi::plugin_capnp::object_info

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

#### Reader::get_key

```rust
pub fn get_key(self) -> Result<Reader<'a>>
```

#### Reader::has_key

```rust
pub fn has_key(&self) -> bool
```

#### Reader::get_size

```rust
pub fn get_size(self) -> u64
```

#### Reader::get_etag

```rust
pub fn get_etag(self) -> Result<Reader<'a>>
```

#### Reader::has_etag

```rust
pub fn has_etag(&self) -> bool
```

#### Reader::get_modified_at_ms

```rust
pub fn get_modified_at_ms(self) -> u64
```

#### Reader::get_version_id

```rust
pub fn get_version_id(self) -> Result<Reader<'a>>
```

#### Reader::has_version_id

```rust
pub fn has_version_id(&self) -> bool
```

#### Reader::get_signature

```rust
pub fn get_signature(self) -> Result<Reader<'a>>
```

#### Reader::has_signature

```rust
pub fn has_signature(&self) -> bool
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

#### Builder::get_key

```rust
pub fn get_key(self) -> Result<Builder<'a>>
```

#### Builder::set_key

```rust
pub fn set_key(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_key

```rust
pub fn init_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_key

```rust
pub fn has_key(&self) -> bool
```

#### Builder::get_size

```rust
pub fn get_size(self) -> u64
```

#### Builder::set_size

```rust
pub fn set_size(&mut self, value: u64)
```

#### Builder::get_etag

```rust
pub fn get_etag(self) -> Result<Builder<'a>>
```

#### Builder::set_etag

```rust
pub fn set_etag(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_etag

```rust
pub fn init_etag(self, size: u32) -> Builder<'a>
```

#### Builder::has_etag

```rust
pub fn has_etag(&self) -> bool
```

#### Builder::get_modified_at_ms

```rust
pub fn get_modified_at_ms(self) -> u64
```

#### Builder::set_modified_at_ms

```rust
pub fn set_modified_at_ms(&mut self, value: u64)
```

#### Builder::get_version_id

```rust
pub fn get_version_id(self) -> Result<Builder<'a>>
```

#### Builder::set_version_id

```rust
pub fn set_version_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_version_id

```rust
pub fn init_version_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_version_id

```rust
pub fn has_version_id(&self) -> bool
```

#### Builder::get_signature

```rust
pub fn get_signature(self) -> Result<Builder<'a>>
```

#### Builder::set_signature

```rust
pub fn set_signature(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_signature

```rust
pub fn init_signature(self, size: u32) -> Builder<'a>
```

#### Builder::has_signature

```rust
pub fn has_signature(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
