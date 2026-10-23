[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / ScanSummary

# Interface: ScanSummary

Defined in: [generated.ts:1583](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1583)

Summary result of `ContentSource.scan`.

## Properties

### accounts

> **accounts**: `number`

Defined in: [generated.ts:1585](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1585)

Number of accounts touched during the scan.

***

### books

> **books**: [`ScanBook`](ScanBook.md)[]

Defined in: [generated.ts:1596](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1596)

Titles for the host to upsert. Prefer this over plugin-side DB writes.

***

### booksUpserted

> **booksUpserted**: `number`

Defined in: [generated.ts:1590](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1590)

Count of titles the guest expects the host to upsert; may mirror
`books.length`.

***

### pages

> **pages**: `number`

Defined in: [generated.ts:1592](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1592)

Number of storefront pages fetched.

***

### skippedDisabled

> **skippedDisabled**: `number`

Defined in: [generated.ts:1594](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1594)

Accounts skipped because `scanEnabled` was false.
