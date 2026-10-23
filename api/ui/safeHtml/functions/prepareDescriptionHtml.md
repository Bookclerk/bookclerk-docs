[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [safeHtml](../README.md) / prepareDescriptionHtml

# Function: prepareDescriptionHtml()

> **prepareDescriptionHtml**(`raw`): `string`

Defined in: [safeHtml.ts:178](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/safeHtml.ts#L178)

Prepares description HTML for safe rendering.

Plain text is escaped; markup is reduced to `p` / `br` / `b`/`strong` /
`i`/`em` / lists (no attributes). Guided JSON bodies return empty string
(render structured React instead).

Uses `sanitizeMarkupToAllowlist` (string token walk). See the module
comment for why we avoid `DOMParser` / `innerHTML` and when to reconsider.

## Parameters

### raw

`string`

Store or review description.

## Returns

`string`

Sanitized HTML fragment.
