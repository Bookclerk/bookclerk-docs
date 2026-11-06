# bookclerk_plugin_abi::plugin_capnp::external_user

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

#### Reader::get_provider

```rust
pub fn get_provider(self) -> Result<Reader<'a>>
```

#### Reader::has_provider

```rust
pub fn has_provider(&self) -> bool
```

#### Reader::get_external_user_id

```rust
pub fn get_external_user_id(self) -> Result<Reader<'a>>
```

#### Reader::has_external_user_id

```rust
pub fn has_external_user_id(&self) -> bool
```

#### Reader::get_display_name

```rust
pub fn get_display_name(self) -> Result<Reader<'a>>
```

#### Reader::has_display_name

```rust
pub fn has_display_name(&self) -> bool
```

#### Reader::get_access_token

```rust
pub fn get_access_token(self) -> Result<Reader<'a>>
```

#### Reader::has_access_token

```rust
pub fn has_access_token(&self) -> bool
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

#### Builder::get_provider

```rust
pub fn get_provider(self) -> Result<Builder<'a>>
```

#### Builder::set_provider

```rust
pub fn set_provider(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_provider

```rust
pub fn init_provider(self, size: u32) -> Builder<'a>
```

#### Builder::has_provider

```rust
pub fn has_provider(&self) -> bool
```

#### Builder::get_external_user_id

```rust
pub fn get_external_user_id(self) -> Result<Builder<'a>>
```

#### Builder::set_external_user_id

```rust
pub fn set_external_user_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_external_user_id

```rust
pub fn init_external_user_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_external_user_id

```rust
pub fn has_external_user_id(&self) -> bool
```

#### Builder::get_display_name

```rust
pub fn get_display_name(self) -> Result<Builder<'a>>
```

#### Builder::set_display_name

```rust
pub fn set_display_name(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_display_name

```rust
pub fn init_display_name(self, size: u32) -> Builder<'a>
```

#### Builder::has_display_name

```rust
pub fn has_display_name(&self) -> bool
```

#### Builder::get_access_token

```rust
pub fn get_access_token(self) -> Result<Builder<'a>>
```

#### Builder::set_access_token

```rust
pub fn set_access_token(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_access_token

```rust
pub fn init_access_token(self, size: u32) -> Builder<'a>
```

#### Builder::has_access_token

```rust
pub fn has_access_token(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
