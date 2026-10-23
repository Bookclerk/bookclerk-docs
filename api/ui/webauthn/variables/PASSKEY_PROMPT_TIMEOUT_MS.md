[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [webauthn](../README.md) / PASSKEY\_PROMPT\_TIMEOUT\_MS

# Variable: PASSKEY\_PROMPT\_TIMEOUT\_MS

> `const` **PASSKEY\_PROMPT\_TIMEOUT\_MS**: `45000` = `45_000`

Defined in: [webauthn.ts:141](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/webauthn.ts#L141)

How long to wait for a WebAuthn prompt. Browsers without passkey support
(or in-app WebViews that stub the API) can hang forever on
`navigator.credentials.get` / `.create`.
