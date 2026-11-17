# bookclerk_plugin_abi::plugin_capnp::metadata_file_parse_params

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

#### Reader::get_dialect

```rust
pub fn get_dialect(self) -> Result<Reader<'a>>
```

#### Reader::has_dialect

```rust
pub fn has_dialect(&self) -> bool
```

#### Reader::get_file_name

```rust
pub fn get_file_name(self) -> Result<Reader<'a>>
```

#### Reader::has_file_name

```rust
pub fn has_file_name(&self) -> bool
```

#### Reader::get_bytes

```rust
pub fn get_bytes(self) -> Result<Reader<'a>>
```

#### Reader::has_bytes

```rust
pub fn has_bytes(&self) -> bool
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

#### Builder::get_dialect

```rust
pub fn get_dialect(self) -> Result<Builder<'a>>
```

#### Builder::set_dialect

```rust
pub fn set_dialect(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_dialect

```rust
pub fn init_dialect(self, size: u32) -> Builder<'a>
```

#### Builder::has_dialect

```rust
pub fn has_dialect(&self) -> bool
```

#### Builder::get_file_name

```rust
pub fn get_file_name(self) -> Result<Builder<'a>>
```

#### Builder::set_file_name

```rust
pub fn set_file_name(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_file_name

```rust
pub fn init_file_name(self, size: u32) -> Builder<'a>
```

#### Builder::has_file_name

```rust
pub fn has_file_name(&self) -> bool
```

#### Builder::get_bytes

```rust
pub fn get_bytes(self) -> Result<Builder<'a>>
```

#### Builder::set_bytes

```rust
pub fn set_bytes(&mut self, value: Reader<'_>)
```

#### Builder::init_bytes

```rust
pub fn init_bytes(self, size: u32) -> Builder<'a>
```

#### Builder::has_bytes

```rust
pub fn has_bytes(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
