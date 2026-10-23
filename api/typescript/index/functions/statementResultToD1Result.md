[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / statementResultToD1Result

# Function: statementResultToD1Result()

> **statementResultToD1Result**(`stmt`, `timing`): [`D1Result`](../interfaces/D1Result.md)

Defined in: [db-execute.ts:423](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/db-execute.ts#L423)

Maps one typed statement result to Cloudflare [D1Result](../interfaces/D1Result.md).

## Parameters

### stmt

[`StatementResult`](../interfaces/StatementResult.md)

Per-statement rows, columns, and `rowsAffected`.

### timing

[`DbTiming`](../interfaces/DbTiming.md)

Handler/engine timing from the execute reply.

## Returns

[`D1Result`](../interfaces/D1Result.md)

Cloudflare-shaped `{ success, results, meta }`.
