# bookclerk_plugin_abi::plugin_capnp::cli_arg_spec

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

#### Reader::get_name

```rust
pub fn get_name(self) -> Result<Reader<'a>>
```

#### Reader::has_name

```rust
pub fn has_name(&self) -> bool
```

#### Reader::get_long

```rust
pub fn get_long(self) -> Result<Reader<'a>>
```

#### Reader::has_long

```rust
pub fn has_long(&self) -> bool
```

#### Reader::get_short

```rust
pub fn get_short(self) -> Result<Reader<'a>>
```

#### Reader::has_short

```rust
pub fn has_short(&self) -> bool
```

#### Reader::get_kind

```rust
pub fn get_kind(self) -> Result<CliArgKind, NotInSchema>
```

#### Reader::get_required

```rust
pub fn get_required(self) -> bool
```

#### Reader::get_default

```rust
pub fn get_default(self) -> Result<Reader<'a>>
```

#### Reader::has_default

```rust
pub fn has_default(&self) -> bool
```

#### Reader::get_about

```rust
pub fn get_about(self) -> Result<Reader<'a>>
```

#### Reader::has_about

```rust
pub fn has_about(&self) -> bool
```

#### Reader::get_positional

```rust
pub fn get_positional(self) -> bool
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

#### Builder::get_name

```rust
pub fn get_name(self) -> Result<Builder<'a>>
```

#### Builder::set_name

```rust
pub fn set_name(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_name

```rust
pub fn init_name(self, size: u32) -> Builder<'a>
```

#### Builder::has_name

```rust
pub fn has_name(&self) -> bool
```

#### Builder::get_long

```rust
pub fn get_long(self) -> Result<Builder<'a>>
```

#### Builder::set_long

```rust
pub fn set_long(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_long

```rust
pub fn init_long(self, size: u32) -> Builder<'a>
```

#### Builder::has_long

```rust
pub fn has_long(&self) -> bool
```

#### Builder::get_short

```rust
pub fn get_short(self) -> Result<Builder<'a>>
```

#### Builder::set_short

```rust
pub fn set_short(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_short

```rust
pub fn init_short(self, size: u32) -> Builder<'a>
```

#### Builder::has_short

```rust
pub fn has_short(&self) -> bool
```

#### Builder::get_kind

```rust
pub fn get_kind(self) -> Result<CliArgKind, NotInSchema>
```

#### Builder::set_kind

```rust
pub fn set_kind(&mut self, value: CliArgKind)
```

#### Builder::get_required

```rust
pub fn get_required(self) -> bool
```

#### Builder::set_required

```rust
pub fn set_required(&mut self, value: bool)
```

#### Builder::get_default

```rust
pub fn get_default(self) -> Result<Builder<'a>>
```

#### Builder::set_default

```rust
pub fn set_default(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_default

```rust
pub fn init_default(self, size: u32) -> Builder<'a>
```

#### Builder::has_default

```rust
pub fn has_default(&self) -> bool
```

#### Builder::get_about

```rust
pub fn get_about(self) -> Result<Builder<'a>>
```

#### Builder::set_about

```rust
pub fn set_about(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_about

```rust
pub fn init_about(self, size: u32) -> Builder<'a>
```

#### Builder::has_about

```rust
pub fn has_about(&self) -> bool
```

#### Builder::get_positional

```rust
pub fn get_positional(self) -> bool
```

#### Builder::set_positional

```rust
pub fn set_positional(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
