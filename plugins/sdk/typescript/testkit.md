# testkit

Contract conformance test kit: a plugin database binding that behaves
like the one a deployed host grants.

A plain in-memory fake accepts anything. On a host, `env.DB` reaches an
isolated unit through the adapter the operator deployed: SQL is scoped to
the plugin's own tables, every batch is one transaction held to the
adapter's advertised caps, and a result past a cap is a typed
`limit_exceeded` error. [createTestDatabase](#createtestdatabase) gives a Vitest or
`node:test` suite that same binding, backed by the `bookclerk-db-testkit`
binary from the Bookclerk workspace under one of three capability
profiles:

- `first-party`: the shipped sqlite and Postgres adapters (256 KiB
  results, streamed reads).
- `d1`: the D1 adapter (smaller payload and statement limits).
- `minimum`: the smallest advertisement a host connects to (4,096-byte
  results, 1,024-byte cells, one row per statement, no streamed reads).

Run plugin database code under all three ([TEST\_DATABASE\_PROFILES](#test_database_profiles)):
what works on the shipped adapters has to work when an adapter
advertises only the minimums.

```ts
import { createTestDatabase, TEST_DATABASE_PROFILES } from "@bookclerk/plugin-sdk/testkit";

for (const profile of TEST_DATABASE_PROFILES) {
  const db = await createTestDatabase({ profile, migrations: MIGRATIONS });
  try {
    await db.binding.prepare("INSERT INTO notes (id, body) VALUES (?, ?)").bind(1, "hi").run();
    const { results } = await db.binding.prepare("SELECT body FROM notes").all();
  } finally {
    await db.close();
  }
}
```

The binary is found through the `BOOKCLERK_DB_TESTKIT` environment
variable or `options.binary`, else as `bookclerk-db-testkit` on `PATH`
(`cargo build -p bookclerk-db-testkit` in the workspace builds it). This
module runs under Node, not inside an isolate.

## Interfaces

### TestDatabase

A served test binding and the process behind it.

#### Properties

##### binding

```ts
readonly binding: DatabaseBinding;
```

The D1-shaped binding, as `env.<BINDING>` on a host.

##### profile

```ts
readonly profile: "first-party" | "d1" | "minimum";
```

The profile being enforced.

#### Methods

##### close()

```ts
close(): Promise<void>;
```

Stops the kit process; the in-memory unit goes with it.

###### Returns

`Promise`\<`void`\>

Once the process has exited.

##### limits()

```ts
limits(): Promise<DbCapabilities>;
```

The profile's advertisement: the caps every batch is held to.

###### Returns

`Promise`\<[`DbCapabilities`](generated.md#dbcapabilities)\>

The capability advertisement, as `binding.limits()` reports it.

***

### TestDatabaseOptions

Options for [createTestDatabase](#createtestdatabase).

#### Properties

##### binary?

```ts
binary?: string;
```

Path of the `bookclerk-db-testkit` binary; see the module docs for the default.

##### migrations?

```ts
migrations?: PluginMigration[];
```

The plugin's `databaseMigrations` registration, applied before the
binding is handed out the way the host applies it at enrollment.

##### profile?

```ts
profile?: "first-party" | "d1" | "minimum";
```

Capability profile to enforce (default `minimum`, the strictest).

## Type Aliases

### TestDatabaseProfile

```ts
type TestDatabaseProfile = typeof TEST_DATABASE_PROFILES[number];
```

One of [TEST\_DATABASE\_PROFILES](#test_database_profiles).

## Variables

### TEST\_DATABASE\_PROFILES

```ts
const TEST_DATABASE_PROFILES: readonly ["first-party", "d1", "minimum"];
```

The capability profiles the kit enforces, in the order conformance runs report them.

## Functions

### createTestDatabase()

```ts
function createTestDatabase(options?): Promise<TestDatabase>;
```

Spawns the kit under `options.profile` with `options.migrations` applied
and returns the served binding.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`TestDatabaseOptions`](#testdatabaseoptions) | Profile, registration and binary location. |

#### Returns

`Promise`\<[`TestDatabase`](#testdatabase)\>

The binding and its process; call `close()` when the test ends.

#### Throws

Error when the binary cannot be started, refuses the profile or
  the registration (the kit's message is included), or exits early.

***

### testkitBinary()

```ts
function testkitBinary(binary?): string;
```

Picks the kit binary: the option, then `BOOKCLERK_DB_TESTKIT`, then the
bare name on `PATH`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `binary?` | `string` | Explicit path, when given. |

#### Returns

`string`

The command to spawn.
