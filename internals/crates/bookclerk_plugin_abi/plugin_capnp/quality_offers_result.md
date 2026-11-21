# bookclerk_plugin_abi::plugin_capnp::quality_offers_result

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

#### Reader::get_offers

```rust
pub fn get_offers(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_offers

```rust
pub fn has_offers(&self) -> bool
```

#### Reader::get_refreshed_credentials

```rust
pub fn get_refreshed_credentials(self) -> Result<Reader<'a>>
```

#### Reader::has_refreshed_credentials

```rust
pub fn has_refreshed_credentials(&self) -> bool
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

#### Builder::get_offers

```rust
pub fn get_offers(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_offers

```rust
pub fn set_offers(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_offers

```rust
pub fn init_offers(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_offers

```rust
pub fn has_offers(&self) -> bool
```

#### Builder::get_refreshed_credentials

```rust
pub fn get_refreshed_credentials(self) -> Result<Builder<'a>>
```

#### Builder::set_refreshed_credentials

```rust
pub fn set_refreshed_credentials(&mut self, value: Reader<'_>)
```

#### Builder::init_refreshed_credentials

```rust
pub fn init_refreshed_credentials(self, size: u32) -> Builder<'a>
```

#### Builder::has_refreshed_credentials

```rust
pub fn has_refreshed_credentials(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
