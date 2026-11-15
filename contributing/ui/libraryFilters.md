# libraryFilters

## Functions

### formatDate()

```ts
function formatDate(iso): string;
```

Formats an ISO date for library tables.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `iso` | `string` \| `null` | ISO timestamp, or `null`. |

#### Returns

`string`

Locale short date, raw string on parse failure, or `—`.

***

### formatDuration()

```ts
function formatDuration(minutes): string;
```

Formats runtime minutes as `Xh Ym` (or `—` when unknown).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `minutes` | `number` \| `null` | Length in minutes, or `null`. |

#### Returns

`string`

Human-readable duration.
