# bookclerk_plugin_abi::plugin_capnp::move_many_request

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

#### Reader::get_target

```rust
pub fn get_target(self) -> Result<Reader<'a>>
```

#### Reader::has_target

```rust
pub fn has_target(&self) -> bool
```

#### Reader::get_cursor

```rust
pub fn get_cursor(self) -> Result<Reader<'a>>
```

#### Reader::has_cursor

```rust
pub fn has_cursor(&self) -> bool
```

#### Reader::get_limit

```rust
pub fn get_limit(self) -> u32
```

#### Reader::get_options

```rust
pub fn get_options(self) -> Result<Reader<'a>>
```

#### Reader::has_options

```rust
pub fn has_options(&self) -> bool
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

#### Builder::get_target

```rust
pub fn get_target(self) -> Result<Builder<'a>>
```

#### Builder::set_target

```rust
pub fn set_target(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_target

```rust
pub fn init_target(self) -> Builder<'a>
```

#### Builder::has_target

```rust
pub fn has_target(&self) -> bool
```

#### Builder::get_cursor

```rust
pub fn get_cursor(self) -> Result<Builder<'a>>
```

#### Builder::set_cursor

```rust
pub fn set_cursor(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cursor

```rust
pub fn init_cursor(self, size: u32) -> Builder<'a>
```

#### Builder::has_cursor

```rust
pub fn has_cursor(&self) -> bool
```

#### Builder::get_limit

```rust
pub fn get_limit(self) -> u32
```

#### Builder::set_limit

```rust
pub fn set_limit(&mut self, value: u32)
```

#### Builder::get_options

```rust
pub fn get_options(self) -> Result<Builder<'a>>
```

#### Builder::set_options

```rust
pub fn set_options(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_options

```rust
pub fn init_options(self) -> Builder<'a>
```

#### Builder::has_options

```rust
pub fn has_options(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_target

```rust
pub fn get_target(&self) -> Pipeline
```

#### Pipeline::get_options

```rust
pub fn get_options(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
