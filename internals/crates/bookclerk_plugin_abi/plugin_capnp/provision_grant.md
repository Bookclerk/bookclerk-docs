# bookclerk_plugin_abi::plugin_capnp::provision_grant

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
pub fn get_kind(self) -> Result<ProvisionGrantKind, NotInSchema>
```

#### Reader::get_token

```rust
pub fn get_token(self) -> Result<Reader<'a>>
```

#### Reader::has_token

```rust
pub fn has_token(&self) -> bool
```

#### Reader::get_expires_at_unix_ms

```rust
pub fn get_expires_at_unix_ms(self) -> u64
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
pub fn get_kind(self) -> Result<ProvisionGrantKind, NotInSchema>
```

#### Builder::set_kind

```rust
pub fn set_kind(&mut self, value: ProvisionGrantKind)
```

#### Builder::get_token

```rust
pub fn get_token(self) -> Result<Builder<'a>>
```

#### Builder::set_token

```rust
pub fn set_token(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_token

```rust
pub fn init_token(self, size: u32) -> Builder<'a>
```

#### Builder::has_token

```rust
pub fn has_token(&self) -> bool
```

#### Builder::get_expires_at_unix_ms

```rust
pub fn get_expires_at_unix_ms(self) -> u64
```

#### Builder::set_expires_at_unix_ms

```rust
pub fn set_expires_at_unix_ms(&mut self, value: u64)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
