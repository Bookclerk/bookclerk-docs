# bookclerk_plugin_abi::plugin_capnp::purchase_hint

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

#### Reader::get_product_id

```rust
pub fn get_product_id(self) -> Result<Reader<'a>>
```

#### Reader::has_product_id

```rust
pub fn has_product_id(&self) -> bool
```

#### Reader::get_title

```rust
pub fn get_title(self) -> Result<Reader<'a>>
```

#### Reader::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Reader::get_url

```rust
pub fn get_url(self) -> Result<Reader<'a>>
```

#### Reader::has_url

```rust
pub fn has_url(&self) -> bool
```

#### Reader::get_price_cents

```rust
pub fn get_price_cents(self) -> i64
```

#### Reader::get_currency

```rust
pub fn get_currency(self) -> Result<Reader<'a>>
```

#### Reader::has_currency

```rust
pub fn has_currency(&self) -> bool
```

#### Reader::get_price_label

```rust
pub fn get_price_label(self) -> Result<Reader<'a>>
```

#### Reader::has_price_label

```rust
pub fn has_price_label(&self) -> bool
```

#### Reader::get_list_price_cents

```rust
pub fn get_list_price_cents(self) -> i64
```

#### Reader::get_list_price_label

```rust
pub fn get_list_price_label(self) -> Result<Reader<'a>>
```

#### Reader::has_list_price_label

```rust
pub fn has_list_price_label(&self) -> bool
```

#### Reader::get_member_price_cents

```rust
pub fn get_member_price_cents(self) -> i64
```

#### Reader::get_member_price_label

```rust
pub fn get_member_price_label(self) -> Result<Reader<'a>>
```

#### Reader::has_member_price_label

```rust
pub fn has_member_price_label(&self) -> bool
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

#### Builder::get_product_id

```rust
pub fn get_product_id(self) -> Result<Builder<'a>>
```

#### Builder::set_product_id

```rust
pub fn set_product_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_product_id

```rust
pub fn init_product_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_product_id

```rust
pub fn has_product_id(&self) -> bool
```

#### Builder::get_title

```rust
pub fn get_title(self) -> Result<Builder<'a>>
```

#### Builder::set_title

```rust
pub fn set_title(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_title

```rust
pub fn init_title(self, size: u32) -> Builder<'a>
```

#### Builder::has_title

```rust
pub fn has_title(&self) -> bool
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

#### Builder::get_price_cents

```rust
pub fn get_price_cents(self) -> i64
```

#### Builder::set_price_cents

```rust
pub fn set_price_cents(&mut self, value: i64)
```

#### Builder::get_currency

```rust
pub fn get_currency(self) -> Result<Builder<'a>>
```

#### Builder::set_currency

```rust
pub fn set_currency(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_currency

```rust
pub fn init_currency(self, size: u32) -> Builder<'a>
```

#### Builder::has_currency

```rust
pub fn has_currency(&self) -> bool
```

#### Builder::get_price_label

```rust
pub fn get_price_label(self) -> Result<Builder<'a>>
```

#### Builder::set_price_label

```rust
pub fn set_price_label(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_price_label

```rust
pub fn init_price_label(self, size: u32) -> Builder<'a>
```

#### Builder::has_price_label

```rust
pub fn has_price_label(&self) -> bool
```

#### Builder::get_list_price_cents

```rust
pub fn get_list_price_cents(self) -> i64
```

#### Builder::set_list_price_cents

```rust
pub fn set_list_price_cents(&mut self, value: i64)
```

#### Builder::get_list_price_label

```rust
pub fn get_list_price_label(self) -> Result<Builder<'a>>
```

#### Builder::set_list_price_label

```rust
pub fn set_list_price_label(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_list_price_label

```rust
pub fn init_list_price_label(self, size: u32) -> Builder<'a>
```

#### Builder::has_list_price_label

```rust
pub fn has_list_price_label(&self) -> bool
```

#### Builder::get_member_price_cents

```rust
pub fn get_member_price_cents(self) -> i64
```

#### Builder::set_member_price_cents

```rust
pub fn set_member_price_cents(&mut self, value: i64)
```

#### Builder::get_member_price_label

```rust
pub fn get_member_price_label(self) -> Result<Builder<'a>>
```

#### Builder::set_member_price_label

```rust
pub fn set_member_price_label(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_member_price_label

```rust
pub fn init_member_price_label(self, size: u32) -> Builder<'a>
```

#### Builder::has_member_price_label

```rust
pub fn has_member_price_label(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
