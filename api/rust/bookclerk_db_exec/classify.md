**bookclerk_db_exec > classify**

# Module: classify

## Contents

**Enums**

- [`DbErrorClass`](#dberrorclass) - Portable Bookclerk SQL error category.

**Functions**

- [`classify_db_err`](#classify_db_err) - Classifies `err` from adapter-stamped codes, not English engine messages.
- [`classify_db_err_message`](#classify_db_err_message) - Classifies an already-formatted adapter/ORM error string.
- [`is_schema_apply_retryable`](#is_schema_apply_retryable) - True when host schema apply may retry after re-reading durable schema state.

---

## bookclerk_db_exec::classify::DbErrorClass

*Enum*

Portable Bookclerk SQL error category.

**Variants:**
- `Unavailable` - Retry the same operation after re-reading durable state.
- `Conflict` - Uniqueness or duplicate-object; re-read state, do not blindly retry DDL.
- `Other` - Permanent or unclassified failure (including FK / CHECK / NOT NULL).

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &DbErrorClass) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> DbErrorClass`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_db_exec::classify::classify_db_err

*Function*

Classifies `err` from adapter-stamped codes, not English engine messages.

```rust
fn classify_db_err(err: &sea_orm::DbErr) -> DbErrorClass
```



## bookclerk_db_exec::classify::classify_db_err_message

*Function*

Classifies an already-formatted adapter/ORM error string.

```rust
fn classify_db_err_message(message: &str) -> DbErrorClass
```



## bookclerk_db_exec::classify::is_schema_apply_retryable

*Function*

True when host schema apply may retry after re-reading durable schema state.

Uniqueness / duplicate-object races and transient unavailability qualify.
Semantic integrity failures do not.

```rust
fn is_schema_apply_retryable(err: &sea_orm::DbErr) -> bool
```



