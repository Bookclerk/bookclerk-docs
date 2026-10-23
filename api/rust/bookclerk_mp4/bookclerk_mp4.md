**bookclerk_mp4**

# Module: bookclerk_mp4

## Contents

**Modules**

- [`boxutil`](#boxutil) - Low-level big-endian readers and MP4 box walking.
- [`edit`](#edit) - Editing a `moov` that is already in memory: find a box, replace a child, and
- [`fixture`](#fixture) - Synthetic progressive MP4 files for tests.

---

## Module: boxutil

Low-level big-endian readers and MP4 box walking.



## Module: edit

Editing a `moov` that is already in memory: find a box, replace a child, and
keep every enclosing size field consistent.

Rewriting a sample table means growing or shrinking a box that several
ancestors declare the size of, so a splice here is never a plain
`Vec::splice`: [`splice_replace`](edit.md#bookclerk_mp4editsplice_replace) walks back up the tree and patches each
ancestor's size by the same delta.



## Module: fixture

Synthetic progressive MP4 files for tests.

Behind the `fixtures` feature so it never reaches a release build. Tests in
this crate and in the store plugins both need real files to parse — a plugin
proving its decrypt round-trips through [`crate::remux_progressive`](remux.md#bookclerk_mp4remuxremux_progressive) needs
one that carries its sample entry — and a writer they share is one fewer
copy of ISO-BMFF layout to keep correct.



