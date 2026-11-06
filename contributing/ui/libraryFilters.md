# libraryFilters

## Interfaces

### LibraryFilter

Active library filter kind plus selected value.

#### Properties

##### kind

```ts
kind: FilterKind;
```

##### value

```ts
value: string;
```

Selected value within the kind (`all` / empty = no value filter).

## Type Aliases

### FilterKind

```ts
type FilterKind = 
  | "all"
  | "authors"
  | "series"
  | "genres"
  | "tags"
  | "narrators"
  | "publishers"
  | "sources"
  | "progress"
  | "status";
```

AudioBookshelf-style library filter categories.

***

### SortKey

```ts
type SortKey = "title" | "author" | "series" | "purchased" | "length" | "added";
```

Client-side library table sort keys.

## Variables

### FILTER\_KINDS

```ts
const FILTER_KINDS: object[];
```

UI labels for [FilterKind](#filterkind) select options.

#### Type Declaration

| Name | Type |
| ------ | ------ |
| `label` | `string` |
| `value` | [`FilterKind`](#filterkind) |

***

### SORT\_OPTIONS

```ts
const SORT_OPTIONS: object[];
```

UI labels for [SortKey](#sortkey) select options.

#### Type Declaration

| Name | Type |
| ------ | ------ |
| `label` | `string` |
| `value` | [`SortKey`](#sortkey) |

## Functions

### bookMatchesFilter()

```ts
function bookMatchesFilter(book, filter): boolean;
```

Returns whether a book matches the active library filter.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `book` | [`BookRecord`](api.md#bookrecord) | Library row. |
| `filter` | [`LibraryFilter`](#libraryfilter) | Kind + value (`all` / empty value always matches). |

#### Returns

`boolean`

True when the book should remain visible.

***

### compareSeriesIndex()

```ts
function compareSeriesIndex(a, b): number;
```

Ascending series-number compare; missing indexes sort last.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `a` | `string` \| `null` \| `undefined` | Left series index. |
| `b` | `string` \| `null` \| `undefined` | Right series index. |

#### Returns

`number`

Negative / zero / positive compare result.

***

### filterValuesForKind()

```ts
function filterValuesForKind(books, kind): object[];
```

Builds distinct filter value options for a kind from the loaded books.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `books` | [`BookRecord`](api.md#bookrecord)[] | Library rows to mine for facet values. |
| `kind` | [`FilterKind`](#filterkind) | Filter category. |

#### Returns

`object`[]

Value/label pairs (empty for `all`).

***

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

***

### parseSeriesIndex()

```ts
function parseSeriesIndex(raw): number | null;
```

Parses a series position like `3`, `#3`, `3.5`, or `Book 3` for numeric sort.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` \| `null` \| `undefined` | Series index string from the store. |

#### Returns

`number` \| `null`

Parsed number, or `null` when absent / non-numeric.

***

### sortBooks()

```ts
function sortBooks(books, sort): BookRecord[];
```

Returns a new array of books sorted by the given key.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `books` | [`BookRecord`](api.md#bookrecord)[] | Library rows. |
| `sort` | [`SortKey`](#sortkey) | Sort key. |

#### Returns

[`BookRecord`](api.md#bookrecord)[]

Sorted copy (stable enough for UI tables).
