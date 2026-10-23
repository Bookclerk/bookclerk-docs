# bookclerk_plugin_abi::plugin_capnp::scan_summary

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

#### Reader::get_accounts

```rust
pub fn get_accounts(self) -> u32
```

#### Reader::get_books_upserted

```rust
pub fn get_books_upserted(self) -> u32
```

#### Reader::get_pages

```rust
pub fn get_pages(self) -> u32
```

#### Reader::get_skipped_disabled

```rust
pub fn get_skipped_disabled(self) -> u32
```

#### Reader::get_books

```rust
pub fn get_books(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_books

```rust
pub fn has_books(&self) -> bool
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

#### Builder::get_accounts

```rust
pub fn get_accounts(self) -> u32
```

#### Builder::set_accounts

```rust
pub fn set_accounts(&mut self, value: u32)
```

#### Builder::get_books_upserted

```rust
pub fn get_books_upserted(self) -> u32
```

#### Builder::set_books_upserted

```rust
pub fn set_books_upserted(&mut self, value: u32)
```

#### Builder::get_pages

```rust
pub fn get_pages(self) -> u32
```

#### Builder::set_pages

```rust
pub fn set_pages(&mut self, value: u32)
```

#### Builder::get_skipped_disabled

```rust
pub fn get_skipped_disabled(self) -> u32
```

#### Builder::set_skipped_disabled

```rust
pub fn set_skipped_disabled(&mut self, value: u32)
```

#### Builder::get_books

```rust
pub fn get_books(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_books

```rust
pub fn set_books(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_books

```rust
pub fn init_books(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_books

```rust
pub fn has_books(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
