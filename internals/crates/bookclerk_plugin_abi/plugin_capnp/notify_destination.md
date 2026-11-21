# bookclerk_plugin_abi::plugin_capnp::notify_destination

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

#### Reader::get_kind

```rust
pub fn get_kind(self) -> Result<NotifyDestinationKind, NotInSchema>
```

#### Reader::get_destination_id

```rust
pub fn get_destination_id(self) -> Result<Reader<'a>>
```

#### Reader::has_destination_id

```rust
pub fn has_destination_id(&self) -> bool
```

#### Reader::get_address_json

```rust
pub fn get_address_json(self) -> Result<Reader<'a>>
```

#### Reader::has_address_json

```rust
pub fn has_address_json(&self) -> bool
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

#### Builder::get_kind

```rust
pub fn get_kind(self) -> Result<NotifyDestinationKind, NotInSchema>
```

#### Builder::set_kind

```rust
pub fn set_kind(&mut self, value: NotifyDestinationKind)
```

#### Builder::get_destination_id

```rust
pub fn get_destination_id(self) -> Result<Builder<'a>>
```

#### Builder::set_destination_id

```rust
pub fn set_destination_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_destination_id

```rust
pub fn init_destination_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_destination_id

```rust
pub fn has_destination_id(&self) -> bool
```

#### Builder::get_address_json

```rust
pub fn get_address_json(self) -> Result<Builder<'a>>
```

#### Builder::set_address_json

```rust
pub fn set_address_json(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_address_json

```rust
pub fn init_address_json(self, size: u32) -> Builder<'a>
```

#### Builder::has_address_json

```rust
pub fn has_address_json(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
