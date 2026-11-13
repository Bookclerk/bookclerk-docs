# bookclerk_plugin_abi::plugin_capnp::media_support

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

#### Reader::get_work_kinds

```rust
pub fn get_work_kinds(self) -> Result<Reader<'a, WorkKind>>
```

#### Reader::has_work_kinds

```rust
pub fn has_work_kinds(&self) -> bool
```

#### Reader::get_lists

```rust
pub fn get_lists(self) -> Result<Reader<'a, MediaKind>>
```

#### Reader::has_lists

```rust
pub fn has_lists(&self) -> bool
```

#### Reader::get_fetches

```rust
pub fn get_fetches(self) -> Result<Reader<'a, MediaKind>>
```

#### Reader::has_fetches

```rust
pub fn has_fetches(&self) -> bool
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

#### Builder::get_work_kinds

```rust
pub fn get_work_kinds(self) -> Result<Builder<'a, WorkKind>>
```

#### Builder::set_work_kinds

```rust
pub fn set_work_kinds(&mut self, value: impl SetterInput<Owned<WorkKind>>) -> Result<()>
```

#### Builder::init_work_kinds

```rust
pub fn init_work_kinds(self, size: u32) -> Builder<'a, WorkKind>
```

#### Builder::has_work_kinds

```rust
pub fn has_work_kinds(&self) -> bool
```

#### Builder::get_lists

```rust
pub fn get_lists(self) -> Result<Builder<'a, MediaKind>>
```

#### Builder::set_lists

```rust
pub fn set_lists(&mut self, value: impl SetterInput<Owned<MediaKind>>) -> Result<()>
```

#### Builder::init_lists

```rust
pub fn init_lists(self, size: u32) -> Builder<'a, MediaKind>
```

#### Builder::has_lists

```rust
pub fn has_lists(&self) -> bool
```

#### Builder::get_fetches

```rust
pub fn get_fetches(self) -> Result<Builder<'a, MediaKind>>
```

#### Builder::set_fetches

```rust
pub fn set_fetches(&mut self, value: impl SetterInput<Owned<MediaKind>>) -> Result<()>
```

#### Builder::init_fetches

```rust
pub fn init_fetches(self, size: u32) -> Builder<'a, MediaKind>
```

#### Builder::has_fetches

```rust
pub fn has_fetches(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
