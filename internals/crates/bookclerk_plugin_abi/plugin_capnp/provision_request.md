# bookclerk_plugin_abi::plugin_capnp::provision_request

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

#### Reader::get_grant

```rust
pub fn get_grant(self) -> Result<Reader<'a>>
```

#### Reader::has_grant

```rust
pub fn has_grant(&self) -> bool
```

#### Reader::get_inputs_json

```rust
pub fn get_inputs_json(self) -> Result<Reader<'a>>
```

#### Reader::has_inputs_json

```rust
pub fn has_inputs_json(&self) -> bool
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

#### Builder::get_grant

```rust
pub fn get_grant(self) -> Result<Builder<'a>>
```

#### Builder::set_grant

```rust
pub fn set_grant(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_grant

```rust
pub fn init_grant(self) -> Builder<'a>
```

#### Builder::has_grant

```rust
pub fn has_grant(&self) -> bool
```

#### Builder::get_inputs_json

```rust
pub fn get_inputs_json(self) -> Result<Builder<'a>>
```

#### Builder::set_inputs_json

```rust
pub fn set_inputs_json(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_inputs_json

```rust
pub fn init_inputs_json(self, size: u32) -> Builder<'a>
```

#### Builder::has_inputs_json

```rust
pub fn has_inputs_json(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_grant

```rust
pub fn get_grant(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
