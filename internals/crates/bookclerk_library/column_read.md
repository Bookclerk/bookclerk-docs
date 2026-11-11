# bookclerk_library::column_read

Column reads that stay correct on the host's proxied store connection.

The host reaches every database plugin through SeaORM's proxy connection,
and SeaORM's `ProxyRow` keeps a row as a `BTreeMap<String, Value>`. A
positional read therefore sees the columns in alphabetical order by name,
not in `SELECT` order: `select_only().column(b).column(a).into_tuple()`
fills the tuple with `a` first (#854). Every adapter returns its columns in
order; the order is lost when the host builds the proxy row.

`crates/bookclerk-library/clippy.toml` disallows the positional readers
(`Select::into_tuple`, `QueryResult::try_get_by_index`,
`QueryResult::try_get_many_by_index` and their trait forms), so host code
reads by name instead:

- several columns: a `#[derive(FromQueryResult)]` struct read with
  `into_model::<T>()`, which looks each field up by its column name;
- one column of an entity query: [`SelectColumnExt::into_column`](#selectcolumnextinto_column);
- the only column of a raw statement's row: [`only_column`](#only_column).

| Item | Kind | Summary |
| --- | --- | --- |
| [`SelectColumnExt`](#selectcolumnext) | trait | Reads one column of an entity query without a positional tuple. |
| [`only_column`](#only_column) | function | Reads the only column of a raw statement's row. |

## Traits

### SelectColumnExt

```rust
pub trait SelectColumnExt {
    fn into_column<T, C>(self, column: C) -> Selector<SelectGetableTuple<T>>
    where
        T: TryGetable,
        C: ColumnTrait,;
}
```

Reads one column of an entity query without a positional tuple.

#### SelectColumnExt::into_column

```rust
fn into_column<T, C>(self, column: C) -> Selector<SelectGetableTuple<T>>
where
    T: TryGetable,
    C: ColumnTrait,
```

Selects only `column` and reads it into `T`.

Any columns selected earlier are replaced, so the result always has
exactly one column and its position cannot be mistaken. Filters,
ordering, `DISTINCT` and limits applied before the call are kept.

`T` is a single value ([`TryGetable`](https://docs.rs/sea_orm/latest/sea_orm/executor/query/trait.TryGetable.html)), never a tuple: a read of
several columns is a `FromQueryResult` struct instead.

##### Arguments

* `column` - The one column to read.

##### Returns

A selector whose items are the column's values.

Implemented by: `Select<E>`

## Functions

### only_column

```rust
pub fn only_column<T>(row: &QueryResult) -> Result<T, DbErr>
where
    T: TryGetable,
```

Reads the only column of a raw statement's row.

The value is looked up by the column's own name, so the read does not
depend on column order.

#### Arguments

* `row` - A row of a statement that selects exactly one column, such as
  `SELECT COUNT(*) FROM …`.

#### Returns

The column's value.

#### Errors

Returns `DbErr::Type` when the row does not have exactly one column
(read several columns by name instead), or the decode error when the value
is not a `T`.
