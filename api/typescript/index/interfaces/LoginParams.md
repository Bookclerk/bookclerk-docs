[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / LoginParams

# Interface: LoginParams

Defined in: [generated.ts:1384](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1384)

Params of `ContentSource.login` and `ContentSource.loginStart`. Password
sources fill email/password; OAuth sources use callback / external fields.
There is no files-dir root or library DB path -- only `pluginDataDir`.

## Properties

### callbackBind?

> `optional` **callbackBind?**: `string`

Defined in: [generated.ts:1411](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1411)

Bind address for OAuth callback servers (`host:port`). Ignored when
`callbackIpc` is set (host owns the TCP listener).
Omitted when absent (wire zero value).

***

### callbackIpc?

> `optional` **callbackIpc?**: `string`

Defined in: [generated.ts:1417](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1417)

Host-owned callback IPC endpoint the guest must connect to. When set
(with `callbackPublicBase`), the guest must not bind a TCP listener.
Omitted when absent (wire zero value).

***

### callbackPublicBase?

> `optional` **callbackPublicBase?**: `string`

Defined in: [generated.ts:1422](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1422)

Public base URL for the host TCP listener, e.g. `http://127.0.0.1:12345`.
Omitted when absent (wire zero value).

***

### email?

> `optional` **email?**: `string`

Defined in: [generated.ts:1398](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1398)

Account email / username for password logins; empty for pure OAuth.
Omitted when absent (wire zero value).

***

### external

> **external**: `boolean`

Defined in: [generated.ts:1427](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1427)

When true, use external / paste-redirect OAuth instead of a local
callback server.

***

### extra

> **extra**: [`ExtensibleConfig`](ExtensibleConfig.md)

Defined in: [generated.ts:1441](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1441)

Store-specific knobs as `application/json`; guests may ignore unknowns.

***

### force

> **force**: `boolean`

Defined in: [generated.ts:1405](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1405)

When true, overwrite an existing sealed credential for this account.

***

### label?

> `optional` **label?**: `string`

Defined in: [generated.ts:1393](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1393)

Operator label stored on the account row.
Omitted when absent (wire zero value).

***

### marketplace

> **marketplace**: `string`

Defined in: [generated.ts:1388](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1388)

Marketplace / locale for the storefront; empty means the guest default.

***

### password?

> `optional` **password?**: `string`

Defined in: [generated.ts:1403](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1403)

Account password for password logins; never logged; empty for OAuth.
Omitted when absent (wire zero value).

***

### pluginDataDir

> **pluginDataDir**: `string`

Defined in: [generated.ts:1386](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1386)

Scoped writable directory for this plugin only (`.../plugins/<id>/data`).

***

### responseUrl?

> `optional` **responseUrl?**: `string`

Defined in: [generated.ts:1432](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1432)

Pre-supplied OAuth redirect URL (paste flow).
Omitted when absent (wire zero value).

***

### showQr

> **showQr**: `boolean`

Defined in: [generated.ts:1434](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1434)

Prefer QR output when the guest supports it.

***

### timeoutSecs?

> `optional` **timeoutSecs?**: `number`

Defined in: [generated.ts:1439](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1439)

Seconds to wait for OAuth callback capture; guest default when 0.
Omitted when absent (wire zero value).
