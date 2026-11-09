# bookclerk_plugin_sdk.testkit

Contract conformance test kit: a plugin database binding like the host's.

A plain in-memory fake accepts anything. On a host, `env.DB` reaches an
isolated unit through the adapter the operator deployed: SQL is scoped to
the plugin's own tables, every batch is one transaction held to the
adapter's advertised caps, and a result past a cap is a typed
`limit_exceeded` error. [`create_test_database`](#create_test_database) gives a pytest
suite that same binding, backed by the `bookclerk-db-testkit` binary from
the Bookclerk workspace under one of three capability profiles:

- `first-party`: the shipped sqlite and Postgres adapters (256 KiB
  results, streamed reads).
- `d1`: the D1 adapter (smaller payload and statement limits).
- `minimum`: the smallest advertisement a host connects to (4,096-byte
  results, 1,024-byte cells, one row per statement, no streamed reads).

Run plugin database code under all three ([`TEST_DATABASE_PROFILES`](#test_database_profiles)):
what works on the shipped adapters has to work when an adapter advertises
only the minimums.

```python
from bookclerk_plugin_sdk.testkit import TEST_DATABASE_PROFILES, create_test_database

for profile in TEST_DATABASE_PROFILES:
    async with create_test_database(profile=profile, migrations=MIGRATIONS) as db:
        await db.binding.prepare("INSERT INTO notes (id, body) VALUES (?, ?)").bind(1, "hi").run()
        rows = (await db.binding.prepare("SELECT body FROM notes").all()).results
```

The binary is found through the `BOOKCLERK_DB_TESTKIT` environment variable
or the `binary` argument, else as `bookclerk-db-testkit` on `PATH`
(`cargo build -p bookclerk-db-testkit` in the workspace builds it). This
module runs under CPython, not inside an isolate.

| Name | Kind | Summary |
| --- | --- | --- |
| [`TestDatabase`](#testdatabase) | class | A served test binding and the process behind it. |
| [`testkit_binary`](#testkit_binary) | function | Pick the kit binary. |
| [`create_test_database`](#create_test_database) | function | Spawn the kit under `profile` with `migrations` applied. |
| [`TestDatabaseProfile`](#testdatabaseprofile) | constant | One of `TEST_DATABASE_PROFILES`. |
| [`TEST_DATABASE_PROFILES`](#test_database_profiles) | constant |  |

## Classes

### TestDatabase

```python
class TestDatabase
```

A served test binding and the process behind it.

Use it as an async context manager, or call `close` when the test
ends; the in-memory unit goes with the process.

#### TestDatabase.__init__

```python
def __init__(self, profile: TestDatabaseProfile, process: Any, serving: _Serving) -> None
```

Wrap a running kit process.

**Args**

- `profile`: The profile being enforced.
- `process`: The `asyncio` subprocess serving the binding.
- `serving`: Its announcement.

#### TestDatabase.limits

```python
async def limits(self) -> DbCapabilities
```

Read the profile's advertisement: the caps every batch is held to.

**Returns**

The capability advertisement, as `binding.limits()` reports it.

#### TestDatabase.close

```python
async def close(self) -> None
```

Stop the kit process; the in-memory unit goes with it.

## Functions

### testkit_binary

```python
def testkit_binary(binary: str | None=None) -> str
```

Pick the kit binary.

**Args**

- `binary`: Explicit path, when given.

**Returns**

`binary`, else `BOOKCLERK_DB_TESTKIT` from the environment, else the
bare `bookclerk-db-testkit` name resolved on `PATH`.

### create_test_database

```python
async def create_test_database(
    profile: TestDatabaseProfile='minimum',
    migrations: list[dict[str, Any]] | None=None,
    *,
    binary: str | None=None,
) -> TestDatabase
```

Spawn the kit under `profile` with `migrations` applied.

**Args**

- `profile`: Capability profile to enforce (default `minimum`, the strictest).
- `migrations`: The plugin's `database_migrations` registration (each `{"id": ..., "operations": [{"schema": sql} | {"data": sql}, ...]}`), applied the way the host applies it at enrollment.
- `binary`: Path of the `bookclerk-db-testkit` binary; see the module docs for the default.

**Returns**

The served binding and its process; close it when the test ends.

**Raises**

- `ValueError`: When `profile` is not one of [`TEST_DATABASE_PROFILES`](#test_database_profiles).
- `RuntimeError`: When the binary cannot be started, refuses the profile or the registration (its message is included), or exits early.

## Constants

### TestDatabaseProfile

```python
TestDatabaseProfile = Literal['first-party', 'd1', 'minimum']
```

One of [`TEST_DATABASE_PROFILES`](#test_database_profiles).

### TEST_DATABASE_PROFILES

```python
TEST_DATABASE_PROFILES: tuple[TestDatabaseProfile, ...] = ('first-party', 'd1', 'minimum')
```
