# bookclerk_plugin_abi::plugin_capnp::notification_message

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

#### Reader::get_notification_id

```rust
pub fn get_notification_id(self) -> Result<Reader<'a>>
```

#### Reader::has_notification_id

```rust
pub fn has_notification_id(&self) -> bool
```

#### Reader::get_notification_type

```rust
pub fn get_notification_type(self) -> Result<Reader<'a>>
```

#### Reader::has_notification_type

```rust
pub fn has_notification_type(&self) -> bool
```

#### Reader::get_severity

```rust
pub fn get_severity(self) -> Result<NotificationSeverity, NotInSchema>
```

#### Reader::get_title

```rust
pub fn get_title(self) -> Result<Reader<'a>>
```

#### Reader::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Reader::get_text

```rust
pub fn get_text(self) -> Result<Reader<'a>>
```

#### Reader::has_text

```rust
pub fn has_text(&self) -> bool
```

#### Reader::get_markdown

```rust
pub fn get_markdown(self) -> Result<Reader<'a>>
```

#### Reader::has_markdown

```rust
pub fn has_markdown(&self) -> bool
```

#### Reader::get_html

```rust
pub fn get_html(self) -> Result<Reader<'a>>
```

#### Reader::has_html

```rust
pub fn has_html(&self) -> bool
```

#### Reader::get_link

```rust
pub fn get_link(self) -> Result<Reader<'a>>
```

#### Reader::has_link

```rust
pub fn has_link(&self) -> bool
```

#### Reader::get_fields

```rust
pub fn get_fields(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_fields

```rust
pub fn has_fields(&self) -> bool
```

#### Reader::get_group_key

```rust
pub fn get_group_key(self) -> Result<Reader<'a>>
```

#### Reader::has_group_key

```rust
pub fn has_group_key(&self) -> bool
```

#### Reader::get_alert_action

```rust
pub fn get_alert_action(self) -> Result<NotificationAlertAction, NotInSchema>
```

#### Reader::get_alert_key

```rust
pub fn get_alert_key(self) -> Result<Reader<'a>>
```

#### Reader::has_alert_key

```rust
pub fn has_alert_key(&self) -> bool
```

#### Reader::get_sensitive

```rust
pub fn get_sensitive(self) -> bool
```

#### Reader::get_library_origin

```rust
pub fn get_library_origin(self) -> Result<Reader<'a>>
```

#### Reader::has_library_origin

```rust
pub fn has_library_origin(&self) -> bool
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

#### Builder::get_notification_id

```rust
pub fn get_notification_id(self) -> Result<Builder<'a>>
```

#### Builder::set_notification_id

```rust
pub fn set_notification_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_notification_id

```rust
pub fn init_notification_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_notification_id

```rust
pub fn has_notification_id(&self) -> bool
```

#### Builder::get_notification_type

```rust
pub fn get_notification_type(self) -> Result<Builder<'a>>
```

#### Builder::set_notification_type

```rust
pub fn set_notification_type(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_notification_type

```rust
pub fn init_notification_type(self, size: u32) -> Builder<'a>
```

#### Builder::has_notification_type

```rust
pub fn has_notification_type(&self) -> bool
```

#### Builder::get_severity

```rust
pub fn get_severity(self) -> Result<NotificationSeverity, NotInSchema>
```

#### Builder::set_severity

```rust
pub fn set_severity(&mut self, value: NotificationSeverity)
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

#### Builder::get_text

```rust
pub fn get_text(self) -> Result<Builder<'a>>
```

#### Builder::set_text

```rust
pub fn set_text(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_text

```rust
pub fn init_text(self, size: u32) -> Builder<'a>
```

#### Builder::has_text

```rust
pub fn has_text(&self) -> bool
```

#### Builder::get_markdown

```rust
pub fn get_markdown(self) -> Result<Builder<'a>>
```

#### Builder::set_markdown

```rust
pub fn set_markdown(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_markdown

```rust
pub fn init_markdown(self, size: u32) -> Builder<'a>
```

#### Builder::has_markdown

```rust
pub fn has_markdown(&self) -> bool
```

#### Builder::get_html

```rust
pub fn get_html(self) -> Result<Builder<'a>>
```

#### Builder::set_html

```rust
pub fn set_html(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_html

```rust
pub fn init_html(self, size: u32) -> Builder<'a>
```

#### Builder::has_html

```rust
pub fn has_html(&self) -> bool
```

#### Builder::get_link

```rust
pub fn get_link(self) -> Result<Builder<'a>>
```

#### Builder::set_link

```rust
pub fn set_link(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_link

```rust
pub fn init_link(self, size: u32) -> Builder<'a>
```

#### Builder::has_link

```rust
pub fn has_link(&self) -> bool
```

#### Builder::get_fields

```rust
pub fn get_fields(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_fields

```rust
pub fn set_fields(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_fields

```rust
pub fn init_fields(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_fields

```rust
pub fn has_fields(&self) -> bool
```

#### Builder::get_group_key

```rust
pub fn get_group_key(self) -> Result<Builder<'a>>
```

#### Builder::set_group_key

```rust
pub fn set_group_key(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_group_key

```rust
pub fn init_group_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_group_key

```rust
pub fn has_group_key(&self) -> bool
```

#### Builder::get_alert_action

```rust
pub fn get_alert_action(self) -> Result<NotificationAlertAction, NotInSchema>
```

#### Builder::set_alert_action

```rust
pub fn set_alert_action(&mut self, value: NotificationAlertAction)
```

#### Builder::get_alert_key

```rust
pub fn get_alert_key(self) -> Result<Builder<'a>>
```

#### Builder::set_alert_key

```rust
pub fn set_alert_key(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_alert_key

```rust
pub fn init_alert_key(self, size: u32) -> Builder<'a>
```

#### Builder::has_alert_key

```rust
pub fn has_alert_key(&self) -> bool
```

#### Builder::get_sensitive

```rust
pub fn get_sensitive(self) -> bool
```

#### Builder::set_sensitive

```rust
pub fn set_sensitive(&mut self, value: bool)
```

#### Builder::get_library_origin

```rust
pub fn get_library_origin(self) -> Result<Builder<'a>>
```

#### Builder::set_library_origin

```rust
pub fn set_library_origin(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_library_origin

```rust
pub fn init_library_origin(self, size: u32) -> Builder<'a>
```

#### Builder::has_library_origin

```rust
pub fn has_library_origin(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
