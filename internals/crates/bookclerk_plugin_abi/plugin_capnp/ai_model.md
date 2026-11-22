# bookclerk_plugin_abi::plugin_capnp::ai_model

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

#### Reader::get_model_id

```rust
pub fn get_model_id(self) -> Result<Reader<'a>>
```

#### Reader::has_model_id

```rust
pub fn has_model_id(&self) -> bool
```

#### Reader::get_task

```rust
pub fn get_task(self) -> Result<AiTask, NotInSchema>
```

#### Reader::get_revision

```rust
pub fn get_revision(self) -> Result<Reader<'a>>
```

#### Reader::has_revision

```rust
pub fn has_revision(&self) -> bool
```

#### Reader::get_dimensions

```rust
pub fn get_dimensions(self) -> u32
```

#### Reader::get_max_tokens

```rust
pub fn get_max_tokens(self) -> u32
```

#### Reader::get_languages

```rust
pub fn get_languages(self) -> Result<Reader<'a>>
```

#### Reader::has_languages

```rust
pub fn has_languages(&self) -> bool
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

#### Builder::get_model_id

```rust
pub fn get_model_id(self) -> Result<Builder<'a>>
```

#### Builder::set_model_id

```rust
pub fn set_model_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_model_id

```rust
pub fn init_model_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_model_id

```rust
pub fn has_model_id(&self) -> bool
```

#### Builder::get_task

```rust
pub fn get_task(self) -> Result<AiTask, NotInSchema>
```

#### Builder::set_task

```rust
pub fn set_task(&mut self, value: AiTask)
```

#### Builder::get_revision

```rust
pub fn get_revision(self) -> Result<Builder<'a>>
```

#### Builder::set_revision

```rust
pub fn set_revision(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_revision

```rust
pub fn init_revision(self, size: u32) -> Builder<'a>
```

#### Builder::has_revision

```rust
pub fn has_revision(&self) -> bool
```

#### Builder::get_dimensions

```rust
pub fn get_dimensions(self) -> u32
```

#### Builder::set_dimensions

```rust
pub fn set_dimensions(&mut self, value: u32)
```

#### Builder::get_max_tokens

```rust
pub fn get_max_tokens(self) -> u32
```

#### Builder::set_max_tokens

```rust
pub fn set_max_tokens(&mut self, value: u32)
```

#### Builder::get_languages

```rust
pub fn get_languages(self) -> Result<Builder<'a>>
```

#### Builder::set_languages

```rust
pub fn set_languages(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_languages

```rust
pub fn init_languages(self, size: u32) -> Builder<'a>
```

#### Builder::has_languages

```rust
pub fn has_languages(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
