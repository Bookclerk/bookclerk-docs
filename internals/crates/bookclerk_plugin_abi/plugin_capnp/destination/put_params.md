# bookclerk_plugin_abi::plugin_capnp::destination::put_params

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

#### Reader::get_body

```rust
pub fn get_body(self) -> Result<Client>
```

#### Reader::has_body

```rust
pub fn has_body(&self) -> bool
```

#### Reader::get_options

```rust
pub fn get_options(self) -> Result<Reader<'a>>
```

#### Reader::has_options

```rust
pub fn has_options(&self) -> bool
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

#### Builder::get_body

```rust
pub fn get_body(self) -> Result<Client>
```

#### Builder::set_body

```rust
pub fn set_body(&mut self, value: Client)
```

#### Builder::has_body

```rust
pub fn has_body(&self) -> bool
```

#### Builder::get_options

```rust
pub fn get_options(self) -> Result<Builder<'a>>
```

#### Builder::set_options

```rust
pub fn set_options(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_options

```rust
pub fn init_options(self) -> Builder<'a>
```

#### Builder::has_options

```rust
pub fn has_options(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_body

```rust
pub fn get_body(&self) -> Client
```

#### Pipeline::get_options

```rust
pub fn get_options(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
