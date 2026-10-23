# bookclerk_naming

Port of Libation's [NamingTemplate] tag template naming engine.

## Audience

Host acquire / settings code that expands folder and file templates. Guest
plugins do not depend on this crate.

Upstream lives in `FileManager.NamingTemplate` (class `NamingTemplate`); the
book/property tags and formatters come from `LibationFileManager.Templates`.
This crate targets classic / Chardonnay parity for property tags,
conditionals, and the text / number / list / name / series / date formatters.

Current Libation templates use `<tag->…<-tag>` conditionals. Older Libation
forms are still accepted for compatibility: `<if series>…<end if>`,
`%asin%`, and bare `<asin>` tags.

Style: `docs/code-documentation.md`.

[NamingTemplate]: https://github.com/rmcrackan/Libation/tree/master/Source/FileManager/NamingTemplate

## Example
```rust
use bookclerk_naming::{expand_template, BookContext, Contributor};

let book = BookContext {
    isbn: "B00X".into(),
    title_with_subtitle: Some("A Study in Scarlet".into()),
    authors: vec![Contributor::new("Arthur Conan Doyle", None)],
    ..Default::default()
};
let out = expand_template("<author> - <title> [<id>]", &book, None).unwrap();
assert_eq!(out, "Arthur Conan Doyle - A Study in Scarlet [B00X]");
```

| Item | Kind | Summary |
| --- | --- | --- |
| [`BookContext`](#bookcontext) | struct | All data required to evaluate a naming template for a book. |
| [`ChapterContext`](#chaptercontext) | struct | Per-chapter context for chapter file / title templates. |
| [`Contributor`](#contributor) | struct | An author or narrator (parsed lazily into name components). |
| [`Series`](#series) | struct | A series membership. |
| [`TemplatePart`](#templatepart) | struct | A single evaluated fragment of a template. |
| [`Template`](#template) | struct | A parsed, reusable naming template. |
| [`ContentKind`](#contentkind) | enum | The kind of content a book represents. |
| [`NamingError`](#namingerror) | enum | Errors produced while parsing or evaluating a template. |
| [`expand_template`](#expand_template) | function | Parse and evaluate `template` in one step. |
| [`apply_path_replacements`](#apply_path_replacements) | function | Apply classic Libation `ReplacementCharacters` rules to `input`. |
| [`remove_spaces`](#remove_spaces) | function | Collapse whitespace across an ordered list of path-segment part strings, mirroring Bookclerk's `Templates.RemoveSpaces`: |
| [`expand_filename`](#expand_filename) | function | Evaluate `template` into a single sanitised filename component. |
| [`expand_folder`](#expand_folder) | function | Evaluate a folder `template` into its sanitised path segments. |
| [`property_tag_names`](#property_tag_names) | function | Returns `(tag_name, supports_format_specifier)` for every Chardonnay property tag. |
| [`Result`](#result) | type alias | A convenient result alias for this crate. |

## Re-exports

- [`bookclerk_config::ReplacementRule`](../bookclerk_config/index.md#replacementrule)

## Structs

### BookContext

```rust
pub struct BookContext {
    pub isbn: String,
    pub title: Option<String>,
    pub subtitle: Option<String>,
    pub title_with_subtitle: Option<String>,
    pub authors: Vec<Contributor>,
    pub narrators: Vec<Contributor>,
    pub series: Vec<Series>,
    pub tags: Vec<String>,
    pub account: Option<String>,
    pub account_nickname: Option<String>,
    pub locale: Option<String>,
    pub language: Option<String>,
    pub year_published: Option<i32>,
    pub published_at: Option<NaiveDateTime>,
    pub purchased_at: Option<NaiveDateTime>,
    pub file_date: Option<NaiveDateTime>,
    pub length_minutes: Option<f64>,
    pub bitrate: Option<i64>,
    pub samplerate: Option<i64>,
    pub channels: Option<i64>,
    pub codec: Option<String>,
    pub is_abridged: bool,
    pub content_kind: ContentKind,
    pub publisher: Option<String>,
    pub categories: Vec<String>,
    pub bookclerk_version: Option<String>,
    pub file_version: Option<String>,
}
```

All data required to evaluate a naming template for a book.

**Fields**

- `isbn`: Canonical title id (ISBN / ASIN fallback) — used by `<id>`.
- `title`: Audible title (no subtitle) — used by `<audible title>` and `<title short>`.
- `subtitle`: Audible subtitle — used by `<audible subtitle>`.
- `title_with_subtitle`: Full title with subtitle — used by `<title>`. Falls back to [`Self::title`](#bookcontext).
- `authors`: Authors in display order for `<author>` / `<authors>` / `<first author>`.
- `narrators`: Narrators in display order for `<narrator>` / `<narrators>`.
- `series`: Series memberships for `<series>` / `<series#>` / conditionals.
- `tags`: Free-form tags for `<tags>` / `[tag]` search parity helpers.
- `account`: Store account id string for `<account>` when set.
- `account_nickname`: Operator nickname for the account (`<account nickname>`).
- `locale`: Marketplace / locale code (`us`, `uk`, …) for `<locale>`.
- `language`: Content language for `<language>`.
- `year_published`: Publication year for `<year>` (Gregorian calendar).
- `published_at`: Publication date — used by `<pub date>`.
- `purchased_at`: Date added to the account — used by `<date added>`.
- `file_date`: File date — used by `<file date>`.
- `length_minutes`: Runtime in minutes for `<duration>` / related tags (may be fractional).
- `bitrate`: Audio bitrate in kbps when known.
- `samplerate`: Sample rate in Hz when known.
- `channels`: Channel count when known (`1` = mono, `2` = stereo).
- `codec`: Codec / container label when known (`aac`, `mp3`, …).
- `is_abridged`: Whether the title is abridged (`<abridged>` conditional).
- `content_kind`: Book vs podcast/episode classification for podcast-aware templates.
- `publisher`: Publisher imprint for `<publisher>`.
- `categories`: Genre / category labels for `<genre>` / `<categories>`.
- `bookclerk_version`: Bookclerk package version string for `<bookclerk version>` when set.
- `file_version`: Liberated file / encoding version tag for `<file version>` when set.

Implements: `Clone`, `Debug`, `Default`

### ChapterContext

```rust
pub struct ChapterContext {
    pub chapter_number: u32,
    pub chapter_count: u32,
    pub chapter_title: Option<String>,
    pub file_date: Option<NaiveDateTime>,
}
```

Per-chapter context for chapter file / title templates.

**Fields**

- `chapter_number`: 1-based chapter position (`<ch#>`).
- `chapter_count`: Total number of chapters (`<ch count>`).
- `chapter_title`: Chapter title (`<ch title>`).
- `file_date`: Optional chapter file date override (`<file date>`).

Implements: `Clone`, `Debug`, `Default`

### Contributor

```rust
pub struct Contributor {
    pub name: String,
    pub id: Option<String>,
}
```

An author or narrator (parsed lazily into name components).

**Fields**

- `name`: Display name as shown in templates (`<author>`, `<narrator>`, …).
- `id`: Optional store-native person id when known.

#### Contributor::new

```rust
pub fn new(name: impl Into<String>, id: Option<String>) -> Self
```

Build a contributor from a display name and optional store id.

Implements: `Clone`, `Debug`, `Default`

### Series

```rust
pub struct Series {
    pub name: String,
    pub order: Option<String>,
    pub id: Option<String>,
}
```

A series membership.

**Fields**

- `name`: Series display name (`<series>`, `<first series>`, …).
- `order`: Raw order string (e.g. `"1"`, `"1-5"`, `"6"`).
- `id`: Optional store-native series id when known.

#### Series::new

```rust
pub fn new(name: impl Into<String>, order: Option<String>, id: Option<String>) -> Self
```

Build a series membership from name, optional order, and optional id.

Implements: `Clone`, `Debug`, `Default`

### TemplatePart

```rust
pub struct TemplatePart {
    pub value: String,
    pub is_literal: bool,
}
```

A single evaluated fragment of a template.

**Fields**

- `value`: The rendered text.
- `is_literal`: `true` for literal template text, `false` for a resolved tag value.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Template

```rust
pub struct Template {
    /* private fields */
}
```

A parsed, reusable naming template.

#### Template::parse

```rust
pub fn parse(template: &str) -> Result<Self>
```

Parse a template string.

##### Errors
Returns [`NamingError::Parse`](#namingerror) if the template is fundamentally invalid.

#### Template::evaluate

```rust
pub fn evaluate(&self, book: &BookContext, chapter: Option<&ChapterContext>) -> String
```

Evaluate the template against a book (and optional chapter) context,
returning the concatenated result string.

#### Template::evaluate_parts

```rust
pub fn evaluate_parts(
    &self,
    book: &BookContext,
    chapter: Option<&ChapterContext>,
) -> Vec<TemplatePart>
```

Evaluate the template into its individual [`TemplatePart`](#templatepart)s, preserving
whether each fragment came from literal text or a resolved tag. This is
useful for path building where literal `/` should create directories but
tag values should not.

Implements: `Clone`, `Debug`

## Enums

### ContentKind

```rust
pub enum ContentKind {
    Book,
    Episode,
    Podcast,
    PodcastParent,
}
```

The kind of content a book represents.

**Variants**

- `Book`: Ordinary audiobook title (default).
- `Episode`: Podcast episode acquired as a child of a show.
- `Podcast`: Podcast show / feed treated as a liberatable title.
- `PodcastParent`: Parent podcast container (show) without a single episode payload.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `PartialEq`

### NamingError

```rust
pub enum NamingError {
    Parse(String),
}
```

Errors produced while parsing or evaluating a template.

**Variants**

- `Parse`: The template string could not be parsed.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### expand_template

```rust
pub fn expand_template(
    template: &str,
    book: &BookContext,
    chapter: Option<&ChapterContext>,
) -> Result<String>
```

Parse and evaluate `template` in one step.

#### Errors
Returns [`NamingError::Parse`](#namingerror) if the template is fundamentally invalid.

### apply_path_replacements

```rust
pub fn apply_path_replacements(input: &str, rules: &[ReplacementRule]) -> String
```

Apply classic Libation `ReplacementCharacters` rules to `input`.

### remove_spaces

```rust
pub fn remove_spaces(parts: &[String]) -> String
```

Collapse whitespace across an ordered list of path-segment part strings,
mirroring Bookclerk's `Templates.RemoveSpaces`:

* leading / trailing parts that are entirely whitespace are dropped,
* the first part is left-trimmed and the last part is right-trimmed,
* runs of spaces inside each part are collapsed to a single space, and
* a doubled space straddling a part boundary is collapsed.

The parts are then concatenated into the final segment string.

### expand_filename

```rust
pub fn expand_filename(
    template: &str,
    book: &BookContext,
    chapter: Option<&ChapterContext>,
    rules: &[ReplacementRule],
) -> Result<String>
```

Evaluate `template` into a single sanitised filename component.

Every part (literal and tag) has the replacement `rules` applied to it, then
whitespace is collapsed with [`remove_spaces`](#remove_spaces), matching the output of
Bookclerk's `FileTemplate.GetFilename` for one path segment. Passing an empty
`rules` slice performs no character replacement (useful for exact
tag-output assertions).

#### Errors
Returns [`NamingError::Parse`](#namingerror) if the template is fundamentally invalid.

### expand_folder

```rust
pub fn expand_folder(
    template: &str,
    book: &BookContext,
    chapter: Option<&ChapterContext>,
    rules: &[ReplacementRule],
) -> Result<Vec<String>>
```

Evaluate a folder `template` into its sanitised path segments.

Literal `/` characters delimit directories (like Bookclerk's `FolderTemplate`),
while `/` produced by a tag value is replaced via `rules`. Each resulting
segment is space-collapsed with [`remove_spaces`](#remove_spaces); empty segments are dropped.

#### Errors
Returns [`NamingError::Parse`](#namingerror) if the template is fundamentally invalid.

#### Panics

Panics when an internal invariant does not hold.

### property_tag_names

```rust
pub fn property_tag_names() -> &'static [(&'static str, bool)]
```

Returns `(tag_name, supports_format_specifier)` for every Chardonnay property tag.

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, NamingError>;
```

A convenient result alias for this crate.
