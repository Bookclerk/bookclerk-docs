# bookclerk_plugin_abi::plugin_capnp::origin_response

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

#### Reader::get_status

```rust
pub fn get_status(self) -> u16
```

#### Reader::get_headers

```rust
pub fn get_headers(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_headers

```rust
pub fn has_headers(&self) -> bool
```

#### Reader::get_body

```rust
pub fn get_body(self) -> Result<Client>
```

#### Reader::has_body

```rust
pub fn has_body(&self) -> bool
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

#### Builder::get_status

```rust
pub fn get_status(self) -> u16
```

#### Builder::set_status

```rust
pub fn set_status(&mut self, value: u16)
```

#### Builder::get_headers

```rust
pub fn get_headers(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_headers

```rust
pub fn set_headers(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_headers

```rust
pub fn init_headers(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_headers

```rust
pub fn has_headers(&self) -> bool
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

Implements: `FromTypelessPipeline`
