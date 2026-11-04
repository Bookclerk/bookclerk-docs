# bookclerk_plugin_abi::plugin_capnp::metadata_search_params

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

#### Reader::get_title

```rust
pub fn get_title(self) -> Result<Reader<'a>>
```

#### Reader::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Reader::get_author

```rust
pub fn get_author(self) -> Result<Reader<'a>>
```

#### Reader::has_author

```rust
pub fn has_author(&self) -> bool
```

#### Reader::get_language

```rust
pub fn get_language(self) -> Result<Reader<'a>>
```

#### Reader::has_language

```rust
pub fn has_language(&self) -> bool
```

#### Reader::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Reader<'a>>
```

#### Reader::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
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

#### Builder::get_title

```rust
pub fn get_title(self) -> Result<Builder<'a>>
```

#### Builder::set_title

```rust
pub fn set_title(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_title

```rust
pub fn init_title(self, size: u32) -> Builder<'a>
```

#### Builder::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Builder::get_author

```rust
pub fn get_author(self) -> Result<Builder<'a>>
```

#### Builder::set_author

```rust
pub fn set_author(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_author

```rust
pub fn init_author(self, size: u32) -> Builder<'a>
```

#### Builder::has_author

```rust
pub fn has_author(&self) -> bool
```

#### Builder::get_language

```rust
pub fn get_language(self) -> Result<Builder<'a>>
```

#### Builder::set_language

```rust
pub fn set_language(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_language

```rust
pub fn init_language(self, size: u32) -> Builder<'a>
```

#### Builder::has_language

```rust
pub fn has_language(&self) -> bool
```

#### Builder::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Builder<'a>>
```

#### Builder::set_marketplace

```rust
pub fn set_marketplace(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_marketplace

```rust
pub fn init_marketplace(self, size: u32) -> Builder<'a>
```

#### Builder::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
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
