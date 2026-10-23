**bookclerk_plugin_abi > sql_text**

# Module: sql_text

## Contents

**Structs**

- [`SqlFnCall`](#sqlfncall) - One helper call found in canonical SQL (code spans only).

**Enums**

- [`LikePatternSrc`](#likepatternsrc) - Proven LIKE pattern sources in one canonical statement.

**Functions**

- [`admitted_bookclerk_sql_samples`](#admitted_bookclerk_sql_samples) - Grammar-aware BookclerkSQL samples for fuzz corpora and differential tests.
- [`canonical_statements_checksum`](#canonical_statements_checksum) - Length-prefixed SHA-256 of an ordered canonical statement list.
- [`glob_expanded_like_pattern_bytes`](#glob_expanded_like_pattern_bytes) - UTF-8 byte length of a sqlite-family GLOB pattern after Bookclerk LIKE→GLOB
- [`like_pattern_sources`](#like_pattern_sources) - Collects LIKE / NOT LIKE pattern sources with proven lengths when possible.
- [`require_function_args_within`](#require_function_args_within) - Enforces `crate::DbCapabilities::max_function_args` on non-chunkable helpers.
- [`require_like_patterns_within`](#require_like_patterns_within) - Enforces `crate::DbCapabilities::max_pattern_bytes` on LIKE patterns.
- [`require_portable_text`](#require_portable_text) - Rejects BookclerkSQL `TEXT` that contains U+0000.
- [`require_portable_text_binds`](#require_portable_text_binds) - Rejects U+0000 in every BookclerkSQL `TEXT` bind.
- [`sql_v1_function_calls`](#sql_v1_function_calls) - Collects helper calls in code spans (strings/comments are skipped).
- [`sql_v1_helper_is_chunkable`](#sql_v1_helper_is_chunkable) - Helpers adapters may nest so each physical call stays inside
- [`sql_v1_pack_statements`](#sql_v1_pack_statements) - Splits a BookclerkSQL pack into statements using the SQL-v1 lexer.
- [`sqlite_family_like_divmod_insert_upper_bound`](#sqlite_family_like_divmod_insert_upper_bound) - Upper bound on sqlite-family mechanical lowering length for a canonical
- [`sqlite_family_mechanical_len_upper_bound`](#sqlite_family_mechanical_len_upper_bound) - Mechanical sqlite-family length bound for one canonical statement, counting
- [`text_contains_nul`](#text_contains_nul) - True when `text` contains U+0000.

**Constants**

- [`D1_PHYSICAL_LIKE_GLOB_PATTERN_BYTES`](#d1_physical_like_glob_pattern_bytes) - Physical D1 LIKE/GLOB pattern cap in bytes.
- [`D1_PORTABLE_LIKE_PATTERN_BYTES`](#d1_portable_like_pattern_bytes) - Conservative portable LIKE pattern cap for D1 while LIKE lowers to GLOB.
- [`LIKE_GLOB_PATTERN_WRAP_BYTES`](#like_glob_pattern_wrap_bytes) - Bytes added around a LIKE pattern by [`LIKE_GLOB_WRAP_PREFIX`](sql_text.md#bookclerk_plugin_abisql_textlike_glob_wrap_prefix) /
- [`LIKE_GLOB_REWRITE_OVERHEAD`](#like_glob_rewrite_overhead) - Worst-case extra bytes per `LIKE` / `NOT LIKE` after sqlite-family GLOB wrapping.
- [`LIKE_GLOB_WRAP_PREFIX`](#like_glob_wrap_prefix) - Prefix of the sqlite-family LIKE→GLOB `replace(...)` wrap (before the pattern).
- [`LIKE_GLOB_WRAP_SUFFIX`](#like_glob_wrap_suffix) - Suffix of the sqlite-family LIKE→GLOB `replace(...)` wrap (after the pattern).
- [`LIKE_TO_GLOB_KEYWORD_EXTRA`](#like_to_glob_keyword_extra) - Extra byte when `LIKE` is immediately followed by its pattern (no trivia):
- [`SQLITE_FAMILY_DIV_MOD_NULLIF_EXTRA`](#sqlite_family_div_mod_nullif_extra) - Extra bytes for one `/` or `%` rewritten to `NULLIF(operand, 0)`.
- [`SQLITE_FAMILY_INSERT_OR_IGNORE_MAX_EXTRA`](#sqlite_family_insert_or_ignore_max_extra) - Conservative extra bytes for one `INSERT OR IGNORE` unique-conflict rewrite

---

## bookclerk_plugin_abi::sql_text::D1_PHYSICAL_LIKE_GLOB_PATTERN_BYTES

*Constant*: `u32`

Physical D1 LIKE/GLOB pattern cap in bytes.



## bookclerk_plugin_abi::sql_text::D1_PORTABLE_LIKE_PATTERN_BYTES

*Constant*: `u32`

Conservative portable LIKE pattern cap for D1 while LIKE lowers to GLOB.

Cloudflare D1 allows 50 bytes in a LIKE/GLOB pattern. Worst-case expansion
of `[` / `*` / `?` is 3×, so `floor(50/3) = 16`.



## bookclerk_plugin_abi::sql_text::LIKE_GLOB_PATTERN_WRAP_BYTES

*Constant*: `usize`

Bytes added around a LIKE pattern by [`LIKE_GLOB_WRAP_PREFIX`](sql_text.md#bookclerk_plugin_abisql_textlike_glob_wrap_prefix) /
[`LIKE_GLOB_WRAP_SUFFIX`](sql_text.md#bookclerk_plugin_abisql_textlike_glob_wrap_suffix) (the pattern itself is unchanged).



## bookclerk_plugin_abi::sql_text::LIKE_GLOB_REWRITE_OVERHEAD

*Constant*: `usize`

Worst-case extra bytes per `LIKE` / `NOT LIKE` after sqlite-family GLOB wrapping.



## bookclerk_plugin_abi::sql_text::LIKE_GLOB_WRAP_PREFIX

*Constant*: `&str`

Prefix of the sqlite-family LIKE→GLOB `replace(...)` wrap (before the pattern).

Must stay identical to the GLOB wrap in `bookclerk-db-exec`.



## bookclerk_plugin_abi::sql_text::LIKE_GLOB_WRAP_SUFFIX

*Constant*: `&str`

Suffix of the sqlite-family LIKE→GLOB `replace(...)` wrap (after the pattern).



## bookclerk_plugin_abi::sql_text::LIKE_TO_GLOB_KEYWORD_EXTRA

*Constant*: `usize`

Extra byte when `LIKE` is immediately followed by its pattern (no trivia):
the rewrite emits `GLOB ` (5) in place of `LIKE` (4).



## bookclerk_plugin_abi::sql_text::LikePatternSrc

*Enum*

Proven LIKE pattern sources in one canonical statement.

**Variants:**
- `Literal(String)` - `'literal'` (possibly concatenated with `||`).
- `Bind(usize)` - Positional `?` bind (0-based bind index in this statement).
- `Unproven` - Pattern expression whose length cannot be proven.
- `Null` - SQL `NULL` (no pattern bytes).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LikePatternSrc`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &LikePatternSrc) -> bool`



## bookclerk_plugin_abi::sql_text::SQLITE_FAMILY_DIV_MOD_NULLIF_EXTRA

*Constant*: `usize`

Extra bytes for one `/` or `%` rewritten to `NULLIF(operand, 0)`.



## bookclerk_plugin_abi::sql_text::SQLITE_FAMILY_INSERT_OR_IGNORE_MAX_EXTRA

*Constant*: `usize`

Conservative extra bytes for one `INSERT OR IGNORE` unique-conflict rewrite
(conflict clause plus optional `SELECT` source wrap).



## bookclerk_plugin_abi::sql_text::SqlFnCall

*Struct*

One helper call found in canonical SQL (code spans only).

**Fields:**
- `name: String` - Folded helper name (`json_object`, `replace`, …).
- `arg_count: usize` - Top-level argument count (`json_object('k', 1)` is 2).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SqlFnCall`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SqlFnCall) -> bool`



## bookclerk_plugin_abi::sql_text::admitted_bookclerk_sql_samples

*Function*

Grammar-aware BookclerkSQL samples for fuzz corpora and differential tests.

Statements stay inside the SQL-v1 grammar (canonical `?`, `LIKE`,
`INSERT OR IGNORE`). TEXT literals are portable UTF-8 without U+0000.
LIKE patterns stay within [`D1_PORTABLE_LIKE_PATTERN_BYTES`](sql_text.md#bookclerk_plugin_abisql_textd1_portable_like_pattern_bytes).

```rust
fn admitted_bookclerk_sql_samples(seed: u64, count: usize) -> Vec<String>
```



## bookclerk_plugin_abi::sql_text::canonical_statements_checksum

*Function*

Length-prefixed SHA-256 of an ordered canonical statement list.

Statement boundaries are part of the digest: joining with `;` then hashing
would collide with literals such as `DEFAULT 'a;b'`.

```rust
fn canonical_statements_checksum(statements: &[&str]) -> String
```



## bookclerk_plugin_abi::sql_text::glob_expanded_like_pattern_bytes

*Function*

UTF-8 byte length of a sqlite-family GLOB pattern after Bookclerk LIKE→GLOB
metacharacter escaping (`[` `*` `?` become three bytes).

D1's physical GLOB/LIKE pattern cap is 50 bytes. Adapters that perform this
expansion must advertise `super::DbCapabilities::max_pattern_bytes` small
enough that the expanded pattern still fits.

```rust
fn glob_expanded_like_pattern_bytes(like_pattern: &str) -> usize
```



## bookclerk_plugin_abi::sql_text::like_pattern_sources

*Function*

Collects LIKE / NOT LIKE pattern sources with proven lengths when possible.

# Errors

Returns `PluginError::invalid_params` when a string, quoted identifier,
or block comment is unterminated.

```rust
fn like_pattern_sources(sql: &str) -> crate::Result<Vec<LikePatternSrc>>
```



## bookclerk_plugin_abi::sql_text::require_function_args_within

*Function*

Enforces `crate::DbCapabilities::max_function_args` on non-chunkable helpers.

`min` / `max` / `coalesce` are omitted: adapters nest them so each physical
call stays inside the advertised cap. `json_object` is enforced here: the
portable maximum is already [`crate::SQL_V1_JSON_OBJECT_MAX_ARGS`](sql_types.md#bookclerk_plugin_abisql_typessql_v1_json_object_max_args).

# Errors

Returns `PluginError::invalid_params` when a non-chunkable call exceeds
the cap or the cap is unspecified.

```rust
fn require_function_args_within(sql: &str, max_function_args: u32) -> crate::Result<()>
```



## bookclerk_plugin_abi::sql_text::require_like_patterns_within

*Function*

Enforces `crate::DbCapabilities::max_pattern_bytes` on LIKE patterns.

# Errors

Returns `PluginError::invalid_params` when a proven pattern exceeds the
cap or a LIKE pattern length cannot be proven.

```rust
fn require_like_patterns_within(sql: &str, parameters: &[crate::DbValue], max_pattern_bytes: u32) -> crate::Result<()>
```



## bookclerk_plugin_abi::sql_text::require_portable_text

*Function*

Rejects BookclerkSQL `TEXT` that contains U+0000.

# Errors

Returns `PluginError::invalid_params` when `text` contains NUL.

```rust
fn require_portable_text(text: &str) -> crate::Result<()>
```



## bookclerk_plugin_abi::sql_text::require_portable_text_binds

*Function*

Rejects U+0000 in every BookclerkSQL `TEXT` bind.

# Errors

Returns `PluginError::invalid_params` when a `crate::DbValue::Text` bind
contains NUL.

```rust
fn require_portable_text_binds(parameters: &[crate::DbValue]) -> crate::Result<()>
```



## bookclerk_plugin_abi::sql_text::sql_v1_function_calls

*Function*

Collects helper calls in code spans (strings/comments are skipped).

# Errors

Returns `PluginError::invalid_params` when a string, quoted identifier,
or block comment is unterminated.

```rust
fn sql_v1_function_calls(sql: &str) -> crate::Result<Vec<SqlFnCall>>
```



## bookclerk_plugin_abi::sql_text::sql_v1_helper_is_chunkable

*Function*

Helpers adapters may nest so each physical call stays inside
`crate::DbCapabilities::max_function_args`.

`json_object` is not chunkable: portable SQL-v1 already caps it at
[`crate::SQL_V1_JSON_OBJECT_MAX_ARGS`](sql_types.md#bookclerk_plugin_abisql_typessql_v1_json_object_max_args) (D1's physical limit). Nesting via
`json_patch` is not equivalent (JSON Merge Patch deletes nulls).

```rust
fn sql_v1_helper_is_chunkable(name: &str) -> bool
```



## bookclerk_plugin_abi::sql_text::sql_v1_pack_statements

*Function*

Splits a BookclerkSQL pack into statements using the SQL-v1 lexer.

Semicolons inside string literals, quoted identifiers, line comments, and
block comments do not end a statement. Empty fragments are dropped.

# Errors

Returns `PluginError::invalid_params` when quotes or comments are
unterminated, or when a fragment is not a single SQL-v1 statement.

```rust
fn sql_v1_pack_statements(sql: &str) -> crate::Result<Vec<String>>
```



## bookclerk_plugin_abi::sql_text::sqlite_family_like_divmod_insert_upper_bound

*Function*

Upper bound on sqlite-family mechanical lowering length for a canonical
statement of `canonical_len` bytes (`div`/`mod` NULLIF, `INSERT OR IGNORE`,
LIKE→GLOB). Does not include proof-directed INTEGER overflow wraps.

LIKE→GLOB is the densest per-byte expansion among mechanical rewrites
(`LIKE` is 4 bytes; overhead is [`LIKE_GLOB_REWRITE_OVERHEAD`](sql_text.md#bookclerk_plugin_abisql_textlike_glob_rewrite_overhead)). The count
`canonical_len / 4` is the maximum number of `LIKE` keywords that can appear
in `canonical_len` bytes — not an expansion-factor guess. Remaining bytes
after packing as many `LIKE` keywords as possible are charged as `/` `%`
NULLIF wraps (1-byte operators).

Proof-directed INTEGER overflow CASE wraps are applied from typed proofs
before this mechanical bound; they are not a function of payload length
alone.

```rust
fn sqlite_family_like_divmod_insert_upper_bound(canonical_len: usize) -> usize
```



## bookclerk_plugin_abi::sql_text::sqlite_family_mechanical_len_upper_bound

*Function*

Mechanical sqlite-family length bound for one canonical statement, counting
actual `LIKE` occurrences, `/` `%` operators in code spans, and at most one
`INSERT OR IGNORE` rewrite.

Always `<=` [`sqlite_family_like_divmod_insert_upper_bound`](sql_text.md#bookclerk_plugin_abisql_textsqlite_family_like_divmod_insert_upper_bound)`(sql.len())`
for well-formed packs: the length formula packs the densest rewrite into
every 4-byte window, while this preflight charges only constructs that
lowering actually rewrites. INTEGER overflow wraps are applied from typed
proofs *before* this mechanical bound.

# Errors

Returns `PluginError::invalid_params` when quotes or block comments are
unterminated.

```rust
fn sqlite_family_mechanical_len_upper_bound(sql: &str) -> crate::Result<usize>
```



## bookclerk_plugin_abi::sql_text::text_contains_nul

*Function*

True when `text` contains U+0000.

```rust
fn text_contains_nul(text: &str) -> bool
```



