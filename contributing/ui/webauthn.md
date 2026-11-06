# webauthn

## Interfaces

### PasskeyCeremonyOptions

Daemon registration challenge (`CreationChallengeResponse` + `challenge_id`).

#### Properties

##### challenge\_id

```ts
challenge_id: string;
```

##### publicKey

```ts
publicKey: Record<string, unknown>;
```

## Variables

### PASSKEY\_PROMPT\_TIMEOUT\_MS

```ts
const PASSKEY_PROMPT_TIMEOUT_MS: 45000 = 45_000;
```

How long to wait for a WebAuthn prompt. Browsers without passkey support
(or in-app WebViews that stub the API) can hang forever on
`navigator.credentials.get` / `.create`.

## Functions

### assertPasskey()

```ts
function assertPasskey(options): Promise<{
  challenge_id: string;
  credential: Record<string, unknown>;
}>;
```

Run an assertion ceremony and return JSON for login/elevate finish.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options` | [`PasskeyCeremonyOptions`](#passkeyceremonyoptions) | - |

#### Returns

`Promise`\<\{
  `challenge_id`: `string`;
  `credential`: `Record`\<`string`, `unknown`\>;
\}\>

***

### createPasskey()

```ts
function createPasskey(options): Promise<{
  challenge_id: string;
  credential: Record<string, unknown>;
}>;
```

Run a registration ceremony and return JSON for `/register/finish`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options` | [`PasskeyCeremonyOptions`](#passkeyceremonyoptions) | - |

#### Returns

`Promise`\<\{
  `challenge_id`: `string`;
  `credential`: `Record`\<`string`, `unknown`\>;
\}\>

***

### creationOptionsFromJson()

```ts
function creationOptionsFromJson(publicKey): PublicKeyCredentialCreationOptions;
```

Decode a registration `publicKey` for `navigator.credentials.create`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `publicKey` | `Record`\<`string`, `unknown`\> | - |

#### Returns

`PublicKeyCredentialCreationOptions`

***

### credentialToJson()

```ts
function credentialToJson(cred): Record<string, unknown>;
```

Serialize a WebAuthn credential for the daemon finish endpoints.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `cred` | `PublicKeyCredential` | - |

#### Returns

`Record`\<`string`, `unknown`\>

***

### passkeysSupported()

```ts
function passkeysSupported(): boolean;
```

True when the WebAuthn credential API is present.

#### Returns

`boolean`

Whether `PublicKeyCredential` and credential create/get are available.

***

### requestOptionsFromJson()

```ts
function requestOptionsFromJson(publicKey): PublicKeyCredentialRequestOptions;
```

Decode an assertion `publicKey` for `navigator.credentials.get`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `publicKey` | `Record`\<`string`, `unknown`\> | - |

#### Returns

`PublicKeyCredentialRequestOptions`
