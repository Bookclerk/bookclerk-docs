# bookclerk_plugin_abi::plugin_capnp::authenticate_user_params

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

#### Reader::get_username

```rust
pub fn get_username(self) -> Result<Reader<'a>>
```

#### Reader::has_username

```rust
pub fn has_username(&self) -> bool
```

#### Reader::get_password

```rust
pub fn get_password(self) -> Result<Reader<'a>>
```

#### Reader::has_password

```rust
pub fn has_password(&self) -> bool
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

#### Builder::get_username

```rust
pub fn get_username(self) -> Result<Builder<'a>>
```

#### Builder::set_username

```rust
pub fn set_username(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_username

```rust
pub fn init_username(self, size: u32) -> Builder<'a>
```

#### Builder::has_username

```rust
pub fn has_username(&self) -> bool
```

#### Builder::get_password

```rust
pub fn get_password(self) -> Result<Builder<'a>>
```

#### Builder::set_password

```rust
pub fn set_password(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_password

```rust
pub fn init_password(self, size: u32) -> Builder<'a>
```

#### Builder::has_password

```rust
pub fn has_password(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
