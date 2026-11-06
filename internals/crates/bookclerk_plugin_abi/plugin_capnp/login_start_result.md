# bookclerk_plugin_abi::plugin_capnp::login_start_result

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

#### Reader::get_session_id

```rust
pub fn get_session_id(self) -> Result<Reader<'a>>
```

#### Reader::has_session_id

```rust
pub fn has_session_id(&self) -> bool
```

#### Reader::get_url

```rust
pub fn get_url(self) -> Result<Reader<'a>>
```

#### Reader::has_url

```rust
pub fn has_url(&self) -> bool
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

#### Builder::get_session_id

```rust
pub fn get_session_id(self) -> Result<Builder<'a>>
```

#### Builder::set_session_id

```rust
pub fn set_session_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_session_id

```rust
pub fn init_session_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_session_id

```rust
pub fn has_session_id(&self) -> bool
```

#### Builder::get_url

```rust
pub fn get_url(self) -> Result<Builder<'a>>
```

#### Builder::set_url

```rust
pub fn set_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_url

```rust
pub fn init_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_url

```rust
pub fn has_url(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
