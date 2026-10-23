[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / DatabaseBinding

# Interface: DatabaseBinding

Defined in: [db-execute.ts:257](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L257)

Host-mediated typed SQL surface for plugin guests.

Public API is Cloudflare-style `prepare().bind().run()/first()/all()` plus
atomic `batch()`. Raw `execute` stays internal.

## Methods

### batch()

> **batch**(`statements`, `options?`): `Promise`\<[`D1Result`](D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>[]\>

Defined in: [db-execute.ts:275](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L275)

Run prepared statements as one typed atomic batch.

Ordinary bound prepared statements use Cloudflare `run()` semantics
(`resultSelection: rows`) by default. Optional `asRun` / `asFirst` /
`asAll` override per-statement intent. Returns one Cloudflare-shaped
[D1Result](D1Result.md) per statement.

#### Parameters

##### statements

[`PreparedStatement`](PreparedStatement.md)[]

Prepared statements (binds and intent already applied).

##### options?

Retry token.

###### retry?

[`RetryToken`](RetryToken.md)

#### Returns

`Promise`\<[`D1Result`](D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>[]\>

***

### exec()

> **exec**(`query`, `options?`): `Promise`\<[`D1ExecResult`](D1ExecResult.md)\>

Defined in: [db-execute.ts:285](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L285)

Execute raw SQL without bind parameters (Cloudflare `D1Database.exec`).

#### Parameters

##### query

`string`

Canonical SQL string.

##### options?

Optional retry token.

###### retry?

[`RetryToken`](RetryToken.md)

#### Returns

`Promise`\<[`D1ExecResult`](D1ExecResult.md)\>

***

### execute()

> **execute**(`batch`, `options?`): `Promise`\<[`ExecuteReply`](ExecuteReply.md)\>

Defined in: [db-execute.ts:292](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L292)

Internal typed-batch transport. Prefer [DatabaseBinding.prepare](#prepare).

#### Parameters

##### batch

[`TypedDbStatement`](TypedDbStatement.md)[]

Ordered statements with typed `DbValue` parameters.

##### options?

Optional retry token.

###### retry?

[`RetryToken`](RetryToken.md)

#### Returns

`Promise`\<[`ExecuteReply`](ExecuteReply.md)\>

***

### prepare()

> **prepare**(`sql`): [`PreparedStatement`](PreparedStatement.md)

Defined in: [db-execute.ts:263](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L263)

Prepare one canonical-SQL statement (`?` placeholders).

#### Parameters

##### sql

`string`

Host-mediated SQL. Kind and bounds are derived by the host.

#### Returns

[`PreparedStatement`](PreparedStatement.md)
