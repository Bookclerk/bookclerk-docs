# bookclerk_intake

Drop folder items and when they have settled, without IO.

A drop folder is a storage plugin instance with the intake role (see
`docs/server/drop-folders.md`). The host lists it page by page and feeds
every listed object to an [`ItemGrouper`](#itemgrouper), which:

1. drops objects that are never part of an item: names starting with
   `.`, partial downloads and temporaries, the host's own `_imported` and
   `_failed` folders, and keys that are not a clean relative path;
2. groups the rest into items, one per top-level entry (one level
   deeper in `subfolder` mode, where the top level names the patron);
3. gives each item a [`ItemSignature`](#itemsignature) over its members' keys, sizes and
   change signatures, independent of listing order.

[`step`](#step-1) then compares each item with what the host stored for it last
time and says what to store now, and whether the item is ready to
ingest: an item is ready once its signature has stayed the same for the
folder's quiet period. No portable way exists to see whether another
program still has a file open, so an unchanged signature over time is
the only test of a finished copy.

Keys come from the folder, which anyone with access to it can fill, so
this crate is an untrusted-input boundary: the `intake_items` fuzz
target checks it against an independent reading.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ListedObject`](#listedobject) | struct | One object as a drop folder listing reports it. |
| [`ItemMember`](#itemmember) | struct | A member file of an item. |
| [`ItemSignature`](#itemsignature) | struct | A digest of an item's members, independent of listing order. |
| [`ObservedItem`](#observeditem) | struct | One item as a whole listing shows it. |
| [`ItemGrouper`](#itemgrouper) | struct | Groups a drop folder's listing into items. |
| [`TrackedItem`](#trackeditem) | struct | One item's stored record. |
| [`Placement`](#placement) | enum | Where the grouper put one listed object. |
| [`ItemLimit`](#itemlimit) | enum | Why an item can never become ready as listed. |
| [`TrackedState`](#trackedstate) | enum | What the host stored about one item. |
| [`Step`](#step) | enum | What to do with one item's record after a listing. |
| [`signature_of`](#signature_of) | function | The order-independent digest of sorted `members`: each member's key, size and signature, length-prefixed so no two lists collide by concatenation, then a marker when the item is over a limit. An item within every limit hashes exactly as before the marker existed. |
| [`step`](#step-1) | function | Compares one item's stored record with the latest full listing. |
| [`MAX_KEY_BYTES`](#max_key_bytes) | constant | Longest object key the grouper accepts, in UTF-8 bytes (the largest object-store key). |
| [`MAX_ITEM_MEMBERS`](#max_item_members) | constant | Most member files one item may hold. An item with more is reported as `ItemLimit::TooManyMembers` and never becomes ready. |

## Structs

### ListedObject

```rust
pub struct ListedObject<'a> {
    pub key: &'a str,
    pub size: u64,
    pub signature: &'a str,
}
```

One object as a drop folder listing reports it.

**Fields**

- `key`: Object key relative to the drop folder's root, `/`-separated.
- `size`: Size in bytes.
- `signature`: The backend's change signature (`ObjectInfo.signature`, else its
  ETag or modification time); empty when the backend has none, in
  which case only the size can show a change.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### ItemMember

```rust
pub struct ItemMember {
    pub key: String,
    pub size: u64,
    pub signature: String,
}
```

A member file of an item.

**Fields**

- `key`: Object key relative to the drop folder's root.
- `size`: Size in bytes.
- `signature`: The backend's change signature, possibly empty.

Implements: `Clone`, `Debug`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`

### ItemSignature

```rust
pub struct ItemSignature(/* private */);
```

A digest of an item's members, independent of listing order.

#### ItemSignature::to_hex

```rust
pub fn to_hex(self) -> String
```

The signature as 64 lowercase hex characters, as the host stores it.

#### ItemSignature::from_hex

```rust
pub fn from_hex(text: &str) -> Option<Self>
```

Reads a stored signature; `None` for anything but 64 hex characters.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`

### ObservedItem

```rust
pub struct ObservedItem {
    pub key: String,
    pub patron_folder: Option<String>,
    pub members: Vec<ItemMember>,
    pub total_bytes: u64,
    pub signature: ItemSignature,
    pub limit: Option<ItemLimit>,
}
```

One item as a whole listing shows it.

**Fields**

- `key`: The item's key: its top-level name, or `<patron folder>/<name>` in
  `subfolder` mode.
- `patron_folder`: The patron folder's name in `subfolder` mode; `None` in `owner`
  mode.
- `members`: Member files sorted by key, at most [`MAX_ITEM_MEMBERS`](#max_item_members).
- `total_bytes`: Sum of the members' sizes, saturating.
- `signature`: Digest of every member's key, size and signature, and of whether
  the item went past [`MAX_ITEM_MEMBERS`](#max_item_members) (so a member added past the
  cap still changes it).
- `limit`: Set when the item can never become ready as listed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ItemGrouper

```rust
pub struct ItemGrouper<'s> {
    /* private fields */
}
```

Groups a drop folder's listing into items.

#### ItemGrouper::new

```rust
pub fn new(settings: &'s IntakeSettings) -> Self
```

An empty grouper for one listing of a folder with `settings`.

#### ItemGrouper::add

```rust
pub fn add(&mut self, object: ListedObject<'_>) -> Placement
```

Adds one listed object and says where it went.

A key listed twice counts once, with the size and signature of the
last listing.

#### ItemGrouper::finish

```rust
pub fn finish(self) -> Vec<ObservedItem>
```

Every item seen, sorted by key.

Implements: `Debug`

### TrackedItem

```rust
pub struct TrackedItem {
    pub state: TrackedState,
    pub signature: ItemSignature,
    pub quiet_since_ms: u64,
}
```

One item's stored record.

**Fields**

- `state`: The item's state.
- `signature`: The signature the state applies to.
- `quiet_since_ms`: When the signature was first seen unchanged, in unix milliseconds.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Enums

### Placement

```rust
pub enum Placement {
    Member(String),
    Ignored,
    Stray,
}
```

Where the grouper put one listed object.

**Variants**

- `Member`: A member of the item with this key.
- `Ignored`: Never part of an item: an ignored name, a host folder, or a key
  that is not a clean relative path.
- `Stray`: A file directly under the root in `subfolder` mode, which belongs
  to no patron folder; it is left alone.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ItemLimit

```rust
pub enum ItemLimit {
    TooManyMembers,
}
```

Why an item can never become ready as listed.

**Variants**

- `TooManyMembers`: The item holds more than [`MAX_ITEM_MEMBERS`](#max_item_members) files.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### TrackedState

```rust
pub enum TrackedState {
    Settling,
    Queued,
    Done,
    Failed,
}
```

What the host stored about one item.

**Variants**

- `Settling`: Seen, not yet quiet for long enough.
- `Queued`: Handed to an ingest job that has not finished.
- `Done`: Ingested and left in place (`after_ingest = leave`); ingested again
  only if it changes.
- `Failed`: Ingest failed; tried again only if it changes.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Step

```rust
pub enum Step {
    Keep,
    Store(TrackedItem),
    Ready(TrackedItem),
    Forget,
}
```

What to do with one item's record after a listing.

**Variants**

- `Keep`: Nothing changed.
- `Store`: Store this record (new or changed).
- `Ready`: The item has settled: store this `Queued` record and queue one
  ingest job for it.
- `Forget`: The item is gone: delete its record.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### signature_of

```rust
pub fn signature_of(members: &[ItemMember], limit: Option<ItemLimit>) -> ItemSignature
```

The order-independent digest of sorted `members`: each member's key,
size and signature, length-prefixed so no two lists collide by
concatenation, then a marker when the item is over a limit. An item
within every limit hashes exactly as before the marker existed.

#### Arguments

* `members` - The members, sorted by key.
* `limit` - The limit the item went past, if any.

### step

```rust
pub fn step(
    tracked: Option<&TrackedItem>,
    observed: Option<&ObservedItem>,
    now_ms: u64,
    settle_ms: u64,
) -> Step
```

Compares one item's stored record with the latest full listing.

#### Arguments

* `tracked` - The stored record, if any.
* `observed` - The item as the listing shows it; `None` when the
  listing no longer has it.
* `now_ms` - The current time, unix milliseconds.
* `settle_ms` - The folder's quiet period, milliseconds.

#### Returns

What to store. A queued item keeps its record until the ingest job
finishes, even if its files changed or vanished: the job compares the
signature before it reads and reports the item failed if it moved.

## Constants

### MAX_KEY_BYTES

```rust
pub const MAX_KEY_BYTES: usize = 1024;
```

Longest object key the grouper accepts, in UTF-8 bytes (the largest
object-store key).

### MAX_ITEM_MEMBERS

```rust
pub const MAX_ITEM_MEMBERS: usize = 4096;
```

Most member files one item may hold. An item with more is reported as
[`ItemLimit::TooManyMembers`](#itemlimit) and never becomes ready.
