# bookclerk_plugin_abi::plugin_capnp::notify_deliverability_check

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

#### Reader::get_service

```rust
pub fn get_service(self) -> Result<Reader<'a>>
```

#### Reader::has_service

```rust
pub fn has_service(&self) -> bool
```

#### Reader::get_outcome

```rust
pub fn get_outcome(self) -> Result<NotifyCheckOutcome, NotInSchema>
```

#### Reader::get_summary

```rust
pub fn get_summary(self) -> Result<Reader<'a>>
```

#### Reader::has_summary

```rust
pub fn has_summary(&self) -> bool
```

#### Reader::get_report_url

```rust
pub fn get_report_url(self) -> Result<Reader<'a>>
```

#### Reader::has_report_url

```rust
pub fn has_report_url(&self) -> bool
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

#### Builder::get_service

```rust
pub fn get_service(self) -> Result<Builder<'a>>
```

#### Builder::set_service

```rust
pub fn set_service(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_service

```rust
pub fn init_service(self, size: u32) -> Builder<'a>
```

#### Builder::has_service

```rust
pub fn has_service(&self) -> bool
```

#### Builder::get_outcome

```rust
pub fn get_outcome(self) -> Result<NotifyCheckOutcome, NotInSchema>
```

#### Builder::set_outcome

```rust
pub fn set_outcome(&mut self, value: NotifyCheckOutcome)
```

#### Builder::get_summary

```rust
pub fn get_summary(self) -> Result<Builder<'a>>
```

#### Builder::set_summary

```rust
pub fn set_summary(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_summary

```rust
pub fn init_summary(self, size: u32) -> Builder<'a>
```

#### Builder::has_summary

```rust
pub fn has_summary(&self) -> bool
```

#### Builder::get_report_url

```rust
pub fn get_report_url(self) -> Result<Builder<'a>>
```

#### Builder::set_report_url

```rust
pub fn set_report_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_report_url

```rust
pub fn init_report_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_report_url

```rust
pub fn has_report_url(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
