# bookclerk_archive

Bounded extraction of the archives people upload.

An uploaded ZIP or tar file is untrusted input: its entry names may try to
escape the output folder, its sizes may lie, and its contents may expand
without end. This crate decides what an archive may write and stops it
when it overreaches. It never opens a file itself:

- [`EntryPath::admit`](#entrypathadmit) turns a raw entry name into a relative path that
  cannot leave the output folder, or refuses it.
- [`extract`](#extract) walks an archive read from any `Read + Seek` source and
  writes each admitted entry through a [`Sink`](#sink), counting the bytes it
  actually writes against [`Limits`](#limits) rather than trusting the archive's
  headers.
- [`ExtractReport::validate`](#extractreportvalidate) re-checks a report that crossed a process
  boundary, because the code that produced it parsed untrusted bytes.

The same code runs in the confined media worker's `extract` job, in the
`archive_extract` fuzz target (over an in-memory sink) and in the unit
tests. Nested archives are never expanded, links and devices are never
created, and encrypted entries are refused.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Sink`](#sink) | trait | Where an extraction writes its files. |
| [`Limits`](#limits) | struct | The caps one extraction runs under. |
| [`EntryPath`](#entrypath) | struct | A relative path that stays inside the folder it is joined to. |
| [`FoldedNames`](#foldednames) | struct | The files written so far, by folded name, and the folders they imply. |
| [`ExtractReport`](#extractreport) | struct | What one extraction did. |
| [`RefusedEntry`](#refusedentry) | struct | One entry the extraction did not write, and why. |
| [`WrittenEntry`](#writtenentry) | struct | One file the extraction wrote. |
| [`DirSink`](#dirsink) | struct | Writes files under one folder on disk, which should start empty. |
| [`MemorySink`](#memorysink) | struct | Keeps files in memory, for tests and the fuzz target. |
| [`ArchiveFormat`](#archiveformat) | enum | The archive formats Bookclerk expands. |
| [`NameClash`](#nameclash) | enum | Why a path cannot join a `FoldedNames` set. |
| [`PathRefusal`](#pathrefusal) | enum | Why a name was not admitted as an `EntryPath`. |
| [`RefusalReason`](#refusalreason) | enum | Why one entry was not written. |
| [`ReportError`](#reporterror) | enum | A report that breaks the limits it was produced under. |
| [`StopReason`](#stopreason) | enum | Why an extraction stopped before the end of the archive. |
| [`ExtractError`](#extracterror) | enum | Why an archive could not be walked at all. |
| [`extract`](#extract) | function | Expands an archive through `sink` under `limits`. |
| [`SNIFF_BYTES`](#sniff_bytes) | constant | Bytes `ArchiveFormat::sniff` needs to recognise every format: the tar magic sits at offset 257. |
| [`MAX_DEPTH`](#max_depth) | constant | Most components an admitted path may have. |
| [`MAX_PATH_BYTES`](#max_path_bytes) | constant | Longest admitted path, in bytes of UTF-8. |
| [`MAX_REFUSED_NAME_CHARS`](#max_refused_name_chars) | constant | Longest name kept for a refused entry, in characters. Longer names are cut; the name is only shown to people, never used as a path. |

## Modules

- [`fixtures`](fixtures.md): Archive builders for tests and fuzz seeds, including archives a careful tool would refuse to write.

## Traits

### Sink

```rust
pub trait Sink {
    type Writer: Write;
    fn create(&mut self, path: &EntryPath) -> Result<Self::Writer>;
    fn discard(&mut self, path: &EntryPath);
}
```

Where an extraction writes its files.

[`crate::extract`](#extract) only ever names admitted [`EntryPath`](#entrypath)s, checked
against earlier names, so a sink never sees two paths that clash.

#### Sink::Writer

```rust
type Writer: Write
```

The writer for one file.

#### Sink::create

```rust
fn create(&mut self, path: &EntryPath) -> Result<Self::Writer>
```

Creates the file at `path`, and any folders it sits in.

##### Arguments

* `path` - The file to create.

##### Errors

The I/O error, when the file cannot be created; the entry is then
refused as corrupt and the walk goes on.

#### Sink::discard

```rust
fn discard(&mut self, path: &EntryPath)
```

Removes a file created for an entry that was then refused.

##### Arguments

* `path` - A file this sink created.

Implemented by: `DirSink`, `MemorySink`

## Structs

### Limits

```rust
pub struct Limits {
    pub max_entries: u32,
    pub max_total_bytes: u64,
    pub max_entry_bytes: u64,
    pub max_ratio: u64,
    pub ratio_floor_bytes: u64,
}
```

The caps one extraction runs under.

Every cap counts bytes as they are written, never sizes an archive
declares. The host chooses them per job (the total is never more than the
uploader's remaining quota) and sends them to the worker with the job.

**Fields**

- `max_entries`: Most entries the archive may list, counting refused ones. An archive
  that lists more is stopped before anything is written.
- `max_total_bytes`: Most bytes the whole extraction may write. Reaching it stops the
  extraction.
- `max_entry_bytes`: Most bytes one entry may write. An entry that needs more is refused
  and its partial file discarded.
- `max_ratio`: Largest ratio of written bytes to compressed bytes an entry may reach
  once it is past [`Limits::ratio_floor_bytes`](#limits). Audio barely
  compresses, so an honest audiobook sits near 1:1.
- `ratio_floor_bytes`: Bytes an entry (or a whole gzip stream) may write before the ratio
  applies, so small, very compressible text files are not refused.

#### Limits::UPLOAD_DEFAULTS

```rust
pub const UPLOAD_DEFAULTS: Self = _
```

The defaults for uploads: 10,000 entries, 64 GiB in all, 20 GiB per
entry, 100:1 past 1 MiB.

#### Limits::with_total_at_most

```rust
pub fn with_total_at_most(self, total: u64) -> Self
```

The same caps with the total lowered to `total` when that is smaller,
for an uploader with less quota left than the default total.

##### Arguments

* `total` - The most bytes this extraction may write.

#### Limits::ratio_allowance

```rust
pub fn ratio_allowance(&self, compressed: u64) -> u64
```

Most bytes something compressed to `compressed` bytes may expand to.

##### Arguments

* `compressed` - The entry's (or stream's) compressed size in bytes.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EntryPath

```rust
pub struct EntryPath(/* private */);
```

A relative path that stays inside the folder it is joined to.

Components are separated by `/`, none is empty, `.` or `..`, and none
holds a control character, a `:` or a `\`. The path is valid UTF-8, at
most [`MAX_PATH_BYTES`](#max_path_bytes) long and at most [`MAX_DEPTH`](#max_depth) deep, and no
component is a Windows device name or ends in a dot or a space, so the
same path is safe on every filesystem a destination may use.

#### EntryPath::admit

```rust
pub fn admit(raw: &str) -> Result<Self, PathRefusal>
```

Admits a name as a relative path, or says why it cannot be one.

`\` is read as a separator (ZIP tools on Windows write it), empty and
`.` components are dropped, and everything else in the policy above is
refused rather than repaired.

##### Arguments

* `raw` - The name as the archive or manifest spells it.

##### Errors

The [`PathRefusal`](#pathrefusal) for the first rule the name breaks.

#### EntryPath::admit_bytes

```rust
pub fn admit_bytes(raw: &[u8]) -> Result<Self, PathRefusal>
```

Admits a name given as bytes, which must be UTF-8.

##### Arguments

* `raw` - The name's bytes, as a tar header carries them.

##### Errors

[`PathRefusal::NotUtf8`](#pathrefusal), or what [`EntryPath::admit`](#entrypathadmit) refuses.

#### EntryPath::as_str

```rust
pub fn as_str(&self) -> &str
```

The path, components separated by `/`.

#### EntryPath::components

```rust
pub fn components(&self) -> impl Iterator<Item = &str>
```

The path's components, outermost first.

#### EntryPath::file_name

```rust
pub fn file_name(&self) -> &str
```

The last component.

#### EntryPath::fold_key

```rust
pub fn fold_key(&self) -> String
```

The key two paths clash on: NFC-normalised and lower-cased, so names
a case-insensitive or normalising filesystem would merge share it.

#### EntryPath::is_system_file

```rust
pub fn is_system_file(&self) -> bool
```

Whether this is an operating system's bookkeeping file rather than
content: macOS resource forks (`__MACOSX/`, `._name`), `.DS_Store`,
and Windows' `Thumbs.db` and `desktop.ini`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### FoldedNames

```rust
pub struct FoldedNames {
    /* private fields */
}
```

The files written so far, by folded name, and the folders they imply.

[`FoldedNames::insert`](#foldednamesinsert) refuses a path that would overwrite an earlier
file on any filesystem, or needs a folder where a file is.

#### FoldedNames::new

```rust
pub fn new() -> Self
```

An empty set.

#### FoldedNames::check

```rust
pub fn check(&self, path: &EntryPath) -> Result<(), NameClash>
```

Checks a file path against the set without adding it.

##### Arguments

* `path` - The file about to be written.

##### Errors

The clash, when the path cannot be written beside the others.

#### FoldedNames::insert

```rust
pub fn insert(&mut self, path: &EntryPath) -> Result<(), NameClash>
```

Adds a file path, after [`FoldedNames::check`](#foldednamescheck).

##### Arguments

* `path` - The file being written.

##### Errors

The clash, when the path cannot be written beside the others; the set
is then unchanged.

#### FoldedNames::len

```rust
pub fn len(&self) -> usize
```

How many files the set holds.

#### FoldedNames::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Whether the set holds no file.

Implements: `Clone`, `Debug`, `Default`

### ExtractReport

```rust
pub struct ExtractReport {
    pub format: ArchiveFormat,
    pub written: Vec<WrittenEntry>,
    pub refused: Vec<RefusedEntry>,
    pub stopped: Option<StopReason>,
    pub total_bytes: u64,
}
```

What one extraction did.

**Fields**

- `format`: The format the archive was walked as.
- `written`: The files written, in archive order.
- `refused`: The entries refused, in archive order. Folders are not listed.
- `stopped`: Why the walk stopped early, if it did.
- `total_bytes`: Bytes written in all; the sum of [`WrittenEntry::bytes`](#writtenentry).

#### ExtractReport::new

```rust
pub fn new(format: ArchiveFormat) -> Self
```

An empty report for an archive of `format`.

##### Arguments

* `format` - The format being walked.

#### ExtractReport::refuse

```rust
pub fn refuse(&mut self, raw_name: &[u8], reason: RefusalReason)
```

Records a refused entry, cleaning its name for display.

##### Arguments

* `raw_name` - The entry's name bytes.
* `reason` - Why it was refused.

#### ExtractReport::validate

```rust
pub fn validate(&self, limits: &Limits) -> Result<(), ReportError>
```

Checks a report produced by another process against the limits the
job ran under.

Paths were already admitted when the report was deserialised; this
adds the checks that span entries.

##### Arguments

* `limits` - The limits sent with the job.

##### Errors

The first [`ReportError`](#reporterror) found.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RefusedEntry

```rust
pub struct RefusedEntry {
    pub name: String,
    pub reason: RefusalReason,
}
```

One entry the extraction did not write, and why.

**Fields**

- `name`: The entry's name as the archive spells it, for people to read:
  invalid UTF-8 and control characters replaced, cut to
  [`MAX_REFUSED_NAME_CHARS`](#max_refused_name_chars).
- `reason`: Why it was refused.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WrittenEntry

```rust
pub struct WrittenEntry {
    pub path: EntryPath,
    pub bytes: u64,
}
```

One file the extraction wrote.

**Fields**

- `path`: Where it was written, relative to the output folder.
- `bytes`: How many bytes were written.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DirSink

```rust
pub struct DirSink {
    /* private fields */
}
```

Writes files under one folder on disk, which should start empty.

Files are created with `create_new`, so nothing that already exists is
opened and no link is followed. Folders are created one component at a time and must be
real folders.

#### DirSink::new

```rust
pub fn new(root: impl Into<PathBuf>) -> Self
```

A sink writing under `root`.

##### Arguments

* `root` - An existing, empty folder.

#### DirSink::root

```rust
pub fn root(&self) -> &Path
```

The output folder.

Implements: `Clone`, `Debug`, `Sink`

### MemorySink

```rust
pub struct MemorySink {
    /* private fields */
}
```

Keeps files in memory, for tests and the fuzz target.

#### MemorySink::new

```rust
pub fn new() -> Self
```

An empty sink.

#### MemorySink::files

```rust
pub fn files(&self) -> BTreeMap<EntryPath, Vec<u8>>
```

The files written so far, by path.

Implements: `Clone`, `Debug`, `Default`, `Sink`

## Enums

### ArchiveFormat

```rust
pub enum ArchiveFormat {
    Zip,
    Tar,
    TarGz,
}
```

The archive formats Bookclerk expands.

The format is always decided from the bytes ([`ArchiveFormat::sniff`](#archiveformatsniff)),
never from the file name, so a renamed file cannot pick a decoder.

**Variants**

- `Zip`: A ZIP archive (stored, Deflate or Deflate64 entries; Zip64 sizes).
- `Tar`: An uncompressed POSIX, GNU or PAX tar archive.
- `TarGz`: A tar archive inside one gzip stream.

#### ArchiveFormat::sniff

```rust
pub fn sniff(head: &[u8]) -> Option<Self>
```

Recognises an archive from its first bytes.

##### Arguments

* `head` - The file's first bytes; [`SNIFF_BYTES`](#sniff_bytes) is enough for every
  format, fewer may miss a tar archive.

##### Returns

The format, or `None` when the bytes start no format this crate
expands. A gzip stream is reported as [`ArchiveFormat::TarGz`](#archiveformat); whether
it really holds a tar archive shows when it is walked.

#### ArchiveFormat::looks_like_archive

```rust
pub fn looks_like_archive(head: &[u8]) -> bool
```

Whether bytes look like any archive or compressed container, this
crate's formats or others (7z, RAR, xz, bzip2, zstd).

Used to refuse nested archives: an entry that is itself an archive is
not expanded and not kept.

##### Arguments

* `head` - The entry's first bytes.

#### ArchiveFormat::as_str

```rust
pub fn as_str(self) -> &'static str
```

The name used in logs and replies.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### NameClash

```rust
pub enum NameClash {
    Duplicate,
    FileFolder,
}
```

Why a path cannot join a [`FoldedNames`](#foldednames) set.

**Variants**

- `Duplicate`: Another file already has the same folded name.
- `FileFolder`: The path needs a folder where a file already is, or a file where a
  folder already is.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Error`, `PartialEq`, `Serialize`

### PathRefusal

```rust
pub enum PathRefusal {
    NotUtf8,
    Empty,
    Absolute,
    Colon,
    ParentDir,
    Control,
    Reserved,
    TooLong,
    TooDeep,
}
```

Why a name was not admitted as an [`EntryPath`](#entrypath).

**Variants**

- `NotUtf8`: The name is not valid UTF-8.
- `Empty`: Nothing is left once empty and `.` components are dropped (a
  directory entry such as `./`).
- `Absolute`: The name starts at a root (`/`, `\`).
- `Colon`: The name holds a `:`, which names a drive, a stream or a device on
  Windows.
- `ParentDir`: A component is `..`.
- `Control`: The name holds a control character (including NUL).
- `Reserved`: A component is a Windows device name or ends in a dot or a space.
- `TooLong`: The name or one of its components is too long.
- `TooDeep`: The name has more than [`MAX_DEPTH`](#max_depth) components.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Display`, `Eq`, `Error`, `Hash`, `PartialEq`, `Serialize`

### RefusalReason

```rust
pub enum RefusalReason {
    Path { refusal: PathRefusal },
    Clash { clash: NameClash },
    Link,
    Special,
    Encrypted,
    Compression,
    NestedArchive,
    EntryTooLarge,
    RatioExceeded,
    Overlap,
    SystemFile,
    Corrupt,
    WriteFailed,
}
```

Why one entry was not written.

**Variants**

- `Path`: Its name breaks the path policy.
  - `refusal`: The rule it breaks.
- `Clash`: Its name clashes with an earlier entry.
  - `clash`: How it clashes.
- `Link`: It is a symbolic or hard link.
- `Special`: It is a device, a FIFO or another special file.
- `Encrypted`: It is encrypted.
- `Compression`: It uses a compression method this crate does not decode.
- `NestedArchive`: It is itself an archive; nested archives are not expanded.
- `EntryTooLarge`: It would write more than [`Limits::max_entry_bytes`](#limits).
- `RatioExceeded`: It expands past [`Limits::max_ratio`](#limits).
- `Overlap`: Its data overlaps another entry's (a ZIP that reuses one stream for
  many names).
- `SystemFile`: It is an operating system's bookkeeping file, such as `__MACOSX/`.
- `Corrupt`: Its data could not be decoded or failed its checksum.
- `WriteFailed`: The output file could not be created or written.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ReportError

```rust
pub enum ReportError {
    TooManyEntries,
    EntryTooLarge(String),
    Total,
    Clash(String),
    RefusedName,
}
```

A report that breaks the limits it was produced under.

**Variants**

- `TooManyEntries`: More entries than [`Limits::max_entries`](#limits).
- `EntryTooLarge`: One file larger than [`Limits::max_entry_bytes`](#limits).
- `Total`: The total is not the sum of the files, or is past the limit.
- `Clash`: Two files clash.
- `RefusedName`: A refused entry's name is too long or holds a control character.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### StopReason

```rust
pub enum StopReason {
    TooManyEntries,
    TotalExceeded,
    Corrupt,
}
```

Why an extraction stopped before the end of the archive.

A stopped extraction's files are kept on disk only until the host reads
the report; the host discards the whole archive.

**Variants**

- `TooManyEntries`: The archive lists more than [`Limits::max_entries`](#limits) entries.
- `TotalExceeded`: Writing went past [`Limits::max_total_bytes`](#limits), or a gzip stream past
  its ratio allowance.
- `Corrupt`: The archive's structure broke partway through.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ExtractError

```rust
pub enum ExtractError {
    NotArchive,
    Io(Error),
}
```

Why an archive could not be walked at all.

**Variants**

- `NotArchive`: The bytes start no format this crate expands.
- `Io`: The archive could not be read.

Implements: `Debug`, `Display`, `Error`, `From<Error>`

## Functions

### extract

```rust
pub fn extract<R: Read + Seek, S: Sink>(
    reader: R,
    limits: &Limits,
    sink: &mut S,
) -> Result<ExtractReport, ExtractError>
```

Expands an archive through `sink` under `limits`.

The format is sniffed from the first bytes. Each entry's name is admitted
by [`EntryPath::admit`](#entrypathadmit) and checked against the names written before it;
links, devices, encrypted entries, nested archives and system files are
refused; bytes are counted as they are written. A refused entry's partial
file is discarded. Reaching the entry count or the total stops the walk,
and the caller should then discard everything ([`StopReason`](#stopreason)).

#### Arguments

* `reader` - The archive, positioned anywhere.
* `limits` - The caps for this extraction.
* `sink` - Where files go.

#### Returns

The [`ExtractReport`]: files written, entries refused and why, and why
the walk stopped if it did.

#### Errors

[`ExtractError::NotArchive`](#extracterror) when the format is not recognised, and
[`ExtractError::Io`](#extracterror) when the archive cannot be read before the walk
starts. A structure that breaks partway is reported as
[`StopReason::Corrupt`](#stopreason) instead.

## Constants

### SNIFF_BYTES

```rust
pub const SNIFF_BYTES: usize = 265;
```

Bytes [`ArchiveFormat::sniff`](#archiveformatsniff) needs to recognise every format: the tar
magic sits at offset 257.

### MAX_DEPTH

```rust
pub const MAX_DEPTH: usize = 32;
```

Most components an admitted path may have.

### MAX_PATH_BYTES

```rust
pub const MAX_PATH_BYTES: usize = 1024;
```

Longest admitted path, in bytes of UTF-8.

### MAX_REFUSED_NAME_CHARS

```rust
pub const MAX_REFUSED_NAME_CHARS: usize = 256;
```

Longest name kept for a refused entry, in characters. Longer names are
cut; the name is only shown to people, never used as a path.
