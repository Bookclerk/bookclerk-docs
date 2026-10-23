[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / RemoteLibrary

# Interface: RemoteLibrary

Defined in: [generated.ts:1060](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1060)

`remoteLibrary` entrypoint: long-running remote-library lifecycle (start /
stop, library rescan, listening sync, external-user polling). Event delivery
is `EventConsumer`; credential verification is `Oidc`.

## Methods

### diagnose()

> **diagnose**(): `Promise`\<[`DiagnoseReply`](../type-aliases/DiagnoseReply.md)\>

Defined in: [generated.ts:1084](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1084)

Human-readable diagnostic lines.

#### Returns

`Promise`\<[`DiagnoseReply`](../type-aliases/DiagnoseReply.md)\>

[DiagnoseReply](../type-aliases/DiagnoseReply.md)

***

### health()

> **health**(): `Promise`\<[`HealthReply`](../type-aliases/HealthReply.md)\>

Defined in: [generated.ts:1066](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1066)

Liveness / readiness probe.

#### Returns

`Promise`\<[`HealthReply`](../type-aliases/HealthReply.md)\>

[HealthReply](../type-aliases/HealthReply.md)

***

### pollEvents()

> **pollEvents**(): `Promise`\<[`EventPollReply`](../type-aliases/EventPollReply.md)\>

Defined in: [generated.ts:1103](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1103)

Drain external users the remote side observed since the last poll.

#### Returns

`Promise`\<[`EventPollReply`](../type-aliases/EventPollReply.md)\>

[EventPollReply](../type-aliases/EventPollReply.md)

***

### scanLibrary()

> **scanLibrary**(`params`): `Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

Defined in: [generated.ts:1091](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1091)

Re-sync the remote library.

#### Parameters

##### params

[`ScanLibraryParams`](ScanLibraryParams.md)

Full-rescan flag.

#### Returns

`Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

[EmptyReply](../type-aliases/EmptyReply.md)

***

### start()

> **start**(): `Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

Defined in: [generated.ts:1072](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1072)

Start background work after the host has granted bindings.

#### Returns

`Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

[EmptyReply](../type-aliases/EmptyReply.md)

***

### stop()

> **stop**(): `Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

Defined in: [generated.ts:1078](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1078)

Stop background work; the host may drop the capability afterwards.

#### Returns

`Promise`\<[`EmptyReply`](../type-aliases/EmptyReply.md)\>

[EmptyReply](../type-aliases/EmptyReply.md)

***

### syncListening()

> **syncListening**(): `Promise`\<[`SyncListeningReply`](../type-aliases/SyncListeningReply.md)\>

Defined in: [generated.ts:1097](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L1097)

Push / pull listening progress.

#### Returns

`Promise`\<[`SyncListeningReply`](../type-aliases/SyncListeningReply.md)\>

[SyncListeningReply](../type-aliases/SyncListeningReply.md)
