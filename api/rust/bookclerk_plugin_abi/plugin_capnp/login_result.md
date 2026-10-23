# bookclerk_plugin_abi::plugin_capnp::login_result

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

#### Reader::get_account

```rust
pub fn get_account(self) -> Result<Reader<'a>>
```

#### Reader::has_account

```rust
pub fn has_account(&self) -> bool
```

#### Reader::get_credentials

```rust
pub fn get_credentials(self) -> Result<Reader<'a>>
```

#### Reader::has_credentials

```rust
pub fn has_credentials(&self) -> bool
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

#### Builder::get_account

```rust
pub fn get_account(self) -> Result<Builder<'a>>
```

#### Builder::set_account

```rust
pub fn set_account(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_account

```rust
pub fn init_account(self) -> Builder<'a>
```

#### Builder::has_account

```rust
pub fn has_account(&self) -> bool
```

#### Builder::get_credentials

```rust
pub fn get_credentials(self) -> Result<Builder<'a>>
```

#### Builder::set_credentials

```rust
pub fn set_credentials(&mut self, value: Reader<'_>)
```

#### Builder::init_credentials

```rust
pub fn init_credentials(self, size: u32) -> Builder<'a>
```

#### Builder::has_credentials

```rust
pub fn has_credentials(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_account

```rust
pub fn get_account(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
