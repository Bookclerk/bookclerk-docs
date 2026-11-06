# bookclerk_plugin_abi::plugin_capnp::job_controller

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

#### Reader::get_invocation

```rust
pub fn get_invocation(self) -> Result<Reader<'a>>
```

#### Reader::has_invocation

```rust
pub fn has_invocation(&self) -> bool
```

#### Reader::get_input

```rust
pub fn get_input(self) -> Result<Client>
```

#### Reader::has_input

```rust
pub fn has_input(&self) -> bool
```

#### Reader::get_output

```rust
pub fn get_output(self) -> Result<Client>
```

#### Reader::has_output

```rust
pub fn has_output(&self) -> bool
```

#### Reader::get_progress

```rust
pub fn get_progress(self) -> Result<Client>
```

#### Reader::has_progress

```rust
pub fn has_progress(&self) -> bool
```

#### Reader::get_cancel

```rust
pub fn get_cancel(self) -> Result<Client>
```

#### Reader::has_cancel

```rust
pub fn has_cancel(&self) -> bool
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

#### Builder::get_invocation

```rust
pub fn get_invocation(self) -> Result<Builder<'a>>
```

#### Builder::set_invocation

```rust
pub fn set_invocation(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_invocation

```rust
pub fn init_invocation(self) -> Builder<'a>
```

#### Builder::has_invocation

```rust
pub fn has_invocation(&self) -> bool
```

#### Builder::get_input

```rust
pub fn get_input(self) -> Result<Client>
```

#### Builder::set_input

```rust
pub fn set_input(&mut self, value: Client)
```

#### Builder::has_input

```rust
pub fn has_input(&self) -> bool
```

#### Builder::get_output

```rust
pub fn get_output(self) -> Result<Client>
```

#### Builder::set_output

```rust
pub fn set_output(&mut self, value: Client)
```

#### Builder::has_output

```rust
pub fn has_output(&self) -> bool
```

#### Builder::get_progress

```rust
pub fn get_progress(self) -> Result<Client>
```

#### Builder::set_progress

```rust
pub fn set_progress(&mut self, value: Client)
```

#### Builder::has_progress

```rust
pub fn has_progress(&self) -> bool
```

#### Builder::get_cancel

```rust
pub fn get_cancel(self) -> Result<Client>
```

#### Builder::set_cancel

```rust
pub fn set_cancel(&mut self, value: Client)
```

#### Builder::has_cancel

```rust
pub fn has_cancel(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_invocation

```rust
pub fn get_invocation(&self) -> Pipeline
```

#### Pipeline::get_input

```rust
pub fn get_input(&self) -> Client
```

#### Pipeline::get_output

```rust
pub fn get_output(&self) -> Client
```

#### Pipeline::get_progress

```rust
pub fn get_progress(&self) -> Client
```

#### Pipeline::get_cancel

```rust
pub fn get_cancel(&self) -> Client
```

Implements: `FromTypelessPipeline`
