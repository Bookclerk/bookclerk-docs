# bookclerk_plugin_abi::plugin_capnp::destination::abort_stage_params

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

#### Reader::get_key

```rust
pub fn get_key(self) -> Result<Reader<'a>>
```

#### Reader::has_key

```rust
pub fn has_key(&self) -> bool
```

#### Reader::get_commit_token

```rust
pub fn get_commit_token(self) -> Result<Reader<'a>>
```

#### Reader::has_commit_token

```rust
pub fn has_commit_token(&self) -> bool
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

#### Builder::get_key

```rust
pub fn get_key(self) -> Result<Builder<'a>>
```

#### Builder::set_key

```rust
pub fn set_key(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_key

```rust
pub fn init_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_key

```rust
pub fn has_key(&self) -> bool
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

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
