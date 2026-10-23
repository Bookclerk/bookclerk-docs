[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / PreparedStatement

# Interface: PreparedStatement

Defined in: [db-execute.ts:208](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L208)

Cloudflare-style prepared statement. Kind, bounds, and request hash are
derived by the trusted host. Only universal [DbValue](../type-aliases/DbValue.md)s are public.

## Methods

### all()

> **all**(`options?`): `Promise`\<[`D1Result`](D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>\>

Defined in: [db-execute.ts:242](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L242)

Execute as a row-returning query. Returns a Cloudflare-shaped [D1Result](D1Result.md).

#### Parameters

##### options?

Optional retry token.

###### retry?

[`RetryToken`](RetryToken.md)

#### Returns

`Promise`\<[`D1Result`](D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>\>

***

### asAll()

> **asAll**(): `PreparedStatement`

Defined in: [db-execute.ts:248](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L248)

Mark row-returning intent for [DatabaseBinding.batch](DatabaseBinding.md#batch).

#### Returns

`PreparedStatement`

***

### asFirst()

> **asFirst**(): `PreparedStatement`

Defined in: [db-execute.ts:246](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L246)

Mark `maxRows = 1` row intent for [DatabaseBinding.batch](DatabaseBinding.md#batch).

#### Returns

`PreparedStatement`

***

### asRun()

> **asRun**(): `PreparedStatement`

Defined in: [db-execute.ts:244](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L244)

Mark DML intent for [DatabaseBinding.batch](DatabaseBinding.md#batch).

#### Returns

`PreparedStatement`

***

### bind()

> **bind**(...`values`): `PreparedStatement`

Defined in: [db-execute.ts:214](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L214)

Replace bound parameters.

#### Parameters

##### values

...[`DbValue`](../type-aliases/DbValue.md)[]

Universal `DbValue`s only.

#### Returns

`PreparedStatement`

***

### first()

> **first**(`colName?`, `options?`): `Promise`\<[`DbValue`](../type-aliases/DbValue.md) \| `Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\> \| `null`\>

Defined in: [db-execute.ts:227](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L227)

First row as a name→value map, or one column when `colName` is set.

#### Parameters

##### colName?

`string`

Optional column name (Cloudflare `first(colName)`).

##### options?

Optional retry token.

###### retry?

[`RetryToken`](RetryToken.md)

#### Returns

`Promise`\<[`DbValue`](../type-aliases/DbValue.md) \| `Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\> \| `null`\>

***

### raw()

> **raw**(`options?`): `Promise`\<[`DbValue`](../type-aliases/DbValue.md)[][]\>

Defined in: [db-execute.ts:236](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L236)

Positional cell values per row (Cloudflare `raw()`).

#### Parameters

##### options?

Optional retry token.

###### retry?

[`RetryToken`](RetryToken.md)

#### Returns

`Promise`\<[`DbValue`](../type-aliases/DbValue.md)[][]\>

***

### run()

> **run**(`options?`): `Promise`\<[`D1Result`](D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>\>

Defined in: [db-execute.ts:220](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L220)

Execute as DML. Returns a Cloudflare-shaped [D1Result](D1Result.md).

#### Parameters

##### options?

Optional retry token.

###### retry?

[`RetryToken`](RetryToken.md)

#### Returns

`Promise`\<[`D1Result`](D1Result.md)\<`Record`\<`string`, [`DbValue`](../type-aliases/DbValue.md)\>\>\>
