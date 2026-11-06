# bookclerk_plugin_sdk.guest_sql

Guest SQL classification — must match host `guest_statement_kind`.

| Name | Kind | Summary |
| --- | --- | --- |
| [`guest_statement_kind`](#guest_statement_kind) | function | Classify guest SQL kind to match the host canonical request hash. |
| [`split_exec_queries`](#split_exec_queries) | function | Split Cloudflare-style `exec()` newline-separated queries. |
| [`DbStatementKind`](#dbstatementkind) | constant |  |

## Functions

### guest_statement_kind

```python
def guest_statement_kind(sql: str) -> DbStatementKind
```

Classify guest SQL kind to match the host canonical request hash.

### split_exec_queries

```python
def split_exec_queries(query: str) -> list[str]
```

Split Cloudflare-style `exec()` newline-separated queries.

## Constants

### DbStatementKind

```python
DbStatementKind = Literal['execute', 'select', 'returning']
```
