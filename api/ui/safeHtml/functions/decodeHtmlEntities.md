[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [safeHtml](../README.md) / decodeHtmlEntities

# Function: decodeHtmlEntities()

> **decodeHtmlEntities**(`raw`): `string`

Defined in: [safeHtml.ts:88](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/safeHtml.ts#L88)

Decodes HTML entities (`&rsquo;`, `&#8217;`, …) into Unicode text.

Uses a pure string decoder (no `innerHTML` / DOM text round-trip) so the
result is never sourced from reinterpreted DOM HTML.

## Parameters

### raw

`string`

Text that may contain entities.

## Returns

`string`

Decoded string. Unknown named entities are left unchanged; numeric
  entities use `String.fromCodePoint` when valid.
