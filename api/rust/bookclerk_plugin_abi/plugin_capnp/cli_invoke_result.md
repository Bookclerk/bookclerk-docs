# bookclerk_plugin_abi::plugin_capnp::cli_invoke_result

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

#### Reader::get_exit_code

```rust
pub fn get_exit_code(self) -> i32
```

#### Reader::get_stdout

```rust
pub fn get_stdout(self) -> Result<Reader<'a>>
```

#### Reader::has_stdout

```rust
pub fn has_stdout(&self) -> bool
```

#### Reader::get_stderr

```rust
pub fn get_stderr(self) -> Result<Reader<'a>>
```

#### Reader::has_stderr

```rust
pub fn has_stderr(&self) -> bool
```

#### Reader::get_payload

```rust
pub fn get_payload(self) -> Result<Reader<'a>>
```

#### Reader::has_payload

```rust
pub fn has_payload(&self) -> bool
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

#### Builder::get_exit_code

```rust
pub fn get_exit_code(self) -> i32
```

#### Builder::set_exit_code

```rust
pub fn set_exit_code(&mut self, value: i32)
```

#### Builder::get_stdout

```rust
pub fn get_stdout(self) -> Result<Builder<'a>>
```

#### Builder::set_stdout

```rust
pub fn set_stdout(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_stdout

```rust
pub fn init_stdout(self, size: u32) -> Builder<'a>
```

#### Builder::has_stdout

```rust
pub fn has_stdout(&self) -> bool
```

#### Builder::get_stderr

```rust
pub fn get_stderr(self) -> Result<Builder<'a>>
```

#### Builder::set_stderr

```rust
pub fn set_stderr(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_stderr

```rust
pub fn init_stderr(self, size: u32) -> Builder<'a>
```

#### Builder::has_stderr

```rust
pub fn has_stderr(&self) -> bool
```

#### Builder::get_payload

```rust
pub fn get_payload(self) -> Result<Builder<'a>>
```

#### Builder::set_payload

```rust
pub fn set_payload(&mut self, value: Reader<'_>) -> Result<()>
```

#### Builder::init_payload

```rust
pub fn init_payload(self) -> Builder<'a>
```

#### Builder::has_payload

```rust
pub fn has_payload(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_payload

```rust
pub fn get_payload(&self) -> Pipeline
```

Implements: `FromTypelessPipeline`
