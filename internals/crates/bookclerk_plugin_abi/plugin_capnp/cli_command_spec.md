# bookclerk_plugin_abi::plugin_capnp::cli_command_spec

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

#### Reader::get_about

```rust
pub fn get_about(self) -> Result<Reader<'a>>
```

#### Reader::has_about

```rust
pub fn has_about(&self) -> bool
```

#### Reader::get_args

```rust
pub fn get_args(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_args

```rust
pub fn has_args(&self) -> bool
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

#### Builder::get_args

```rust
pub fn get_args(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_args

```rust
pub fn set_args(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_args

```rust
pub fn init_args(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_args

```rust
pub fn has_args(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
