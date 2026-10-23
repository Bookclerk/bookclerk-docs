**bookclerk_mp4 > edit**

# Module: edit

## Contents

**Functions**

- [`find_box_range`](#find_box_range) - Find the first box of `fourcc` anywhere under the well-known containers.
- [`find_child_in_range`](#find_child_in_range) - Find a box of `fourcc` among the siblings laid out in `[start, end)`.
- [`find_direct_child`](#find_direct_child) - Find an immediate child of the box that starts at `parent_start`.
- [`splice_replace`](#splice_replace) - Replace `buf[start..end]` with `replacement`, fixing every ancestor's size.

---

## bookclerk_mp4::edit::find_box_range

*Function*

Find the first box of `fourcc` anywhere under the well-known containers.

Returns the byte range `[start, end)` of the whole box, header included.

# Arguments

* `buf` - Destination buffer resized to `size`.
* `fourcc` - Numeric `fourcc` value for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn find_box_range(buf: &[u8], fourcc: &[u8; 4]) -> crate::error::Result<Option<(usize, usize)>>
```



## bookclerk_mp4::edit::find_child_in_range

*Function*

Find a box of `fourcc` among the siblings laid out in `[start, end)`.

# Arguments

* `buf` - Destination buffer resized to `size`.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `fourcc` - `fourcc` input for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

# Panics

Panics when an internal invariant does not hold.

```rust
fn find_child_in_range(buf: &[u8], start: usize, end: usize, fourcc: &[u8; 4]) -> crate::error::Result<Option<(usize, usize)>>
```



## bookclerk_mp4::edit::find_direct_child

*Function*

Find an immediate child of the box that starts at `parent_start`.

# Arguments

* `buf` - Destination buffer resized to `size`.
* `parent_start` - Numeric `parent_start` value for this call.
* `parent_end` - Numeric `parent_end` value for this call.
* `fourcc` - `fourcc` input for this call.

# Returns

The successful result value for this operation.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn find_direct_child(buf: &[u8], parent_start: usize, parent_end: usize, fourcc: &[u8; 4]) -> crate::error::Result<Option<(usize, usize)>>
```



## bookclerk_mp4::edit::splice_replace

*Function*

Replace `buf[start..end]` with `replacement`, fixing every ancestor's size.

# Arguments

* `buf` - Destination buffer resized to `size`.
* `start` - Numeric `start` value for this call.
* `end` - Numeric `end` value for this call.
* `replacement` - `replacement` input for this call.

# Returns

On success, the inner `Vec<u8>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn splice_replace(buf: &[u8], start: usize, end: usize, replacement: &[u8]) -> crate::error::Result<Vec<u8>>
```



