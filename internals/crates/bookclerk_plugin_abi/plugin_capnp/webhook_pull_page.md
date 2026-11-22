# bookclerk_plugin_abi::plugin_capnp::webhook_pull_page

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

#### Reader::get_deliveries

```rust
pub fn get_deliveries(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_deliveries

```rust
pub fn has_deliveries(&self) -> bool
```

#### Reader::get_missed

```rust
pub fn get_missed(self) -> bool
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

#### Builder::get_deliveries

```rust
pub fn get_deliveries(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_deliveries

```rust
pub fn set_deliveries(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_deliveries

```rust
pub fn init_deliveries(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_deliveries

```rust
pub fn has_deliveries(&self) -> bool
```

#### Builder::get_missed

```rust
pub fn get_missed(self) -> bool
```

#### Builder::set_missed

```rust
pub fn set_missed(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
