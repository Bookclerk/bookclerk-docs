# bookclerk_plugin_abi::plugin_capnp::scan_book

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

#### Reader::get_account_id

```rust
pub fn get_account_id(self) -> Result<Reader<'a>>
```

#### Reader::has_account_id

```rust
pub fn has_account_id(&self) -> bool
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

#### Reader::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Reader<'a>>
```

#### Reader::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
```

#### Reader::get_publisher

```rust
pub fn get_publisher(self) -> Result<Reader<'a>>
```

#### Reader::has_publisher

```rust
pub fn has_publisher(&self) -> bool
```

#### Reader::get_length_minutes

```rust
pub fn get_length_minutes(self) -> i64
```

#### Reader::get_subtitle

```rust
pub fn get_subtitle(self) -> Result<Reader<'a>>
```

#### Reader::has_subtitle

```rust
pub fn has_subtitle(&self) -> bool
```

#### Reader::get_work_kind

```rust
pub fn get_work_kind(self) -> Result<WorkKind, NotInSchema>
```

#### Reader::get_expression_kind

```rust
pub fn get_expression_kind(self) -> Result<ExpressionKind, NotInSchema>
```

#### Reader::get_media_kind

```rust
pub fn get_media_kind(self) -> Result<MediaKind, NotInSchema>
```

#### Reader::get_identifiers

```rust
pub fn get_identifiers(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_identifiers

```rust
pub fn has_identifiers(&self) -> bool
```

#### Reader::get_credits

```rust
pub fn get_credits(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_credits

```rust
pub fn has_credits(&self) -> bool
```

#### Reader::get_series_entries

```rust
pub fn get_series_entries(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_series_entries

```rust
pub fn has_series_entries(&self) -> bool
```

#### Reader::get_parent_product_id

```rust
pub fn get_parent_product_id(self) -> Result<Reader<'a>>
```

#### Reader::has_parent_product_id

```rust
pub fn has_parent_product_id(&self) -> bool
```

#### Reader::get_language

```rust
pub fn get_language(self) -> Result<Reader<'a>>
```

#### Reader::has_language

```rust
pub fn has_language(&self) -> bool
```

#### Reader::get_abridgement

```rust
pub fn get_abridgement(self) -> Result<Abridgement, NotInSchema>
```

#### Reader::get_published_at

```rust
pub fn get_published_at(self) -> Result<Reader<'a>>
```

#### Reader::has_published_at

```rust
pub fn has_published_at(&self) -> bool
```

#### Reader::get_purchased_at

```rust
pub fn get_purchased_at(self) -> Result<Reader<'a>>
```

#### Reader::has_purchased_at

```rust
pub fn has_purchased_at(&self) -> bool
```

#### Reader::get_categories

```rust
pub fn get_categories(self) -> Result<Reader<'a>>
```

#### Reader::has_categories

```rust
pub fn has_categories(&self) -> bool
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

#### Builder::get_account_id

```rust
pub fn get_account_id(self) -> Result<Builder<'a>>
```

#### Builder::set_account_id

```rust
pub fn set_account_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_account_id

```rust
pub fn init_account_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_account_id

```rust
pub fn has_account_id(&self) -> bool
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

#### Builder::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Builder<'a>>
```

#### Builder::set_marketplace

```rust
pub fn set_marketplace(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_marketplace

```rust
pub fn init_marketplace(self, size: u32) -> Builder<'a>
```

#### Builder::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
```

#### Builder::get_publisher

```rust
pub fn get_publisher(self) -> Result<Builder<'a>>
```

#### Builder::set_publisher

```rust
pub fn set_publisher(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_publisher

```rust
pub fn init_publisher(self, size: u32) -> Builder<'a>
```

#### Builder::has_publisher

```rust
pub fn has_publisher(&self) -> bool
```

#### Builder::get_length_minutes

```rust
pub fn get_length_minutes(self) -> i64
```

#### Builder::set_length_minutes

```rust
pub fn set_length_minutes(&mut self, value: i64)
```

#### Builder::get_subtitle

```rust
pub fn get_subtitle(self) -> Result<Builder<'a>>
```

#### Builder::set_subtitle

```rust
pub fn set_subtitle(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_subtitle

```rust
pub fn init_subtitle(self, size: u32) -> Builder<'a>
```

#### Builder::has_subtitle

```rust
pub fn has_subtitle(&self) -> bool
```

#### Builder::get_work_kind

```rust
pub fn get_work_kind(self) -> Result<WorkKind, NotInSchema>
```

#### Builder::set_work_kind

```rust
pub fn set_work_kind(&mut self, value: WorkKind)
```

#### Builder::get_expression_kind

```rust
pub fn get_expression_kind(self) -> Result<ExpressionKind, NotInSchema>
```

#### Builder::set_expression_kind

```rust
pub fn set_expression_kind(&mut self, value: ExpressionKind)
```

#### Builder::get_media_kind

```rust
pub fn get_media_kind(self) -> Result<MediaKind, NotInSchema>
```

#### Builder::set_media_kind

```rust
pub fn set_media_kind(&mut self, value: MediaKind)
```

#### Builder::get_identifiers

```rust
pub fn get_identifiers(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_identifiers

```rust
pub fn set_identifiers(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_identifiers

```rust
pub fn init_identifiers(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_identifiers

```rust
pub fn has_identifiers(&self) -> bool
```

#### Builder::get_credits

```rust
pub fn get_credits(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_credits

```rust
pub fn set_credits(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_credits

```rust
pub fn init_credits(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_credits

```rust
pub fn has_credits(&self) -> bool
```

#### Builder::get_series_entries

```rust
pub fn get_series_entries(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_series_entries

```rust
pub fn set_series_entries(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_series_entries

```rust
pub fn init_series_entries(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_series_entries

```rust
pub fn has_series_entries(&self) -> bool
```

#### Builder::get_parent_product_id

```rust
pub fn get_parent_product_id(self) -> Result<Builder<'a>>
```

#### Builder::set_parent_product_id

```rust
pub fn set_parent_product_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_parent_product_id

```rust
pub fn init_parent_product_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_parent_product_id

```rust
pub fn has_parent_product_id(&self) -> bool
```

#### Builder::get_language

```rust
pub fn get_language(self) -> Result<Builder<'a>>
```

#### Builder::set_language

```rust
pub fn set_language(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_language

```rust
pub fn init_language(self, size: u32) -> Builder<'a>
```

#### Builder::has_language

```rust
pub fn has_language(&self) -> bool
```

#### Builder::get_abridgement

```rust
pub fn get_abridgement(self) -> Result<Abridgement, NotInSchema>
```

#### Builder::set_abridgement

```rust
pub fn set_abridgement(&mut self, value: Abridgement)
```

#### Builder::get_published_at

```rust
pub fn get_published_at(self) -> Result<Builder<'a>>
```

#### Builder::set_published_at

```rust
pub fn set_published_at(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_published_at

```rust
pub fn init_published_at(self, size: u32) -> Builder<'a>
```

#### Builder::has_published_at

```rust
pub fn has_published_at(&self) -> bool
```

#### Builder::get_purchased_at

```rust
pub fn get_purchased_at(self) -> Result<Builder<'a>>
```

#### Builder::set_purchased_at

```rust
pub fn set_purchased_at(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_purchased_at

```rust
pub fn init_purchased_at(self, size: u32) -> Builder<'a>
```

#### Builder::has_purchased_at

```rust
pub fn has_purchased_at(&self) -> bool
```

#### Builder::get_categories

```rust
pub fn get_categories(self) -> Result<Builder<'a>>
```

#### Builder::set_categories

```rust
pub fn set_categories(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_categories

```rust
pub fn init_categories(self, size: u32) -> Builder<'a>
```

#### Builder::has_categories

```rust
pub fn has_categories(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
