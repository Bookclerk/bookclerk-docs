# bookclerk_plugin_abi::plugin_capnp::list_options

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

#### Reader::get_prefix

```rust
pub fn get_prefix(self) -> Result<Reader<'a>>
```

#### Reader::has_prefix

```rust
pub fn has_prefix(&self) -> bool
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

#### Builder::get_prefix

```rust
pub fn get_prefix(self) -> Result<Builder<'a>>
```

#### Builder::set_prefix

```rust
pub fn set_prefix(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_prefix

```rust
pub fn init_prefix(self, size: u32) -> Builder<'a>
```

#### Builder::has_prefix

```rust
pub fn has_prefix(&self) -> bool
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

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
