**bookclerk_library > store > title_files**

# Module: store::title_files

## Contents

**Structs**

- [`TitleFile`](#titlefile) - Who holds a recorded file key.

**Constants**

- [`ACQUIRE_WRITER`](#acquire_writer) - The `writer` of a key an acquire run on this node wrote.

---

## bookclerk_library::store::title_files::ACQUIRE_WRITER

*Constant*: `&str`

The `writer` of a key an acquire run on this node wrote.



## bookclerk_library::store::title_files::TitleFile

*Struct*

Who holds a recorded file key.

**Fields:**
- `book_uuid: String` - Library id of the title the file belongs to.
- `writer: String` - The host upload id that claimed the key last, or [`ACQUIRE_WRITER`](title_files.md#bookclerk_librarystoretitle_filesacquire_writer).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &TitleFile) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> TitleFile`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



