# bookclerk_resolve::fuzzing

Fuzz input decoding and the oracle the `library_resolve` target checks.

[`run_case`](#run_case) turns arbitrary bytes into a sequence of plugin-reported
listings drawn from small pools (so identifiers, titles and authors
collide often), records each one into a [`MemoryCatalog`](memory.md#memorycatalog), and checks
the resolver's invariants after every step:

- one strong identifier never maps to two editions, and no edition holds
  two values of one strong scheme in one marketplace;
- a weak identifier alone never joins an edition of another media kind,
  work kind or an incompatible expression;
- strong identifiers that name different editions never merge them;
  the listing reaches one and the conflict is recorded;
- a work found by identifier or matcher has the listing's kind, and an
  episode matched by text shares its parent show, while a book or a show
  never matches an episode by text, and no work becomes its own parent;
- a catalog listing (recorded with no holding) never gains a holding
  and never announces a holding or its own new link;
- resolving the same input twice changes nothing.

The fuzz target and the replay test in `tests/fuzz_corpus.rs` both call
[`run_case`](#run_case), so a corpus file replays exactly what the fuzzer ran.

## Functions

### run_case

```rust
pub fn run_case(data: &[u8]) -> Result<usize, String>
```

Resolves the listings `data` decodes to and checks every invariant.

#### Arguments

* `data` - Arbitrary bytes: a fuzz input or a corpus file.

#### Returns

The number of listings recorded.

#### Errors

Returns a description of the first broken invariant.
