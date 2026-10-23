# bookclerk_plugin_abi::plugin_capnp::brand

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

#### Reader::get_id

```rust
pub fn get_id(self) -> Result<Reader<'a>>
```

#### Reader::has_id

```rust
pub fn has_id(&self) -> bool
```

#### Reader::get_name

```rust
pub fn get_name(self) -> Result<Reader<'a>>
```

#### Reader::has_name

```rust
pub fn has_name(&self) -> bool
```

#### Reader::get_bg

```rust
pub fn get_bg(self) -> Result<Reader<'a>>
```

#### Reader::has_bg

```rust
pub fn has_bg(&self) -> bool
```

#### Reader::get_fg

```rust
pub fn get_fg(self) -> Result<Reader<'a>>
```

#### Reader::has_fg

```rust
pub fn has_fg(&self) -> bool
```

#### Reader::get_accent

```rust
pub fn get_accent(self) -> Result<Reader<'a>>
```

#### Reader::has_accent

```rust
pub fn has_accent(&self) -> bool
```

#### Reader::get_icon_url

```rust
pub fn get_icon_url(self) -> Result<Reader<'a>>
```

#### Reader::has_icon_url

```rust
pub fn has_icon_url(&self) -> bool
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

#### Builder::get_id

```rust
pub fn get_id(self) -> Result<Builder<'a>>
```

#### Builder::set_id

```rust
pub fn set_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_id

```rust
pub fn init_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_id

```rust
pub fn has_id(&self) -> bool
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

#### Builder::get_bg

```rust
pub fn get_bg(self) -> Result<Builder<'a>>
```

#### Builder::set_bg

```rust
pub fn set_bg(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_bg

```rust
pub fn init_bg(self, size: u32) -> Builder<'a>
```

#### Builder::has_bg

```rust
pub fn has_bg(&self) -> bool
```

#### Builder::get_fg

```rust
pub fn get_fg(self) -> Result<Builder<'a>>
```

#### Builder::set_fg

```rust
pub fn set_fg(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_fg

```rust
pub fn init_fg(self, size: u32) -> Builder<'a>
```

#### Builder::has_fg

```rust
pub fn has_fg(&self) -> bool
```

#### Builder::get_accent

```rust
pub fn get_accent(self) -> Result<Builder<'a>>
```

#### Builder::set_accent

```rust
pub fn set_accent(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_accent

```rust
pub fn init_accent(self, size: u32) -> Builder<'a>
```

#### Builder::has_accent

```rust
pub fn has_accent(&self) -> bool
```

#### Builder::get_icon_url

```rust
pub fn get_icon_url(self) -> Result<Builder<'a>>
```

#### Builder::set_icon_url

```rust
pub fn set_icon_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_icon_url

```rust
pub fn init_icon_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_icon_url

```rust
pub fn has_icon_url(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
