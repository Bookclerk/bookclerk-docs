# bookclerk_resolve::names

Splitting a store's display string of contributor names.

| Item | Kind | Summary |
| --- | --- | --- |
| [`split_display_names`](#split_display_names) | function | Splits a display string of contributor names into one name per entry. |
| [`NAME_SEPARATOR`](#name_separator) | constant | Separator between names in a flat display string, and between a name and its suffix (`Martin Luther King, Jr.`). |

## Functions

### split_display_names

```rust
pub fn split_display_names(display: &str) -> Vec<String>
```

Splits a display string of contributor names into one name per entry.

Splits on `,`, `;`, `&` and the word `and`, and keeps a name suffix such
as `Jr.` with the name before it. The caller decodes HTML entities first.

#### Arguments

* `display` - Names as one string (`"Jane Doe, John Roe & Ann Poe"`).

#### Returns

The trimmed, non-empty names in the order they appear.

## Constants

### NAME_SEPARATOR

```rust
pub const NAME_SEPARATOR: &str = ", ";
```

Separator between names in a flat display string, and between a name and
its suffix (`Martin Luther King, Jr.`).
