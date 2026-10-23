# safeHtml

## Type Aliases

### GuidedReviewSection

```ts
type GuidedReviewSection = object;
```

One prompt/answer pair from Audible's guided questionnaire review body.

#### Properties

##### answer

```ts
answer: string;
```

##### question?

```ts
optional question?: string;
```

##### type

```ts
type: string;
```

## Functions

### decodeHtmlEntities()

```ts
function decodeHtmlEntities(raw): string;
```

Decodes HTML entities (`&rsquo;`, `&#8217;`, …) into Unicode text.

Uses a pure string decoder (no `innerHTML` / DOM text round-trip) so the
result is never sourced from reinterpreted DOM HTML.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` | Text that may contain entities. |

#### Returns

`string`

Decoded string. Unknown named entities are left unchanged; numeric
  entities use `String.fromCodePoint` when valid.

***

### descriptionPlainText()

```ts
function descriptionPlainText(raw): string;
```

Builds a plain-text teaser for cards (HTML stripped).

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` \| `null` \| `undefined` | Optional HTML or guided-review JSON body. |

#### Returns

`string`

Single-line plain text.

***

### escapeHtml()

```ts
function escapeHtml(raw): string;
```

Escapes plain text for HTML embedding.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` | Untrusted text. |

#### Returns

`string`

Entity-escaped string safe for HTML text nodes.

***

### parseGuidedReviewBody()

```ts
function parseGuidedReviewBody(raw): GuidedReviewSection[] | null;
```

Best-effort parse of Audible guided review JSON.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` | Review body that may be a JSON array of `{type,question,answer}`. |

#### Returns

[`GuidedReviewSection`](#guidedreviewsection)[] \| `null`

Parsed sections, or `null` when the body is not that schema.

***

### prepareDescriptionHtml()

```ts
function prepareDescriptionHtml(raw): string;
```

Prepares description HTML for safe rendering.

Plain text is escaped; markup is reduced to `p` / `br` / `b`/`strong` /
`i`/`em` / lists (no attributes). Guided JSON bodies return empty string
(render structured React instead).

Uses `sanitizeMarkupToAllowlist` (string token walk). See the module
comment for why we avoid `DOMParser` / `innerHTML` and when to reconsider.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `raw` | `string` | Store or review description. |

#### Returns

`string`

Sanitized HTML fragment.
