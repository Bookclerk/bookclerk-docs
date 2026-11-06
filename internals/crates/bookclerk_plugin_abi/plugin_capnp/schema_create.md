# bookclerk_plugin_abi::plugin_capnp::schema_create

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

#### Reader::get_schema

```rust
pub fn get_schema(self) -> Result<Reader<'a>>
```

#### Reader::has_schema

```rust
pub fn has_schema(&self) -> bool
```

#### Reader::get_fingerprint

```rust
pub fn get_fingerprint(self) -> Result<Reader<'a>>
```

#### Reader::has_fingerprint

```rust
pub fn has_fingerprint(&self) -> bool
```

#### Reader::get_noop

```rust
pub fn get_noop(self) -> bool
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

#### Builder::get_schema

```rust
pub fn get_schema(self) -> Result<Builder<'a>>
```

#### Builder::set_schema

```rust
pub fn set_schema(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_schema

```rust
pub fn init_schema(self) -> Builder<'a>
```

#### Builder::has_schema

```rust
pub fn has_schema(&self) -> bool
```

#### Builder::get_fingerprint

```rust
pub fn get_fingerprint(self) -> Result<Builder<'a>>
```

#### Builder::set_fingerprint

```rust
pub fn set_fingerprint(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_fingerprint

```rust
pub fn init_fingerprint(self, size: u32) -> Builder<'a>
```

#### Builder::has_fingerprint

```rust
pub fn has_fingerprint(&self) -> bool
```

#### Builder::get_noop

```rust
pub fn get_noop(self) -> bool
```

#### Builder::set_noop

```rust
pub fn set_noop(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_schema

```rust
pub fn get_schema(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
