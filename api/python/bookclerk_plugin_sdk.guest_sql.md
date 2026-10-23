# bookclerk\_plugin\_sdk.guest\_sql

Guest SQL classification — must match host ``guest_statement_kind``.

#### guest\_statement\_kind

```python
def guest_statement_kind(sql: str) -> DbStatementKind
```

Classify guest SQL kind to match the host canonical request hash.

#### split\_exec\_queries

```python
def split_exec_queries(query: str) -> list[str]
```

Split Cloudflare-style ``exec()`` newline-separated queries.

