# bookclerk_plugin_abi::plugin_capnp::metadata_file_render_params

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

#### Reader::get_record

```rust
pub fn get_record(self) -> Result<Reader<'a>>
```

#### Reader::has_record

```rust
pub fn has_record(&self) -> bool
```

#### Reader::get_fields

```rust
pub fn get_fields(self) -> Result<Reader<'a, MetadataField>>
```

#### Reader::has_fields

```rust
pub fn has_fields(&self) -> bool
```

#### Reader::get_existing

```rust
pub fn get_existing(self) -> Result<Reader<'a>>
```

#### Reader::has_existing

```rust
pub fn has_existing(&self) -> bool
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

#### Builder::get_record

```rust
pub fn get_record(self) -> Result<Builder<'a>>
```

#### Builder::set_record

```rust
pub fn set_record(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_record

```rust
pub fn init_record(self) -> Builder<'a>
```

#### Builder::has_record

```rust
pub fn has_record(&self) -> bool
```

#### Builder::get_fields

```rust
pub fn get_fields(self) -> Result<Builder<'a, MetadataField>>
```

#### Builder::set_fields

```rust
pub fn set_fields(&mut self, value: impl SetterInput<Owned<MetadataField>>) -> Result<()>
```

#### Builder::init_fields

```rust
pub fn init_fields(self, size: u32) -> Builder<'a, MetadataField>
```

#### Builder::has_fields

```rust
pub fn has_fields(&self) -> bool
```

#### Builder::get_existing

```rust
pub fn get_existing(self) -> Result<Builder<'a>>
```

#### Builder::set_existing

```rust
pub fn set_existing(&mut self, value: Reader<'_>)
```

#### Builder::init_existing

```rust
pub fn init_existing(self, size: u32) -> Builder<'a>
```

#### Builder::has_existing

```rust
pub fn has_existing(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_record

```rust
pub fn get_record(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
