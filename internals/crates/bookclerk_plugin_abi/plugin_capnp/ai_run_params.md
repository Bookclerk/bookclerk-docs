# bookclerk_plugin_abi::plugin_capnp::ai_run_params

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

#### Reader::get_model

```rust
pub fn get_model(self) -> Result<Reader<'a>>
```

#### Reader::has_model

```rust
pub fn has_model(&self) -> bool
```

#### Reader::get_input

```rust
pub fn get_input(self) -> Result<Reader<'a>>
```

#### Reader::has_input

```rust
pub fn has_input(&self) -> bool
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

#### Builder::get_model

```rust
pub fn get_model(self) -> Result<Builder<'a>>
```

#### Builder::set_model

```rust
pub fn set_model(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_model

```rust
pub fn init_model(self, size: u32) -> Builder<'a>
```

#### Builder::has_model

```rust
pub fn has_model(&self) -> bool
```

#### Builder::get_input

```rust
pub fn get_input(self) -> Result<Builder<'a>>
```

#### Builder::set_input

```rust
pub fn set_input(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_input

```rust
pub fn init_input(self) -> Builder<'a>
```

#### Builder::has_input

```rust
pub fn has_input(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_input

```rust
pub fn get_input(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
