**bookclerk_naming > context**

# Module: context

## Contents

**Structs**

- [`BookContext`](#bookcontext) - All data required to evaluate a naming template for a book.
- [`ChapterContext`](#chaptercontext) - Per-chapter context for chapter file / title templates.
- [`Contributor`](#contributor) - An author or narrator (parsed lazily into name components).
- [`Series`](#series) - A series membership.

**Enums**

- [`ContentKind`](#contentkind) - The kind of content a book represents.

---

## bookclerk_naming::context::BookContext

*Struct*

All data required to evaluate a naming template for a book.

**Fields:**
- `isbn: String` - Canonical title id (ISBN / ASIN fallback) — used by `<id>`.
- `title: Option<String>` - Audible title (no subtitle) — used by `<audible title>` and `<title short>`.
- `subtitle: Option<String>` - Audible subtitle — used by `<audible subtitle>`.
- `title_with_subtitle: Option<String>` - Full title with subtitle — used by `<title>`. Falls back to `Self::title`.
- `authors: Vec<Contributor>` - Authors in display order for `<author>` / `<authors>` / `<first author>`.
- `narrators: Vec<Contributor>` - Narrators in display order for `<narrator>` / `<narrators>`.
- `series: Vec<Series>` - Series memberships for `<series>` / `<series#>` / conditionals.
- `tags: Vec<String>` - Free-form tags for `<tags>` / `[tag]` search parity helpers.
- `account: Option<String>` - Store account id string for `<account>` when set.
- `account_nickname: Option<String>` - Operator nickname for the account (`<account nickname>`).
- `locale: Option<String>` - Marketplace / locale code (`us`, `uk`, …) for `<locale>`.
- `language: Option<String>` - Content language for `<language>`.
- `year_published: Option<i32>` - Publication year for `<year>` (Gregorian calendar).
- `published_at: Option<chrono::NaiveDateTime>` - Publication date — used by `<pub date>`.
- `purchased_at: Option<chrono::NaiveDateTime>` - Date added to the account — used by `<date added>`.
- `file_date: Option<chrono::NaiveDateTime>` - File date — used by `<file date>`.
- `length_minutes: Option<f64>` - Runtime in minutes for `<duration>` / related tags (may be fractional).
- `bitrate: Option<i64>` - Audio bitrate in kbps when known.
- `samplerate: Option<i64>` - Sample rate in Hz when known.
- `channels: Option<i64>` - Channel count when known (`1` = mono, `2` = stereo).
- `codec: Option<String>` - Codec / container label when known (`aac`, `mp3`, …).
- `is_abridged: bool` - Whether the title is abridged (`<abridged>` conditional).
- `content_kind: ContentKind` - Book vs podcast/episode classification for podcast-aware templates.
- `publisher: Option<String>` - Publisher imprint for `<publisher>`.
- `categories: Vec<String>` - Genre / category labels for `<genre>` / `<categories>`.
- `bookclerk_version: Option<String>` - Bookclerk package version string for `<bookclerk version>` when set.
- `file_version: Option<String>` - Liberated file / encoding version tag for `<file version>` when set.

**Trait Implementations:**

- **Default**
  - `fn default() -> BookContext`
- **Clone**
  - `fn clone(self: &Self) -> BookContext`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_naming::context::ChapterContext

*Struct*

Per-chapter context for chapter file / title templates.

**Fields:**
- `chapter_number: u32` - 1-based chapter position (`<ch#>`).
- `chapter_count: u32` - Total number of chapters (`<ch count>`).
- `chapter_title: Option<String>` - Chapter title (`<ch title>`).
- `file_date: Option<chrono::NaiveDateTime>` - Optional chapter file date override (`<file date>`).

**Trait Implementations:**

- **Default**
  - `fn default() -> ChapterContext`
- **Clone**
  - `fn clone(self: &Self) -> ChapterContext`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_naming::context::ContentKind

*Enum*

The kind of content a book represents.

**Variants:**
- `Book` - Ordinary audiobook title (default).
- `Episode` - Podcast episode acquired as a child of a show.
- `Podcast` - Podcast show / feed treated as a liberatable title.
- `PodcastParent` - Parent podcast container (show) without a single episode payload.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ContentKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> ContentKind`
- **PartialEq**
  - `fn eq(self: &Self, other: &ContentKind) -> bool`



## bookclerk_naming::context::Contributor

*Struct*

An author or narrator (parsed lazily into name components).

**Fields:**
- `name: String` - Display name as shown in templates (`<author>`, `<narrator>`, …).
- `id: Option<String>` - Optional store-native person id when known.

**Methods:**

- `fn new<impl Into<String>>(name: impl Trait, id: Option<String>) -> Self` - Build a contributor from a display name and optional store id.

**Trait Implementations:**

- **Default**
  - `fn default() -> Contributor`
- **Clone**
  - `fn clone(self: &Self) -> Contributor`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_naming::context::Series

*Struct*

A series membership.

**Fields:**
- `name: String` - Series display name (`<series>`, `<first series>`, …).
- `order: Option<String>` - Raw order string (e.g. `"1"`, `"1-5"`, `"6"`).
- `id: Option<String>` - Optional store-native series id when known.

**Methods:**

- `fn new<impl Into<String>>(name: impl Trait, order: Option<String>, id: Option<String>) -> Self` - Build a series membership from name, optional order, and optional id.

**Trait Implementations:**

- **Default**
  - `fn default() -> Series`
- **Clone**
  - `fn clone(self: &Self) -> Series`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



