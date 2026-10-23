# bookclerk_plugin_abi::plugin_capnp::purchase_hint_result

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

#### Reader::get_found

```rust
pub fn get_found(self) -> bool
```

#### Reader::get_hint

```rust
pub fn get_hint(self) -> Result<Reader<'a>>
```

#### Reader::has_hint

```rust
pub fn has_hint(&self) -> bool
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

#### Builder::get_found

```rust
pub fn get_found(self) -> bool
```

#### Builder::set_found

```rust
pub fn set_found(&mut self, value: bool)
```

#### Builder::get_hint

```rust
pub fn get_hint(self) -> Result<Builder<'a>>
```

#### Builder::set_hint

```rust
pub fn set_hint(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_hint

```rust
pub fn init_hint(self) -> Builder<'a>
```

#### Builder::has_hint

```rust
pub fn has_hint(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_hint

```rust
pub fn get_hint(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
