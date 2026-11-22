# bookclerk_plugin_abi::plugin_capnp::ai_embed_text

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

#### Reader::get_text

```rust
pub fn get_text(self) -> Result<Reader<'a>>
```

#### Reader::has_text

```rust
pub fn has_text(&self) -> bool
```

#### Reader::get_role

```rust
pub fn get_role(self) -> Result<AiTextRole, NotInSchema>
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

#### Builder::get_text

```rust
pub fn get_text(self) -> Result<Builder<'a>>
```

#### Builder::set_text

```rust
pub fn set_text(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_text

```rust
pub fn init_text(self, size: u32) -> Builder<'a>
```

#### Builder::has_text

```rust
pub fn has_text(&self) -> bool
```

#### Builder::get_role

```rust
pub fn get_role(self) -> Result<AiTextRole, NotInSchema>
```

#### Builder::set_role

```rust
pub fn set_role(&mut self, value: AiTextRole)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
