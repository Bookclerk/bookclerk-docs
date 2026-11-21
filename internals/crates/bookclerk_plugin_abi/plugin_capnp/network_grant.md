# bookclerk_plugin_abi::plugin_capnp::network_grant

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

#### Reader::get_outbound

```rust
pub fn get_outbound(self) -> bool
```

#### Reader::get_domains

```rust
pub fn get_domains(self) -> Result<Reader<'a>>
```

#### Reader::has_domains

```rust
pub fn has_domains(&self) -> bool
```

#### Reader::get_tcp

```rust
pub fn get_tcp(self) -> Result<Reader<'a>>
```

#### Reader::has_tcp

```rust
pub fn has_tcp(&self) -> bool
```

#### Reader::get_settings

```rust
pub fn get_settings(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_settings

```rust
pub fn has_settings(&self) -> bool
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

#### Builder::get_outbound

```rust
pub fn get_outbound(self) -> bool
```

#### Builder::set_outbound

```rust
pub fn set_outbound(&mut self, value: bool)
```

#### Builder::get_domains

```rust
pub fn get_domains(self) -> Result<Builder<'a>>
```

#### Builder::set_domains

```rust
pub fn set_domains(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_domains

```rust
pub fn init_domains(self, size: u32) -> Builder<'a>
```

#### Builder::has_domains

```rust
pub fn has_domains(&self) -> bool
```

#### Builder::get_tcp

```rust
pub fn get_tcp(self) -> Result<Builder<'a>>
```

#### Builder::set_tcp

```rust
pub fn set_tcp(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_tcp

```rust
pub fn init_tcp(self, size: u32) -> Builder<'a>
```

#### Builder::has_tcp

```rust
pub fn has_tcp(&self) -> bool
```

#### Builder::get_settings

```rust
pub fn get_settings(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_settings

```rust
pub fn set_settings(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_settings

```rust
pub fn init_settings(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_settings

```rust
pub fn has_settings(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
