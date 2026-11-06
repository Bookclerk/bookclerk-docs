# bookclerk_plugin_abi::plugin_capnp::cover_condition

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

#### Reader::get_if_none_match

```rust
pub fn get_if_none_match(self) -> Result<Reader<'a>>
```

#### Reader::has_if_none_match

```rust
pub fn has_if_none_match(&self) -> bool
```

#### Reader::get_if_modified_since

```rust
pub fn get_if_modified_since(self) -> Result<Reader<'a>>
```

#### Reader::has_if_modified_since

```rust
pub fn has_if_modified_since(&self) -> bool
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

#### Builder::get_if_none_match

```rust
pub fn get_if_none_match(self) -> Result<Builder<'a>>
```

#### Builder::set_if_none_match

```rust
pub fn set_if_none_match(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_if_none_match

```rust
pub fn init_if_none_match(self, size: u32) -> Builder<'a>
```

#### Builder::has_if_none_match

```rust
pub fn has_if_none_match(&self) -> bool
```

#### Builder::get_if_modified_since

```rust
pub fn get_if_modified_since(self) -> Result<Builder<'a>>
```

#### Builder::set_if_modified_since

```rust
pub fn set_if_modified_since(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_if_modified_since

```rust
pub fn init_if_modified_since(self, size: u32) -> Builder<'a>
```

#### Builder::has_if_modified_since

```rust
pub fn has_if_modified_since(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
