[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [safeHtml](../README.md) / parseGuidedReviewBody

# Function: parseGuidedReviewBody()

> **parseGuidedReviewBody**(`raw`): [`GuidedReviewSection`](../type-aliases/GuidedReviewSection.md)[] \| `null`

Defined in: [safeHtml.ts:127](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/safeHtml.ts#L127)

Best-effort parse of Audible guided review JSON.

## Parameters

### raw

`string`

Review body that may be a JSON array of `{type,question,answer}`.

## Returns

[`GuidedReviewSection`](../type-aliases/GuidedReviewSection.md)[] \| `null`

Parsed sections, or `null` when the body is not that schema.
