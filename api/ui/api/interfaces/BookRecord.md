[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / BookRecord

# Interface: BookRecord

Defined in: [api.ts:97](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L97)

Library book row from `/api/library/books`.

Field names match the daemon JSON (`snake_case`). Use `uuid` for acquire /
cover URLs; `asin` / `isbn` are store product ids when known.

## Properties

### account\_id

> **account\_id**: `string`

Defined in: [api.ts:105](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L105)

Sealed account id that owns this title.

***

### acquire\_status

> **acquire\_status**: [`AcquireStatus`](../type-aliases/AcquireStatus.md)

Defined in: [api.ts:127](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L127)

Pipeline state for the main audio acquire job.

***

### asin

> **asin**: `string` \| `null`

Defined in: [api.ts:109](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L109)

Audible ASIN when the source provides one.

***

### authors

> **authors**: `string` \| `null`

Defined in: [api.ts:117](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L117)

Semicolon- or comma-separated author list.

***

### categories

> **categories**: `string` \| `null`

Defined in: [api.ts:157](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L157)

Genre / category list from the store (often `;`- or `,`-separated).

***

### content\_kind

> **content\_kind**: `string`

Defined in: [api.ts:155](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L155)

Content kind slug (`audiobook`, `podcast`, …).

***

### cover\_url

> **cover\_url**: `string` \| `null`

Defined in: [api.ts:167](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L167)

Remote or proxied cover image URL.

***

### created\_at

> **created\_at**: `string`

Defined in: [api.ts:177](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L177)

Row insert time (ISO-8601).

***

### description

> **description**: `string` \| `null`

Defined in: [api.ts:163](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L163)

Long-form description / blurb (may contain light HTML).

***

### enrich\_confidence

> **enrich\_confidence**: `number` \| `null`

Defined in: [api.ts:173](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L173)

Enrichment confidence score (0–1) when present.

***

### enrich\_source

> **enrich\_source**: `string` \| `null`

Defined in: [api.ts:171](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L171)

Enrichment provider id that last wrote metadata.

***

### enrich\_updated\_at

> **enrich\_updated\_at**: `string` \| `null`

Defined in: [api.ts:175](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L175)

When enrichment last ran (ISO-8601).

***

### error\_message

> **error\_message**: `string` \| `null`

Defined in: [api.ts:131](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L131)

Last acquire / scan error message when status is `error`.

***

### id

> **id**: `number`

Defined in: [api.ts:99](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L99)

Library row primary key.

***

### is\_abridged

> **is\_abridged**: `boolean`

Defined in: [api.ts:153](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L153)

True when the edition is abridged.

***

### is\_finished

> **is\_finished**: `boolean`

Defined in: [api.ts:143](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L143)

Whether the listener marked the title finished.

***

### isbn

> **isbn**: `string` \| `null`

Defined in: [api.ts:111](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L111)

ISBN-13 / ISBN-10 when the source provides one.

***

### language

> **language**: `string` \| `null`

Defined in: [api.ts:165](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L165)

BCP-47-ish language code from the store.

***

### length\_minutes

> **length\_minutes**: `number` \| `null`

Defined in: [api.ts:151](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L151)

Runtime length in minutes when known.

***

### marketplace

> **marketplace**: `string`

Defined in: [api.ts:113](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L113)

Marketplace / locale code (`us`, `uk`, …).

***

### narrators

> **narrators**: `string` \| `null`

Defined in: [api.ts:119](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L119)

Semicolon- or comma-separated narrator list.

***

### pdf\_status

> **pdf\_status**: [`AcquireStatus`](../type-aliases/AcquireStatus.md)

Defined in: [api.ts:145](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L145)

Pipeline state for companion PDF acquire.

***

### pdf\_storage\_key

> **pdf\_storage\_key**: `string` \| `null`

Defined in: [api.ts:147](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L147)

Destination key for an acquired companion PDF.

***

### product\_id

> **product\_id**: `string`

Defined in: [api.ts:107](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L107)

Store-native product id (ASIN, ISBN, UUID, …).

***

### published\_at

> **published\_at**: `string` \| `null`

Defined in: [api.ts:161](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L161)

Original publication date (ISO-8601 date or datetime).

***

### publisher

> **publisher**: `string` \| `null`

Defined in: [api.ts:149](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L149)

Publisher name when known.

***

### purchased\_at

> **purchased\_at**: `string` \| `null`

Defined in: [api.ts:133](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L133)

Purchase / library-add timestamp from the store (ISO-8601).

***

### rating\_overall

> **rating\_overall**: `number` \| `null`

Defined in: [api.ts:137](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L137)

Overall community rating (0–5) when enriched.

***

### rating\_performance

> **rating\_performance**: `number` \| `null`

Defined in: [api.ts:139](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L139)

Performance / narrator rating when enriched.

***

### rating\_story

> **rating\_story**: `number` \| `null`

Defined in: [api.ts:141](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L141)

Story rating when enriched.

***

### series

> **series**: `string` \| `null`

Defined in: [api.ts:121](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L121)

Series name when known.

***

### series\_asin

> **series\_asin**: `string` \| `null`

Defined in: [api.ts:125](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L125)

Audible series / podcast-parent ASIN when known.

***

### series\_index

> **series\_index**: `string` \| `null`

Defined in: [api.ts:123](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L123)

Position within the series (string to preserve `#1.5` styles).

***

### source

> **source**: `string`

Defined in: [api.ts:103](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L103)

Storefront plugin id (`audible`, `libro`, …).

***

### storage\_key

> **storage\_key**: `string` \| `null`

Defined in: [api.ts:129](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L129)

Destination object key after a successful acquire.

***

### subjects

> **subjects**: `string` \| `null`

Defined in: [api.ts:169](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L169)

Subject / BISAC-style tags when enriched.

***

### subtitle

> **subtitle**: `string` \| `null`

Defined in: [api.ts:159](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L159)

Subtitle when the store provides one.

***

### tags

> **tags**: `string` \| `null`

Defined in: [api.ts:135](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L135)

Space-separated user tags.

***

### title

> **title**: `string`

Defined in: [api.ts:115](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L115)

Display title.

***

### updated\_at

> **updated\_at**: `string`

Defined in: [api.ts:179](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L179)

Row last update time (ISO-8601).

***

### uuid

> **uuid**: `string`

Defined in: [api.ts:101](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L101)

Stable book uuid used by acquire / cover / detail APIs.
