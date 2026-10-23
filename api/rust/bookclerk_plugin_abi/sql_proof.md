**bookclerk_plugin_abi > sql_proof**

# Module: sql_proof

## Contents

**Structs**

- [`IntegerArithSite`](#integerarithsite) - One INTEGER arithmetic expression that must not wrap or error on overflow.
- [`PhysicalAccess`](#physicalaccess) - Physical table/column access used for authorization.
- [`ResolvedAssignment`](#resolvedassignment) - Destination assignment `lhs = rhs` (INSERT, UPDATE, …).
- [`ResolvedStatement`](#resolvedstatement) - One resolved typed proof bound to an exact canonical statement.
- [`SqlSpan`](#sqlspan) - Byte span in the exact canonical SQL string a proof is bound to.
- [`TextCollateSite`](#textcollatesite) - One TEXT expression that PostgreSQL must collate as `"C"`.

**Enums**

- [`IntegerArithKind`](#integerarithkind) - INTEGER `+` `-` `*` `abs` site lowered to overflow → NULL.
- [`SchemaAction`](#schemaaction) - CREATE/DROP action recorded on a proof.

**Functions**

- [`assert_proof_matches_sql`](#assert_proof_matches_sql) - Require `proof` to be bound to `sql`.

**Constants**

- [`PHYSICAL_STAR_COLUMN`](#physical_star_column) - Column name recorded for `SELECT *` / `alias.*` / `RETURNING *` on a physical table.

---

## bookclerk_plugin_abi::sql_proof::IntegerArithKind

*Enum*

INTEGER `+` `-` `*` `abs` site lowered to overflow → NULL.

**Variants:**
- `Add` - `lhs + rhs`
- `Sub` - `lhs - rhs`
- `Mul` - `lhs * rhs`
- `Abs` - `abs(arg)` (`lhs` is the argument span)

**Traits:** Copy, Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> IntegerArithKind`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &IntegerArithKind) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::sql_proof::IntegerArithSite

*Struct*

One INTEGER arithmetic expression that must not wrap or error on overflow.

**Fields:**
- `full: SqlSpan` - Full expression span (`a + b` or `abs(n)`).
- `lhs: SqlSpan` - Left operand (or `abs` argument).
- `rhs: SqlSpan` - Right operand (`abs` repeats `lhs`).
- `kind: IntegerArithKind` - Operator.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &IntegerArithSite) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> IntegerArithSite`



## bookclerk_plugin_abi::sql_proof::PHYSICAL_STAR_COLUMN

*Constant*: `&str`

Column name recorded for `SELECT *` / `alias.*` / `RETURNING *` on a physical table.



## bookclerk_plugin_abi::sql_proof::PhysicalAccess

*Struct*

Physical table/column access used for authorization.

**Fields:**
- `table: String` - Folded physical table name.
- `column: Option<String>` - Folded column name when the access is column-specific.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PhysicalAccess`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &PhysicalAccess) -> $crate::option::Option<$crate::cmp::Ordering>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PhysicalAccess) -> bool`
- **Ord**
  - `fn cmp(self: &Self, other: &PhysicalAccess) -> $crate::cmp::Ordering`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::sql_proof::ResolvedAssignment

*Struct*

Destination assignment `lhs = rhs` (INSERT, UPDATE, …).

**Fields:**
- `table: String` - Physical table of the destination column.
- `column: String` - Destination column.
- `dest: super::SqlType` - Destination type.
- `source: super::SqlType` - Unified source type.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ResolvedAssignment) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ResolvedAssignment`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::sql_proof::ResolvedStatement

*Struct*

One resolved typed proof bound to an exact canonical statement.

**Fields:**
- `statement_hash: String` - SHA-256 hex of the exact canonical SQL this proof claims.
- `output_columns: Vec<(String, super::SqlType)>` - SELECT / RETURNING / VALUES output columns in order.
- `physical_accesses: Vec<PhysicalAccess>` - Physical tables/columns referenced (authorization).
- `assignments: Vec<ResolvedAssignment>` - Mutation destination assignments.
- `text_collate_sites: Vec<TextCollateSite>` - TEXT expression spans for `COLLATE "C"` (already excluding dest/FROM/AS).
- `integer_arith_sites: Vec<IntegerArithSite>` - INTEGER overflow sites (`+` `-` `*` `abs`).
- `functions: Vec<String>` - Function names invoked in code spans (folded), for authorization.
- `schema_action: SchemaAction` - DDL action + fingerprint.

**Methods:**

- `fn bound_empty(sql: &str) -> Self` - Proof for `sql` with empty semantic payloads (wrapper DDL/DML still hash-bound).
- `fn proves(self: &Self, sql: &str) -> bool` - True when `sql` is the exact statement this proof was built from.
- `fn validate_for(self: &Self, sql: &str) -> crate::Result<()>` - Rejects a hash-correct proof whose TEXT/arithmetic sidecar cannot be applied.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ResolvedStatement) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ResolvedStatement`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::sql_proof::SchemaAction

*Enum*

CREATE/DROP action recorded on a proof.

**Variants:**
- `None` - Not DDL.
- `Create{ schema: Box<super::sql_types::CreateTableSchema>, fingerprint: String, noop: bool }` - `CREATE TABLE` — `noop` when the durable fingerprint already matches.
- `Drop{ table: String }` - `DROP TABLE`.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SchemaAction`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SchemaAction) -> bool`



## bookclerk_plugin_abi::sql_proof::SqlSpan

*Struct*

Byte span in the exact canonical SQL string a proof is bound to.

**Fields:**
- `start: usize` - Inclusive start byte offset.
- `end: usize` - Exclusive end byte offset.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SqlSpan`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SqlSpan) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::sql_proof::TextCollateSite

*Struct*

One TEXT expression that PostgreSQL must collate as `"C"`.

**Fields:**
- `span: SqlSpan` - Identifier or string-literal span in canonical SQL.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TextCollateSite`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &TextCollateSite) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::sql_proof::assert_proof_matches_sql

*Function*

Require `proof` to be bound to `sql`.

# Errors

Returns when the hashes differ (transformed or reindexed SQL).

```rust
fn assert_proof_matches_sql(proof: &ResolvedStatement, sql: &str) -> crate::Result<()>
```



