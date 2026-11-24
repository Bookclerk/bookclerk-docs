# bookclerk_plugin_abi::plugin_capnp::notify_deliverability_advice

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

#### Reader::get_outcome

```rust
pub fn get_outcome(self) -> Result<NotifyCheckOutcome, NotInSchema>
```

#### Reader::get_title

```rust
pub fn get_title(self) -> Result<Reader<'a>>
```

#### Reader::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Reader::get_detail

```rust
pub fn get_detail(self) -> Result<Reader<'a>>
```

#### Reader::has_detail

```rust
pub fn has_detail(&self) -> bool
```

#### Reader::get_sources

```rust
pub fn get_sources(self) -> Result<Reader<'a>>
```

#### Reader::has_sources

```rust
pub fn has_sources(&self) -> bool
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

#### Builder::get_outcome

```rust
pub fn get_outcome(self) -> Result<NotifyCheckOutcome, NotInSchema>
```

#### Builder::set_outcome

```rust
pub fn set_outcome(&mut self, value: NotifyCheckOutcome)
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

#### Builder::get_detail

```rust
pub fn get_detail(self) -> Result<Builder<'a>>
```

#### Builder::set_detail

```rust
pub fn set_detail(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_detail

```rust
pub fn init_detail(self, size: u32) -> Builder<'a>
```

#### Builder::has_detail

```rust
pub fn has_detail(&self) -> bool
```

#### Builder::get_sources

```rust
pub fn get_sources(self) -> Result<Builder<'a>>
```

#### Builder::set_sources

```rust
pub fn set_sources(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_sources

```rust
pub fn init_sources(self, size: u32) -> Builder<'a>
```

#### Builder::has_sources

```rust
pub fn has_sources(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
