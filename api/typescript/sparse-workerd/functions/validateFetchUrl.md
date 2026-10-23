[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [sparse-workerd](../README.md) / validateFetchUrl

# Function: validateFetchUrl()

> **validateFetchUrl**(`url`): `string`

Defined in: [sparse-workerd/ensure.ts:365](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/sparse-workerd/ensure.ts#L365)

Validates a URL before `fetch` / download.

Allows `https:` anywhere, or `http:` only to loopback hosts.

## Parameters

### url

`string`

Absolute URL string.

## Returns

`string`

Canonical href safe to request.

## Throws

When the URL is invalid or uses a disallowed scheme/host.
