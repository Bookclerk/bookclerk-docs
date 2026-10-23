# bookclerk_plugin_abi::plugin_capnp::kv_list_ok

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

#### Reader::get_keys

```rust
pub fn get_keys(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_keys

```rust
pub fn has_keys(&self) -> bool
```

#### Reader::get_list_complete

```rust
pub fn get_list_complete(self) -> bool
```

#### Reader::get_cursor

```rust
pub fn get_cursor(self) -> Result<Reader<'a>>
```

#### Reader::has_cursor

```rust
pub fn has_cursor(&self) -> bool
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

#### Builder::get_keys

```rust
pub fn get_keys(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_keys

```rust
pub fn set_keys(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_keys

```rust
pub fn init_keys(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_keys

```rust
pub fn has_keys(&self) -> bool
```

#### Builder::get_list_complete

```rust
pub fn get_list_complete(self) -> bool
```

#### Builder::set_list_complete

```rust
pub fn set_list_complete(&mut self, value: bool)
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

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
