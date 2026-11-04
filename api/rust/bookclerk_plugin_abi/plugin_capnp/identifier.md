# bookclerk_plugin_abi::plugin_capnp::identifier

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

#### Reader::get_scheme

```rust
pub fn get_scheme(self) -> Result<IdentifierScheme, NotInSchema>
```

#### Reader::get_value

```rust
pub fn get_value(self) -> Result<Reader<'a>>
```

#### Reader::has_value

```rust
pub fn has_value(&self) -> bool
```

#### Reader::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Reader<'a>>
```

#### Reader::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
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

#### Builder::get_scheme

```rust
pub fn get_scheme(self) -> Result<IdentifierScheme, NotInSchema>
```

#### Builder::set_scheme

```rust
pub fn set_scheme(&mut self, value: IdentifierScheme)
```

#### Builder::get_value

```rust
pub fn get_value(self) -> Result<Builder<'a>>
```

#### Builder::set_value

```rust
pub fn set_value(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_value

```rust
pub fn init_value(self, size: u32) -> Builder<'a>
```

#### Builder::has_value

```rust
pub fn has_value(&self) -> bool
```

#### Builder::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Builder<'a>>
```

#### Builder::set_marketplace

```rust
pub fn set_marketplace(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_marketplace

```rust
pub fn init_marketplace(self, size: u32) -> Builder<'a>
```

#### Builder::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
