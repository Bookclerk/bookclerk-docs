# bookclerk_plugin_abi::plugin_capnp::catalog_hit

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

#### Reader::get_cover_url

```rust
pub fn get_cover_url(self) -> Result<Reader<'a>>
```

#### Reader::has_cover_url

```rust
pub fn has_cover_url(&self) -> bool
```

#### Reader::get_origin

```rust
pub fn get_origin(self) -> Result<Reader<'a>>
```

#### Reader::has_origin

```rust
pub fn has_origin(&self) -> bool
```

#### Reader::get_subtitle

```rust
pub fn get_subtitle(self) -> Result<Reader<'a>>
```

#### Reader::has_subtitle

```rust
pub fn has_subtitle(&self) -> bool
```

#### Reader::get_description

```rust
pub fn get_description(self) -> Result<Reader<'a>>
```

#### Reader::has_description

```rust
pub fn has_description(&self) -> bool
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

#### Reader::get_published_at

```rust
pub fn get_published_at(self) -> Result<Reader<'a>>
```

#### Reader::has_published_at

```rust
pub fn has_published_at(&self) -> bool
```

#### Reader::get_categories

```rust
pub fn get_categories(self) -> Result<Reader<'a>>
```

#### Reader::has_categories

```rust
pub fn has_categories(&self) -> bool
```

#### Reader::get_language

```rust
pub fn get_language(self) -> Result<Reader<'a>>
```

#### Reader::has_language

```rust
pub fn has_language(&self) -> bool
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

#### Reader::get_rating_overall

```rust
pub fn get_rating_overall(self) -> f64
```

#### Reader::get_rating_count

```rust
pub fn get_rating_count(self) -> i64
```

#### Reader::get_abridgement

```rust
pub fn get_abridgement(self) -> Result<Abridgement, NotInSchema>
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

#### Builder::get_cover_url

```rust
pub fn get_cover_url(self) -> Result<Builder<'a>>
```

#### Builder::set_cover_url

```rust
pub fn set_cover_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cover_url

```rust
pub fn init_cover_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_cover_url

```rust
pub fn has_cover_url(&self) -> bool
```

#### Builder::get_origin

```rust
pub fn get_origin(self) -> Result<Builder<'a>>
```

#### Builder::set_origin

```rust
pub fn set_origin(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_origin

```rust
pub fn init_origin(self, size: u32) -> Builder<'a>
```

#### Builder::has_origin

```rust
pub fn has_origin(&self) -> bool
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

#### Builder::get_description

```rust
pub fn get_description(self) -> Result<Builder<'a>>
```

#### Builder::set_description

```rust
pub fn set_description(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_description

```rust
pub fn init_description(self, size: u32) -> Builder<'a>
```

#### Builder::has_description

```rust
pub fn has_description(&self) -> bool
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

#### Builder::get_rating_overall

```rust
pub fn get_rating_overall(self) -> f64
```

#### Builder::set_rating_overall

```rust
pub fn set_rating_overall(&mut self, value: f64)
```

#### Builder::get_rating_count

```rust
pub fn get_rating_count(self) -> i64
```

#### Builder::set_rating_count

```rust
pub fn set_rating_count(&mut self, value: i64)
```

#### Builder::get_abridgement

```rust
pub fn get_abridgement(self) -> Result<Abridgement, NotInSchema>
```

#### Builder::set_abridgement

```rust
pub fn set_abridgement(&mut self, value: Abridgement)
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

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
