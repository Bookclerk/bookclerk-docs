# bookclerk_plugin_abi::plugin_capnp::move_many_page

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

#### Reader::get_results

```rust
pub fn get_results(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_results

```rust
pub fn has_results(&self) -> bool
```

#### Reader::get_next_cursor

```rust
pub fn get_next_cursor(self) -> Result<Reader<'a>>
```

#### Reader::has_next_cursor

```rust
pub fn has_next_cursor(&self) -> bool
```

#### Reader::get_atomic

```rust
pub fn get_atomic(self) -> bool
```

#### Reader::get_objects_moved

```rust
pub fn get_objects_moved(self) -> u64
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

#### Builder::get_results

```rust
pub fn get_results(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_results

```rust
pub fn set_results(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_results

```rust
pub fn init_results(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_results

```rust
pub fn has_results(&self) -> bool
```

#### Builder::get_next_cursor

```rust
pub fn get_next_cursor(self) -> Result<Builder<'a>>
```

#### Builder::set_next_cursor

```rust
pub fn set_next_cursor(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_next_cursor

```rust
pub fn init_next_cursor(self, size: u32) -> Builder<'a>
```

#### Builder::has_next_cursor

```rust
pub fn has_next_cursor(&self) -> bool
```

#### Builder::get_atomic

```rust
pub fn get_atomic(self) -> bool
```

#### Builder::set_atomic

```rust
pub fn set_atomic(&mut self, value: bool)
```

#### Builder::get_objects_moved

```rust
pub fn get_objects_moved(self) -> u64
```

#### Builder::set_objects_moved

```rust
pub fn set_objects_moved(&mut self, value: u64)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
