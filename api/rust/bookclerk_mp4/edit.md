# bookclerk_mp4::edit

Editing a `moov` that is already in memory: find a box, replace a child, and
keep every enclosing size field consistent.

Rewriting a sample table means growing or shrinking a box that several
ancestors declare the size of, so a splice here is never a plain
`Vec::splice`: [`splice_replace`](#splice_replace) walks back up the tree and patches each
ancestor's size by the same delta.

| Item | Kind | Summary |
| --- | --- | --- |
| [`find_box_range`](#find_box_range) | function | Find the first box of `fourcc` anywhere under the well-known containers. |
| [`find_direct_child`](#find_direct_child) | function | Find an immediate child of the box that starts at `parent_start`. |
| [`find_child_in_range`](#find_child_in_range) | function | Find a box of `fourcc` among the siblings laid out in `[start, end)`. |
| [`splice_replace`](#splice_replace) | function | Replace `buf[start..end]` with `replacement`, fixing every ancestor's size. |

## Functions

### find_box_range

```rust
pub fn find_box_range(buf: &[u8], fourcc: &[u8; 4]) -> Result<Option<(usize, usize)>>
```

Find the first box of `fourcc` anywhere under the well-known containers.

Returns the byte range `[start, end)` of the whole box, header included.

#### Arguments

* `buf` - Destination buffer resized to `size`.
* `fourcc` - Numeric `fourcc` value for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

### find_direct_child

```rust
pub fn find_direct_child(
    buf: &[u8],
    parent_start: usize,
    parent_end: usize,
    fourcc: &[u8; 4],
) -> Result<Option<(usize, usize)>>
```

Find an immediate child of the box that starts at `parent_start`.

#### Arguments

* `buf` - Destination buffer resized to `size`.
* `parent_start` - Numeric `parent_start` value for this call.
* `parent_end` - Numeric `parent_end` value for this call.
* `fourcc` - `fourcc` input for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

### find_child_in_range

```rust
pub fn find_child_in_range(
    buf: &[u8],
    start: usize,
    end: usize,
    fourcc: &[u8; 4],
) -> Result<Option<(usize, usize)>>
```

Find a box of `fourcc` among the siblings laid out in `[start, end)`.

#### Arguments

* `buf` - Destination buffer resized to `size`.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `fourcc` - `fourcc` input for this call.

#### Returns

The successful result value for this operation.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

#### Panics

Panics when an internal invariant does not hold.

### splice_replace

```rust
pub fn splice_replace(
    buf: &[u8],
    start: usize,
    end: usize,
    replacement: &[u8],
) -> Result<Vec<u8>>
```

Replace `buf[start..end]` with `replacement`, fixing every ancestor's size.

#### Arguments

* `buf` - Destination buffer resized to `size`.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `replacement` - `replacement` input for this call.

#### Returns

On success, the inner `Vec<u8>` value.

#### Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.
