**bookclerk_naming**

# Module: bookclerk_naming

## Contents

**Structs**

- [`Template`](#template) - A parsed, reusable naming template.

**Enums**

- [`NamingError`](#namingerror) - Errors produced while parsing or evaluating a template.

**Functions**

- [`apply_path_replacements`](#apply_path_replacements) - Apply classic Libation `ReplacementCharacters` rules to `input`.
- [`expand_filename`](#expand_filename) - Evaluate `template` into a single sanitised filename component.
- [`expand_folder`](#expand_folder) - Evaluate a folder `template` into its sanitised path segments.
- [`expand_template`](#expand_template) - Parse and evaluate `template` in one step.
- [`property_tag_names`](#property_tag_names) - Returns `(tag_name, supports_format_specifier)` for every Chardonnay property tag.
- [`remove_spaces`](#remove_spaces) - Collapse whitespace across an ordered list of path-segment part strings,

**Type Aliases**

- [`Result`](#result) - A convenient result alias for this crate.

---

## bookclerk_naming::NamingError

*Enum*

Errors produced while parsing or evaluating a template.

**Variants:**
- `Parse(String)` - The template string could not be parsed.

**Traits:** Error, Eq

**Trait Implementations:**

- **Display**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NamingError) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NamingError`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_naming::Result

*Type Alias*: `std::result::Result<T, NamingError>`

A convenient result alias for this crate.



## bookclerk_naming::Template

*Struct*

A parsed, reusable naming template.

**Methods:**

- `fn parse(template: &str) -> Result<Self>` - Parse a template string.
- `fn evaluate(self: &Self, book: &BookContext, chapter: Option<&ChapterContext>) -> String` - Evaluate the template against a book (and optional chapter) context,
- `fn evaluate_parts(self: &Self, book: &BookContext, chapter: Option<&ChapterContext>) -> Vec<TemplatePart>` - Evaluate the template into its individual [`TemplatePart`](engine.md#bookclerk_namingenginetemplatepart)s, preserving

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Template`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_naming::apply_path_replacements

*Function*

Apply classic Libation `ReplacementCharacters` rules to `input`.

```rust
fn apply_path_replacements(input: &str, rules: &[ReplacementRule]) -> String
```



## bookclerk_naming::expand_filename

*Function*

Evaluate `template` into a single sanitised filename component.

Every part (literal and tag) has the replacement `rules` applied to it, then
whitespace is collapsed with [`remove_spaces`](bookclerk_naming.md#bookclerk_namingremove_spaces), matching the output of
Bookclerk's `FileTemplate.GetFilename` for one path segment. Passing an empty
`rules` slice performs no character replacement (useful for exact
tag-output assertions).

# Errors
Returns `NamingError::Parse` if the template is fundamentally invalid.

```rust
fn expand_filename(template: &str, book: &BookContext, chapter: Option<&ChapterContext>, rules: &[ReplacementRule]) -> Result<String>
```



## bookclerk_naming::expand_folder

*Function*

Evaluate a folder `template` into its sanitised path segments.

Literal `/` characters delimit directories (like Bookclerk's `FolderTemplate`),
while `/` produced by a tag value is replaced via `rules`. Each resulting
segment is space-collapsed with [`remove_spaces`](bookclerk_naming.md#bookclerk_namingremove_spaces); empty segments are dropped.

# Errors
Returns `NamingError::Parse` if the template is fundamentally invalid.

# Panics

Panics when an internal invariant does not hold.

```rust
fn expand_folder(template: &str, book: &BookContext, chapter: Option<&ChapterContext>, rules: &[ReplacementRule]) -> Result<Vec<String>>
```



## bookclerk_naming::expand_template

*Function*

Parse and evaluate `template` in one step.

# Errors
Returns `NamingError::Parse` if the template is fundamentally invalid.

```rust
fn expand_template(template: &str, book: &BookContext, chapter: Option<&ChapterContext>) -> Result<String>
```



## bookclerk_naming::property_tag_names

*Function*

Returns `(tag_name, supports_format_specifier)` for every Chardonnay property tag.

```rust
fn property_tag_names() -> &'static [(&'static str, bool)]
```



## bookclerk_naming::remove_spaces

*Function*

Collapse whitespace across an ordered list of path-segment part strings,
mirroring Bookclerk's `Templates.RemoveSpaces`:

* leading / trailing parts that are entirely whitespace are dropped,
* the first part is left-trimmed and the last part is right-trimmed,
* runs of spaces inside each part are collapsed to a single space, and
* a doubled space straddling a part boundary is collapsed.

The parts are then concatenated into the final segment string.

```rust
fn remove_spaces(parts: &[String]) -> String
```



