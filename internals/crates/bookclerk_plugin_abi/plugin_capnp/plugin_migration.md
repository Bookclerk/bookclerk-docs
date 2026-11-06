# bookclerk_plugin_abi::plugin_capnp::plugin_migration

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

#### Reader::get_id

```rust
pub fn get_id(self) -> Result<Reader<'a>>
```

#### Reader::has_id

```rust
pub fn has_id(&self) -> bool
```

#### Reader::get_operations

```rust
pub fn get_operations(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_operations

```rust
pub fn has_operations(&self) -> bool
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

#### Builder::get_id

```rust
pub fn get_id(self) -> Result<Builder<'a>>
```

#### Builder::set_id

```rust
pub fn set_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_id

```rust
pub fn init_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_id

```rust
pub fn has_id(&self) -> bool
```

#### Builder::get_operations

```rust
pub fn get_operations(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_operations

```rust
pub fn set_operations(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_operations

```rust
pub fn init_operations(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_operations

```rust
pub fn has_operations(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
