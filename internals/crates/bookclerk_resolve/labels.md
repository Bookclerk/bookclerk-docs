# bookclerk_resolve::labels

Store labels that name the same thing in different spellings.

Stores label one series and one recording differently: Audible files the
abridged First Family under "King and Maxwell (abridged)", Chirp under
"King & Maxwell" and Libro.fm under "King & Maxwell Series", and Libro.fm
titles that recording "First Family - Abridged". These helpers remove the
edition markers and generic words a store adds, so one series is one
series row and one book matches its own work, whichever store reported it.

| Item | Kind | Summary |
| --- | --- | --- |
| [`strip_edition_marker`](#strip_edition_marker) | function | A title or series name without the edition markers a store appended. |
| [`clean_series_name`](#clean_series_name) | function | A series name as Bookclerk shows and stores it. |
| [`series_key`](#series_key) | function | The key series names are matched by. |

## Functions

### strip_edition_marker

```rust
pub fn strip_edition_marker(text: &str) -> &str
```

A title or series name without the edition markers a store appended.

Removes trailing `(Abridged)`, `[Unabridged]`, ` - Abridged`,
`: Unabridged Edition` and the like, so `First Family - Abridged` and
`First Family (Abridged)` both read `First Family`. A marker inside the
text, or one that is the whole text, is kept.

#### Arguments

* `text` - A display title or series name.

#### Returns

The text without trailing edition markers, trimmed.

### clean_series_name

```rust
pub fn clean_series_name(name: &str) -> String
```

A series name as Bookclerk shows and stores it.

Collapses whitespace, removes a trailing edition marker
(`King and Maxwell (abridged)` reads `King and Maxwell`) and a trailing
generic word (`King & Maxwell Series` reads `King & Maxwell`). The word
stays when only an article would be left (`The Series`). `&` and `and`
are kept as the store spelled them; [`series_key`](#series_key) compares them equal.

#### Arguments

* `name` - The series name a store reported.

#### Returns

The cleaned name; empty only when `name` is blank.

### series_key

```rust
pub fn series_key(name: &str) -> String
```

The key series names are matched by.

The [`clean_series_name`](#clean_series_name) of `name`, with `&` read as `and`, then
lowercased without accents, apostrophes or parenthesised text, so
`King & Maxwell Series`, `King and Maxwell` and
`King and Maxwell (abridged)` share one key.

#### Arguments

* `name` - A series name, as a store reported it or as stored.

#### Returns

The key; empty when `name` is blank.
