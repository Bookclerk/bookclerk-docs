# bookclerk_plugin_abi::plugin_capnp::write_options

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

#### Reader::get_content_type

```rust
pub fn get_content_type(self) -> Result<Reader<'a>>
```

#### Reader::has_content_type

```rust
pub fn has_content_type(&self) -> bool
```

#### Reader::get_content_length

```rust
pub fn get_content_length(self) -> u64
```

#### Reader::get_sha256

```rust
pub fn get_sha256(self) -> Result<Reader<'a>>
```

#### Reader::has_sha256

```rust
pub fn has_sha256(&self) -> bool
```

#### Reader::get_commit_token

```rust
pub fn get_commit_token(self) -> Result<Reader<'a>>
```

#### Reader::has_commit_token

```rust
pub fn has_commit_token(&self) -> bool
```

#### Reader::get_stage_only

```rust
pub fn get_stage_only(self) -> bool
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

#### Builder::get_content_type

```rust
pub fn get_content_type(self) -> Result<Builder<'a>>
```

#### Builder::set_content_type

```rust
pub fn set_content_type(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_content_type

```rust
pub fn init_content_type(self, size: u32) -> Builder<'a>
```

#### Builder::has_content_type

```rust
pub fn has_content_type(&self) -> bool
```

#### Builder::get_content_length

```rust
pub fn get_content_length(self) -> u64
```

#### Builder::set_content_length

```rust
pub fn set_content_length(&mut self, value: u64)
```

#### Builder::get_sha256

```rust
pub fn get_sha256(self) -> Result<Builder<'a>>
```

#### Builder::set_sha256

```rust
pub fn set_sha256(&mut self, value: Reader<'_>)
```

#### Builder::init_sha256

```rust
pub fn init_sha256(self, size: u32) -> Builder<'a>
```

#### Builder::has_sha256

```rust
pub fn has_sha256(&self) -> bool
```

#### Builder::get_commit_token

```rust
pub fn get_commit_token(self) -> Result<Builder<'a>>
```

#### Builder::set_commit_token

```rust
pub fn set_commit_token(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_commit_token

```rust
pub fn init_commit_token(self, size: u32) -> Builder<'a>
```

#### Builder::has_commit_token

```rust
pub fn has_commit_token(&self) -> bool
```

#### Builder::get_stage_only

```rust
pub fn get_stage_only(self) -> bool
```

#### Builder::set_stage_only

```rust
pub fn set_stage_only(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
