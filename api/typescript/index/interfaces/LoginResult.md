[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / LoginResult

# Interface: LoginResult

Defined in: [generated.ts:1449](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1449)

Result of `ContentSource.login` / `loginComplete`: account metadata plus
opaque credentials for the host to seal into `encrypted_secrets`
(`provider = plugin id`). Guests never write secrets into the library DB.

## Properties

### account

> **account**: [`SourceAccount`](SourceAccount.md)

Defined in: [generated.ts:1451](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1451)

Account row fields for the host to upsert.

***

### credentials?

> `optional` **credentials?**: `Uint8Array`\<`ArrayBufferLike`\>

Defined in: [generated.ts:1457](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1457)

Opaque credential blob the host seals; empty when login only refreshed
metadata. Guests choose the encoding (typically JSON bytes).
Omitted when absent (wire zero value).
